# DevGuide Cockpit: Offline Architecture Specification

**Document Type:** Technical Implementation (Tier 5)  
**Status:** Active  
**Version:** 1.0  
**Date:** January 1, 2026  
**Dependencies:** UNIFIED_AI_LAYER.md, DATA_MODEL.md, THE_BIN_SPECIFICATION.md

---

## Executive Summary

This artifact specifies the **Progressive Web App (PWA) and offline-first architecture** for DevGuide Cockpit. Users can work on dashboards without internet connectivity, with changes queued locally and synced when online.

**Core Capabilities:**
- Full dashboard editing offline (Elm app cached in service worker)
- The Bin queues changes locally (IndexedDB)
- Git commits work offline (local repository)
- AI layer gracefully degrades (cached responses)
- Background sync when reconnected

---

## Offline Capability Matrix

### What Works Offline (Tier 1 - Core Functionality)

| Feature | Offline Support | Implementation |
|---------|-----------------|----------------|
| **Dashboard UI** | ✅ Full | Service worker caches Elm compiled JS |
| **Project Editing** | ✅ Full | Changes saved to IndexedDB |
| **The Bin** | ✅ Full | Local queue, syncs when online |
| **Git Commits** | ✅ Full | Local Git, push deferred |
| **Guardrail Validation (Rust)** | ✅ Full | WASM runs in browser |
| **Documentation Generation** | ✅ Full | Python scripts run locally |
| **Version History** | ✅ Full | Git log available locally |
| **Search (Project Knowledge)** | ✅ Full | IndexedDB full-text search |

### What Requires Online (Tier 2 - Enhanced Features)

| Feature | Why Online Required | Offline Fallback |
|---------|---------------------|------------------|
| **AI Assistance** | Claude API remote | Show cached suggestions |
| **Template Downloads** | Community registry | Use cached templates only |
| **Cross-Device Sync** | Central TiDB server | Local-only until online |
| **SSE (Real-Time Updates)** | WebSocket connection | Polling when reconnected |
| **External API Calls** | By definition | Queue requests |

---

## Service Worker Architecture

### Service Worker Registration

**File:** `/chromium-shell/service-worker.js`

```javascript
// Register service worker on app load
if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('/service-worker.js')
        .then(registration => {
            console.log('Service Worker registered:', registration.scope);
            
            // Listen for updates
            registration.addEventListener('updatefound', () => {
                const newWorker = registration.installing;
                newWorker.addEventListener('statechange', () => {
                    if (newWorker.state === 'installed' && navigator.serviceWorker.controller) {
                        // New version available - show update notification
                        showUpdateNotification();
                    }
                });
            });
        })
        .catch(err => console.error('Service Worker registration failed:', err));
}
```

### Caching Strategies

**Strategy 1: Cache First (App Shell)**

Used for: Elm compiled JavaScript, CSS, fonts, icons

```javascript
const APP_SHELL_CACHE = 'devguide-app-shell-v1';
const APP_SHELL_FILES = [
    '/',
    '/index.html',
    '/elm.js',
    '/styles.css',
    '/fonts/inter.woff2',
    '/icons/manifest-icon-192.png',
    '/icons/manifest-icon-512.png',
];

self.addEventListener('install', event => {
    event.waitUntil(
        caches.open(APP_SHELL_CACHE)
            .then(cache => cache.addAll(APP_SHELL_FILES))
    );
});

self.addEventListener('fetch', event => {
    const url = new URL(event.request.url);
    if (APP_SHELL_FILES.includes(url.pathname)) {
        // Cache first, fallback to network
        event.respondWith(
            caches.match(event.request)
                .then(response => response || fetch(event.request))
        );
    }
});
```

**Strategy 2: Network First, Cache Fallback (API Responses)**

Used for: Project data, dashboard configurations, user preferences

```javascript
const API_CACHE = 'devguide-api-v1';

self.addEventListener('fetch', event => {
    if (event.request.url.includes('/api/')) {
        event.respondWith(
            fetch(event.request)
                .then(response => {
                    // Cache successful responses
                    if (response.ok) {
                        const responseClone = response.clone();
                        caches.open(API_CACHE).then(cache => {
                            cache.put(event.request, responseClone);
                        });
                    }
                    return response;
                })
                .catch(() => {
                    // Network failed, try cache
                    return caches.match(event.request)
                        .then(cached => {
                            if (cached) {
                                // Append offline indicator to cached response
                                return addOfflineHeader(cached);
                            }
                            return new Response('Offline and no cached data', { 
                                status: 503,
                                statusText: 'Service Unavailable' 
                            });
                        });
                })
        );
    }
});

function addOfflineHeader(response) {
    const headers = new Headers(response.headers);
    headers.set('X-Offline-Cache', 'true');
    return new Response(response.body, {
        status: response.status,
        statusText: response.statusText,
        headers: headers
    });
}
```

**Strategy 3: Stale While Revalidate (Static Assets)**

Used for: Dashboard templates, documentation pages, guardrail definitions

```javascript
const STATIC_CACHE = 'devguide-static-v1';

self.addEventListener('fetch', event => {
    if (event.request.url.includes('/static/')) {
        event.respondWith(
            caches.match(event.request)
                .then(cached => {
                    // Return cached version immediately
                    const fetchPromise = fetch(event.request)
                        .then(response => {
                            // Update cache in background
                            caches.open(STATIC_CACHE).then(cache => {
                                cache.put(event.request, response.clone());
                            });
                            return response;
                        })
                        .catch(() => cached);  // If fetch fails, keep using cache
                    
                    return cached || fetchPromise;
                })
        );
    }
});
```

---

## IndexedDB Schema

### Database Setup

**Database Name:** `devguide-offline`  
**Version:** 1

```javascript
const DB_NAME = 'devguide-offline';
const DB_VERSION = 1;

function openDB() {
    return new Promise((resolve, reject) => {
        const request = indexedDB.open(DB_NAME, DB_VERSION);
        
        request.onerror = () => reject(request.error);
        request.onsuccess = () => resolve(request.result);
        
        request.onupgradeneeded = event => {
            const db = event.target.result;
            
            // Store 1: Projects
            if (!db.objectStoreNames.contains('projects')) {
                const projectStore = db.createObjectStore('projects', { keyPath: 'id' });
                projectStore.createIndex('owner_id', 'owner_id', { unique: false });
                projectStore.createIndex('last_modified', 'last_modified', { unique: false });
            }
            
            // Store 2: Dashboards
            if (!db.objectStoreNames.contains('dashboards')) {
                const dashboardStore = db.createObjectStore('dashboards', { keyPath: 'id' });
                dashboardStore.createIndex('project_id', 'project_id', { unique: false });
            }
            
            // Store 3: The Bin (pending sync items)
            if (!db.objectStoreNames.contains('bin_queue')) {
                const binStore = db.createObjectStore('bin_queue', { keyPath: 'id' });
                binStore.createIndex('status', 'status', { unique: false });
                binStore.createIndex('created_at', 'created_at', { unique: false });
            }
            
            // Store 4: Cached AI Responses
            if (!db.objectStoreNames.contains('ai_cache')) {
                const aiCacheStore = db.createObjectStore('ai_cache', { keyPath: 'query_hash' });
                aiCacheStore.createIndex('expires_at', 'expires_at', { unique: false });
            }
            
            // Store 5: Offline Changes (pending upload)
            if (!db.objectStoreNames.contains('pending_changes')) {
                const changesStore = db.createObjectStore('pending_changes', { keyPath: 'id' });
                changesStore.createIndex('project_id', 'project_id', { unique: false });
                changesStore.createIndex('retry_count', 'retry_count', { unique: false });
            }
        };
    });
}
```

### CRUD Operations

```javascript
// Save project to IndexedDB
async function saveProjectOffline(project) {
    const db = await openDB();
    const tx = db.transaction('projects', 'readwrite');
    const store = tx.objectStore('projects');
    
    project.last_modified = new Date().toISOString();
    await store.put(project);
    
    return tx.complete;
}

// Get project from IndexedDB
async function getProjectOffline(projectId) {
    const db = await openDB();
    const tx = db.transaction('projects', 'readonly');
    const store = tx.objectStore('projects');
    
    return await store.get(projectId);
}

// Queue change in The Bin
async function queueChangeForSync(change) {
    const db = await openDB();
    const tx = db.transaction('bin_queue', 'readwrite');
    const store = tx.objectStore('bin_queue');
    
    const item = {
        id: generateUUID(),
        project_id: change.project_id,
        change_type: change.type,
        change_data: change.data,
        status: 'pending',
        created_at: new Date().toISOString(),
        retry_count: 0
    };
    
    await store.put(item);
    
    // Register background sync
    if ('serviceWorker' in navigator && 'sync' in ServiceWorkerRegistration.prototype) {
        const registration = await navigator.serviceWorker.ready;
        await registration.sync.register('bin-sync');
    }
    
    return tx.complete;
}

// Get pending changes (for sync)
async function getPendingChanges() {
    const db = await openDB();
    const tx = db.transaction('bin_queue', 'readonly');
    const store = tx.objectStore('bin_queue');
    const index = store.index('status');
    
    return await index.getAll('pending');
}
```

---

## The Bin Offline Behavior

### Offline Queue Strategy

**Workflow:**
1. User makes change in dashboard (Elm)
2. Change sent to The Bin (Rust WASM validation)
3. If validation passes:
   - Change queued in IndexedDB (`bin_queue` store)
   - Status: `pending`
   - Background sync registered
4. When online:
   - Service worker triggers sync event
   - Pending changes uploaded to TiDB
   - Status updated to `committed` or `failed`

### Conflict Resolution

**Problem:** User makes change offline, another user makes conflicting change online.

**Solution:** Last-Write-Wins with Manual Merge UI

```javascript
async function syncBinQueue() {
    const pendingChanges = await getPendingChanges();
    
    for (const change of pendingChanges) {
        try {
            // Upload change to server
            const response = await fetch('/api/bin/commit', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(change)
            });
            
            if (response.ok) {
                // Success - mark as committed
                await updateChangeStatus(change.id, 'committed');
            } else if (response.status === 409) {
                // Conflict detected
                const conflict = await response.json();
                await handleConflict(change, conflict);
            } else {
                // Other error - retry later
                await incrementRetryCount(change.id);
            }
        } catch (err) {
            // Network error - will retry on next sync
            console.error('Sync failed for change', change.id, err);
        }
    }
}

async function handleConflict(localChange, serverConflict) {
    // Show conflict resolution UI to user
    const resolution = await showConflictDialog({
        localChange: localChange,
        serverChange: serverConflict.conflict_data,
        conflictType: serverConflict.conflict_type
    });
    
    if (resolution === 'keep-local') {
        // Force push local change (requires permission)
        await fetch('/api/bin/force-commit', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ ...localChange, force: true })
        });
    } else if (resolution === 'keep-server') {
        // Discard local change
        await updateChangeStatus(localChange.id, 'rejected');
    } else {
        // Merge manually
        const merged = await showMergeEditor(localChange, serverConflict.conflict_data);
        await fetch('/api/bin/commit', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(merged)
        });
    }
}
```

---

## AI Layer Offline Behavior

### Graceful Degradation

**Online:** Full AI assistance (Claude API)  
**Offline:** Cached responses + static suggestions

```rust
// ai/offline_provider.rs
pub struct OfflineAIProvider {
    cache: Arc<IndexedDBCache>,
}

impl AIProvider for OfflineAIProvider {
    async fn complete(&self, prompt: &str, max_tokens: u32) -> Result<AIResponse, AIError> {
        // Check cache first
        let query_hash = hash_prompt(prompt);
        
        if let Some(cached) = self.cache.get(query_hash).await {
            if !cached.is_expired() {
                return Ok(cached.response);
            }
        }
        
        // No cached response - return static suggestion
        Err(AIError::OfflineMode {
            suggestion: self.get_static_suggestion_for_context(prompt)
        })
    }
    
    fn supports_tool_use(&self) -> bool {
        false  // Tool use requires online
    }
}

fn get_static_suggestion_for_context(prompt: &str) -> String {
    // Return pre-generated suggestions based on context
    if prompt.contains("architecture") {
        "Offline mode: Consider reviewing the Architecture Guide in docs."
    } else if prompt.contains("security") {
        "Offline mode: Review Security Best Practices checklist."
    } else {
        "Offline mode: AI assistance unavailable. Reconnect for full help."
    }.to_string()
}
```

### AI Response Caching

**Cache Strategy:**
- Cache all AI responses in IndexedDB (`ai_cache` store)
- TTL: 7 days
- Eviction: LRU (least recently used)

```javascript
// ai/cache.js
async function cacheAIResponse(prompt, response) {
    const db = await openDB();
    const tx = db.transaction('ai_cache', 'readwrite');
    const store = tx.objectStore('ai_cache');
    
    const query_hash = await hashString(prompt);
    const expires_at = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000);  // 7 days
    
    await store.put({
        query_hash: query_hash,
        prompt: prompt,
        response: response,
        created_at: new Date().toISOString(),
        expires_at: expires_at.toISOString(),
        hit_count: 0
    });
    
    return tx.complete;
}

async function getCachedAIResponse(prompt) {
    const db = await openDB();
    const tx = db.transaction('ai_cache', 'readonly');
    const store = tx.objectStore('ai_cache');
    
    const query_hash = await hashString(prompt);
    const cached = await store.get(query_hash);
    
    if (!cached) return null;
    
    // Check expiration
    if (new Date(cached.expires_at) < new Date()) {
        // Expired - delete and return null
        await deleteExpiredCache(query_hash);
        return null;
    }
    
    // Increment hit count
    await incrementCacheHitCount(query_hash);
    
    return cached.response;
}

async function hashString(str) {
    const encoder = new TextEncoder();
    const data = encoder.encode(str);
    const hashBuffer = await crypto.subtle.digest('SHA-256', data);
    const hashArray = Array.from(new Uint8Array(hashBuffer));
    return hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
}
```

---

## Git Offline Operations

### Local Git Repository

**Strategy:** Full Git repository cloned locally (Electron/Chromium shell)

```go
// git/offline_service.go
package git

import (
    "context"
    "os/exec"
    "time"
)

type OfflineGitService struct {
    localRepoPath string
    pendingPushes []PendingPush
}

type PendingPush struct {
    CommitSHA string
    Message   string
    Timestamp time.Time
}

func (g *OfflineGitService) CommitOffline(ctx context.Context, message string) error {
    // Commit to local repo
    cmd := exec.CommandContext(ctx, "git", "commit", "-m", message)
    cmd.Dir = g.localRepoPath
    
    if err := cmd.Run(); err != nil {
        return fmt.Errorf("git commit failed: %w", err)
    }
    
    // Queue push for when online
    g.pendingPushes = append(g.pendingPushes, PendingPush{
        CommitSHA: getCurrentCommitSHA(),
        Message:   message,
        Timestamp: time.Now(),
    })
    
    return nil
}

func (g *OfflineGitService) SyncWhenOnline(ctx context.Context) error {
    if !isOnline() {
        return ErrOffline
    }
    
    // Push all pending commits
    for _, push := range g.pendingPushes {
        cmd := exec.CommandContext(ctx, "git", "push", "origin", "main")
        cmd.Dir = g.localRepoPath
        
        if err := cmd.Run(); err != nil {
            return fmt.Errorf("git push failed for %s: %w", push.CommitSHA, err)
        }
    }
    
    g.pendingPushes = []PendingPush{}
    return nil
}
```

---

## Network Status Detection

### Online/Offline Detection

**Browser API:**
```javascript
// Detect online/offline status
window.addEventListener('online', handleOnline);
window.addEventListener('offline', handleOffline);

function handleOnline() {
    console.log('Back online - syncing...');
    
    // Trigger sync immediately
    syncBinQueue();
    syncGitCommits();
    
    // Update UI
    updateNetworkStatus('online');
}

function handleOffline() {
    console.log('Gone offline - enabling offline mode');
    
    // Update UI
    updateNetworkStatus('offline');
    
    // Show offline indicator
    showOfflineIndicator();
}

// Check connection quality
async function checkConnectionQuality() {
    const connection = navigator.connection || navigator.mozConnection || navigator.webkitConnection;
    
    if (connection) {
        console.log('Connection type:', connection.effectiveType);
        console.log('Downlink speed:', connection.downlink, 'Mbps');
        console.log('RTT:', connection.rtt, 'ms');
        
        // Adapt behavior based on connection quality
        if (connection.effectiveType === 'slow-2g' || connection.effectiveType === '2g') {
            // Treat as offline (too slow for good UX)
            handleOffline();
        }
    }
}
```

### UI Indicators

**Elm Implementation:**
```elm
-- Elm model
type alias Model =
    { networkStatus : NetworkStatus
    , pendingSyncCount : Int
    , lastSyncAt : Maybe Time.Posix
    }

type NetworkStatus
    = Online
    | Offline
    | Syncing

-- View network status indicator
viewNetworkStatus : Model -> Html Msg
viewNetworkStatus model =
    div [ class "network-status" ]
        [ case model.networkStatus of
            Online ->
                div [ class "status-online" ]
                    [ icon "wifi"
                    , text "Online"
                    , if model.pendingSyncCount > 0 then
                        badge model.pendingSyncCount "Pending"
                      else
                        text ""
                    ]
            
            Offline ->
                div [ class "status-offline" ]
                    [ icon "wifi-off"
                    , text "Offline"
                    , badge model.pendingSyncCount "Queued"
                    ]
            
            Syncing ->
                div [ class "status-syncing" ]
                    [ spinner
                    , text "Syncing..."
                    , progress model.pendingSyncCount
                    ]
        ]
```

---

## Background Sync API

### Register Sync Task

```javascript
// Register background sync when user makes change offline
async function registerBackgroundSync() {
    if ('serviceWorker' in navigator && 'sync' in ServiceWorkerRegistration.prototype) {
        const registration = await navigator.serviceWorker.ready;
        await registration.sync.register('bin-sync');
    }
}

// Service worker handles sync event
self.addEventListener('sync', event => {
    if (event.tag === 'bin-sync') {
        event.waitUntil(syncBinQueue());
    }
});

async function syncBinQueue() {
    const pendingChanges = await getPendingChanges();
    
    for (const change of pendingChanges) {
        try {
            await uploadChange(change);
            await markChangeSynced(change.id);
        } catch (err) {
            console.error('Sync failed for', change.id, err);
            // Browser will retry automatically
        }
    }
}
```

**Advantages:**
- Browser retries sync automatically if it fails
- Sync happens even if user closes the app
- Battery-efficient (browser schedules syncs)

---

## PWA Manifest

**File:** `/public/manifest.json`

```json
{
  "name": "DevGuide Cockpit",
  "short_name": "DevGuide",
  "description": "AI-powered software development dashboard system",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#4F46E5",
  "icons": [
    {
      "src": "/icons/manifest-icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/manifest-icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable any"
    }
  ],
  "categories": ["productivity", "developer tools"],
  "screenshots": [
    {
      "src": "/screenshots/desktop-dashboard.png",
      "sizes": "1920x1080",
      "type": "image/png",
      "form_factor": "wide"
    },
    {
      "src": "/screenshots/mobile-dashboard.png",
      "sizes": "750x1334",
      "type": "image/png",
      "form_factor": "narrow"
    }
  ],
  "offline_enabled": true
}
```

---

## Performance Considerations

### Cache Size Limits

**Browser Quotas:**
- Chrome: ~60% of free disk space
- Firefox: ~50% of free disk space
- Safari: 50 MB limit

**Strategy:**
```javascript
// Check quota usage
if ('storage' in navigator && 'estimate' in navigator.storage) {
    navigator.storage.estimate().then(estimate => {
        const percentUsed = (estimate.usage / estimate.quota) * 100;
        console.log(`Storage: ${percentUsed.toFixed(2)}% used`);
        
        if (percentUsed > 80) {
            // Evict old cache entries
            evictOldCache();
        }
    });
}

async function evictOldCache() {
    const db = await openDB();
    const tx = db.transaction(['ai_cache', 'bin_queue'], 'readwrite');
    
    // Delete AI cache entries older than 30 days
    const aiStore = tx.objectStore('ai_cache');
    const aiIndex = aiStore.index('created_at');
    const thirtyDaysAgo = new Date(Date.now() - 30 * 24 * 60 * 60 * 1000);
    
    const oldEntries = await aiIndex.getAll(IDBKeyRange.upperBound(thirtyDaysAgo));
    for (const entry of oldEntries) {
        await aiStore.delete(entry.query_hash);
    }
    
    // Delete committed bin items older than 7 days
    const binStore = tx.objectStore('bin_queue');
    const binItems = await binStore.getAll();
    for (const item of binItems) {
        if (item.status === 'committed' && 
            new Date(item.created_at) < new Date(Date.now() - 7 * 24 * 60 * 60 * 1000)) {
            await binStore.delete(item.id);
        }
    }
    
    await tx.complete;
}
```

---

## Testing Strategy

### Service Worker Testing

```javascript
// test/offline-test.js
describe('Offline Functionality', () => {
    beforeEach(() => {
        // Force offline mode
        navigator.serviceWorker.controller.postMessage({
            type: 'FORCE_OFFLINE'
        });
    });
    
    it('should queue changes when offline', async () => {
        const change = { dashboard_id: 'arch-001', data: {...} };
        await queueChangeForSync(change);
        
        const queue = await getPendingChanges();
        expect(queue).toHaveLength(1);
        expect(queue[0].status).toBe('pending');
    });
    
    it('should sync when online', async () => {
        // Add offline changes
        await queueChangeForSync({ ... });
        
        // Go online
        navigator.serviceWorker.controller.postMessage({
            type: 'GO_ONLINE'
        });
        
        // Wait for sync
        await waitForSync();
        
        const queue = await getPendingChanges();
        expect(queue).toHaveLength(0);  // All synced
    });
});
```

---

**End of ARTIFACT_17_OFFLINE_ARCHITECTURE_SPECIFICATION.md**
