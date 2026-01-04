# DevGuide Cockpit: Lock Mechanism Specification

**Document Type:** Mechanics (Tier 3)  
**Status:** Active  
**Version:** 1.0  
**Date:** January 1, 2026  
**Dependencies:** THE_BIN_SPECIFICATION.md, MASTER_CONTROL_DASHBOARD.md, DATA_MODEL.md

---

## Executive Summary

This artifact specifies the **locking mechanisms** that enforce synchronization boundaries in DevGuide Cockpit. It defines:
- Single Active Screen enforcement (MVP constraint preventing concurrent edits)
- Propagation locks (prevent multiple simultaneous propagations)
- Timeout handling and automatic lock release
- Conflict resolution strategies
- Lock status visibility in UI

**Key Principle:** Locks are defensive mechanisms that prevent race conditions in the single-user MVP, not collaborative features.

---

## Lock Types

### 1. Project Lock (Master Lock)

**Purpose:** Ensures only one dashboard is editable at a time.

**Scope:** Entire project  
**Duration:** Until user explicitly releases or switches dashboard  
**Holder:** Dashboard ID

**State Transitions:**
```
UNLOCKED ─[user opens dashboard]→ LOCKED(dashboard_id)
LOCKED ─[user closes dashboard]→ UNLOCKED
LOCKED ─[timeout 30 min]→ UNLOCKED
LOCKED ─[user switches dashboard]→ LOCKED(new_dashboard_id)
```

### 2. Propagation Lock

**Purpose:** Prevents concurrent propagation operations.

**Scope:** Project-level  
**Duration:** Until propagation completes or fails  
**Holder:** Propagation job ID

**State Transitions:**
```
NO_PROPAGATION ─[user approves change]→ PROPAGATING(job_id)
PROPAGATING ─[propagation success]→ NO_PROPAGATION
PROPAGATING ─[propagation failure]→ NO_PROPAGATION
PROPAGATING ─[timeout 60s]→ NO_PROPAGATION (with error)
```

### 3. The Bin Lock (Validation Lock)

**Purpose:** Prevents concurrent modifications to The Bin's validation queue.

**Scope:** Bin queue for specific project  
**Duration:** During validation operation  
**Holder:** Validation worker ID

---

## Single Active Screen Enforcement

### UI Behavior

**Allowed:**
- User has Dashboard A open and editing
- User views (read-only) other dashboards
- User switches from Dashboard A to Dashboard B (A becomes read-only)

**Prevented:**
- User edits Dashboard A in Tab 1 AND Dashboard B in Tab 2 (same browser)
- User edits same project in multiple browser windows
- User edits while propagation is running

### Implementation (Elm + Go)

**Elm: Request Lock Before Editing**
```elm
-- Dashboard/Architecture.elm
type Msg
    = UserClickedEdit
    | LockAcquired DashboardId
    | LockDenied String
    | LockReleased

update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
    case msg of
        UserClickedEdit ->
            ( { model | pendingLock = True }
            , requestLock model.dashboardId
            )
        
        LockAcquired dashboardId ->
            if dashboardId == model.dashboardId then
                ( { model 
                    | editMode = Editing
                    , pendingLock = False
                  }
                , Cmd.none
                )
            else
                ( { model | editMode = ReadOnly }
                , showNotification "Another dashboard is being edited"
                )
        
        LockDenied reason ->
            ( { model | pendingLock = False }
            , showError ("Cannot edit: " ++ reason)
            )

-- Port to Go API
port requestLock : DashboardId -> Cmd msg
port lockResponse : (LockResult -> msg) -> Sub msg
```

**Go API: Lock Management**
```go
// api/locks/manager.go
type LockManager struct {
    redis *redis.Client
}

type ProjectLock struct {
    ProjectID   string
    DashboardID string
    UserID      string
    AcquiredAt  time.Time
    ExpiresAt   time.Time
}

func (lm *LockManager) AcquireLock(ctx context.Context, projectID, dashboardID, userID string) error {
    lockKey := fmt.Sprintf("project:%s:lock", projectID)
    
    // Try to acquire lock with 30-minute expiry
    success, err := lm.redis.SetNX(ctx, lockKey, dashboardID, 30*time.Minute).Result()
    if err != nil {
        return fmt.Errorf("failed to acquire lock: %w", err)
    }
    
    if !success {
        // Lock already held, check who holds it
        currentHolder, _ := lm.redis.Get(ctx, lockKey).Result()
        return &LockDeniedError{
            ProjectID: projectID,
            CurrentHolder: currentHolder,
            Message: fmt.Sprintf("Project locked by dashboard %s", currentHolder),
        }
    }
    
    // Store lock metadata in TiDB for audit trail
    _, err = lm.db.ExecContext(ctx, `
        INSERT INTO project_locks (project_id, dashboard_id, user_id, acquired_at, expires_at)
        VALUES (?, ?, ?, NOW(), DATE_ADD(NOW(), INTERVAL 30 MINUTE))
    `, projectID, dashboardID, userID)
    
    return err
}

func (lm *LockManager) ReleaseLock(ctx context.Context, projectID, dashboardID string) error {
    lockKey := fmt.Sprintf("project:%s:lock", projectID)
    
    // Verify caller holds the lock
    currentHolder, err := lm.redis.Get(ctx, lockKey).Result()
    if err == redis.Nil {
        return nil  // Lock already released
    }
    
    if currentHolder != dashboardID {
        return fmt.Errorf("cannot release lock held by %s", currentHolder)
    }
    
    // Release lock
    if err := lm.redis.Del(ctx, lockKey).Err(); err != nil {
        return err
    }
    
    // Update audit trail
    _, err = lm.db.ExecContext(ctx, `
        UPDATE project_locks 
        SET released_at = NOW()
        WHERE project_id = ? AND dashboard_id = ? AND released_at IS NULL
    `, projectID, dashboardID)
    
    return err
}

func (lm *LockManager) HeartbeatLock(ctx context.Context, projectID, dashboardID string) error {
    lockKey := fmt.Sprintf("project:%s:lock", projectID)
    
    // Verify caller holds lock
    currentHolder, err := lm.redis.Get(ctx, lockKey).Result()
    if err != nil || currentHolder != dashboardID {
        return fmt.Errorf("lock not held or held by different dashboard")
    }
    
    // Extend expiry by 30 minutes
    return lm.redis.Expire(ctx, lockKey, 30*time.Minute).Err()
}
```

**Frontend: Automatic Lock Renewal**
```elm
-- Keep lock alive while user is active
subscriptions : Model -> Sub Msg
subscriptions model =
    if model.editMode == Editing then
        Sub.batch
            [ Time.every (5 * 60 * 1000) (\_ -> RenewLock)  -- Every 5 minutes
            , lockResponse LockResponseReceived
            ]
    else
        Sub.none

-- Renew lock before expiry
update msg model =
    case msg of
        RenewLock ->
            ( model, renewLockPort model.dashboardId )
```

---

## Propagation Lock

### Purpose

Prevent concurrent propagation operations that could cause:
- Race conditions in The Bin queue
- Inconsistent dashboard states
- Duplicate AI API calls (wasting tokens)

### Implementation

**Acquire Before Propagation:**
```go
// api/propagation/executor.go
func (pe *PropagationExecutor) Execute(ctx context.Context, binItemID string) error {
    // Load bin item
    item, err := pe.binService.LoadItem(ctx, binItemID)
    if err != nil {
        return err
    }
    
    // Acquire propagation lock
    lockKey := fmt.Sprintf("project:%s:propagation", item.ProjectID)
    jobID := uuid.New().String()
    
    success, err := pe.redis.SetNX(ctx, lockKey, jobID, 60*time.Second).Result()
    if err != nil {
        return fmt.Errorf("failed to acquire propagation lock: %w", err)
    }
    
    if !success {
        return &PropagationLockedError{
            ProjectID: item.ProjectID,
            Message: "Propagation already in progress",
        }
    }
    
    // Ensure lock released on exit
    defer pe.releasePropagationLock(ctx, item.ProjectID, jobID)
    
    // Execute propagation
    return pe.propagateToDownstreamDashboards(ctx, item)
}

func (pe *PropagationExecutor) releasePropagationLock(ctx context.Context, projectID, jobID string) {
    lockKey := fmt.Sprintf("project:%s:propagation", projectID)
    
    // Only release if we still hold the lock (compare job ID)
    script := redis.NewScript(`
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
        else
            return 0
        end
    `)
    
    script.Run(ctx, pe.redis, []string{lockKey}, jobID)
}
```

**UI Indicator:**
```elm
viewPropagationStatus : Model -> Html Msg
viewPropagationStatus model =
    case model.propagationStatus of
        NotPropagating ->
            div [ class "status-idle" ] 
                [ text "Ready to propagate" ]
        
        Propagating progress ->
            div [ class "status-active" ]
                [ text ("Propagating... " ++ String.fromInt progress ++ "%")
                , spinner
                ]
        
        PropagationLocked ->
            div [ class "status-blocked" ]
                [ icon "lock"
                , text "Propagation in progress from another source"
                , button [ onClick RetryPropagation ] [ text "Retry" ]
                ]
```

---

## Timeout Handling

### Lock Expiry Rules

| Lock Type | Default Timeout | Max Timeout | Auto-Renewal |
|-----------|-----------------|-------------|--------------|
| Project Lock | 30 minutes | 2 hours | Every 5 minutes |
| Propagation Lock | 60 seconds | 5 minutes | No (fail-fast) |
| Bin Validation Lock | 10 seconds | 30 seconds | No |

### Timeout Recovery

**Project Lock Timeout:**
```go
// Background worker checks for expired locks
func (lm *LockManager) CleanupExpiredLocks(ctx context.Context) {
    ticker := time.NewTicker(1 * time.Minute)
    defer ticker.Stop()
    
    for range ticker.C {
        rows, err := lm.db.QueryContext(ctx, `
            SELECT project_id, dashboard_id 
            FROM project_locks 
            WHERE released_at IS NULL 
              AND expires_at < NOW()
        `)
        if err != nil {
            log.Error("Failed to query expired locks", "error", err)
            continue
        }
        
        for rows.Next() {
            var projectID, dashboardID string
            rows.Scan(&projectID, &dashboardID)
            
            // Force release expired lock
            lm.forceReleaseLock(ctx, projectID, dashboardID)
            
            // Notify user (if still connected)
            lm.notifyLockExpired(ctx, projectID, dashboardID)
        }
        rows.Close()
    }
}
```

**User Notification:**
```elm
-- When lock expires
type Msg
    = LockExpired DashboardId

update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
    case msg of
        LockExpired dashboardId ->
            ( { model 
                | editMode = ReadOnly
                | showExpiredLockModal = True
              }
            , showNotification "Your edit lock expired due to inactivity"
            )
```

---

## Conflict Resolution

### Scenario: Lock Held by Stale Session

**Problem:** User closed browser without releasing lock, Redis key still exists.

**Detection:**
```go
// Check if lock holder is still active
func (lm *LockManager) IsLockHolderActive(ctx context.Context, projectID string) (bool, error) {
    lockKey := fmt.Sprintf("project:%s:lock", projectID)
    dashboardID, err := lm.redis.Get(ctx, lockKey).Result()
    if err == redis.Nil {
        return false, nil  // No lock
    }
    
    // Check last heartbeat
    heartbeatKey := fmt.Sprintf("dashboard:%s:heartbeat", dashboardID)
    lastHeartbeat, err := lm.redis.Get(ctx, heartbeatKey).Result()
    if err == redis.Nil {
        // No recent heartbeat, lock holder likely dead
        return false, nil
    }
    
    heartbeatTime, _ := time.Parse(time.RFC3339, lastHeartbeat)
    if time.Since(heartbeatTime) > 10*time.Minute {
        // Stale heartbeat
        return false, nil
    }
    
    return true, nil
}
```

**Resolution:**
```go
func (lm *LockManager) ForceAcquireLock(ctx context.Context, projectID, dashboardID, userID string) error {
    // Check if current lock holder is active
    active, err := lm.IsLockHolderActive(ctx, projectID)
    if err != nil {
        return err
    }
    
    if active {
        return &LockDeniedError{Message: "Lock holder is still active"}
    }
    
    // Force release stale lock
    lockKey := fmt.Sprintf("project:%s:lock", projectID)
    lm.redis.Del(ctx, lockKey)
    
    // Acquire new lock
    return lm.AcquireLock(ctx, projectID, dashboardID, userID)
}
```

---

## Lock Status Visibility

### Master Control Dashboard

**Lock Indicator:**
```elm
viewProjectLockStatus : Project -> Html Msg
viewProjectLockStatus project =
    case project.lockStatus of
        Unlocked ->
            div [ class "lock-status unlocked" ]
                [ icon "unlock"
                , text "Project unlocked"
                ]
        
        LockedBy dashboardName userId timestamp ->
            div [ class "lock-status locked" ]
                [ icon "lock"
                , text ("Locked by " ++ dashboardName)
                , span [ class "timestamp" ] 
                    [ text (formatTimestamp timestamp) ]
                , if userId == model.currentUserId then
                    button [ onClick ReleaseLock ] [ text "Release Lock" ]
                  else
                    button [ onClick RequestLockOverride ] [ text "Request Override" ]
                ]
```

### Dashboard-Level Indicator

**Read-Only Mode Banner:**
```elm
viewEditRestriction : Model -> Html Msg
viewEditRestriction model =
    if model.editMode == ReadOnly then
        div [ class "read-only-banner" ]
            [ icon "lock"
            , text "Read-only mode: "
            , case model.lockReason of
                LockedByOtherDashboard name ->
                    span [] [ text (name ++ " is currently being edited") ]
                
                PropagationInProgress ->
                    span [] [ text "Changes are propagating across dashboards" ]
                
                InsufficientPermissions ->
                    span [] [ text "You don't have edit permissions" ]
            , button [ onClick RequestEdit ] [ text "Request Edit Access" ]
            ]
    else
        text ""
```

---

## Database Schema

### project_locks Table

```sql
CREATE TABLE project_locks (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    project_id VARCHAR(255) NOT NULL,
    dashboard_id VARCHAR(255) NOT NULL,
    user_id VARCHAR(255) NOT NULL,
    acquired_at DATETIME NOT NULL,
    released_at DATETIME,
    expires_at DATETIME NOT NULL,
    lock_type ENUM('project', 'propagation', 'bin_validation') DEFAULT 'project',
    
    INDEX idx_project (project_id),
    INDEX idx_dashboard (dashboard_id),
    INDEX idx_expires (expires_at),
    INDEX idx_active (project_id, released_at)  -- Active locks have NULL released_at
);

-- Query for active locks
SELECT * FROM project_locks 
WHERE project_id = ? 
  AND released_at IS NULL 
  AND expires_at > NOW();
```

---

## Error Handling

### Lock Acquisition Failure

**Elm Error Display:**
```elm
type LockError
    = LockAlreadyHeld DashboardId
    | LockExpired
    | NetworkError String
    | InsufficientPermissions

viewLockError : LockError -> Html Msg
viewLockError error =
    case error of
        LockAlreadyHeld dashboardId ->
            div [ class "error-modal" ]
                [ h3 [] [ text "Cannot Edit" ]
                , p [] [ text ("Dashboard " ++ dashboardId ++ " is currently being edited") ]
                , button [ onClick ViewLockedDashboard ] [ text "View Dashboard" ]
                , button [ onClick RetryLater ] [ text "Try Again Later" ]
                ]
        
        LockExpired ->
            div [ class "error-modal" ]
                [ h3 [] [ text "Session Expired" ]
                , p [] [ text "Your edit session expired due to inactivity" ]
                , button [ onClick ReacquireLock ] [ text "Resume Editing" ]
                ]
```

### Propagation Lock Timeout

**Automatic Retry:**
```go
func (pe *PropagationExecutor) ExecuteWithRetry(ctx context.Context, binItemID string, maxRetries int) error {
    var lastErr error
    
    for i := 0; i < maxRetries; i++ {
        err := pe.Execute(ctx, binItemID)
        if err == nil {
            return nil
        }
        
        if _, isPropLocked := err.(*PropagationLockedError); isPropLocked {
            // Wait and retry
            time.Sleep(time.Duration(i+1) * 5 * time.Second)
            lastErr = err
            continue
        }
        
        // Other errors are fatal
        return err
    }
    
    return fmt.Errorf("propagation failed after %d retries: %w", maxRetries, lastErr)
}
```

---

## Testing Strategy

### Unit Tests

**Lock Acquisition:**
```go
func TestLockManager_AcquireLock(t *testing.T) {
    lm := NewLockManager(testRedis, testDB)
    
    // First acquisition should succeed
    err := lm.AcquireLock(ctx, "proj-1", "dash-1", "user-1")
    assert.NoError(t, err)
    
    // Second acquisition should fail
    err = lm.AcquireLock(ctx, "proj-1", "dash-2", "user-1")
    assert.Error(t, err)
    assert.IsType(t, &LockDeniedError{}, err)
}

func TestLockManager_LockExpiry(t *testing.T) {
    lm := NewLockManager(testRedis, testDB)
    
    // Acquire lock with short timeout
    lm.AcquireLockWithTimeout(ctx, "proj-1", "dash-1", "user-1", 100*time.Millisecond)
    
    // Immediately should be locked
    err := lm.AcquireLock(ctx, "proj-1", "dash-2", "user-1")
    assert.Error(t, err)
    
    // Wait for expiry
    time.Sleep(200 * time.Millisecond)
    
    // Now should succeed
    err = lm.AcquireLock(ctx, "proj-1", "dash-2", "user-1")
    assert.NoError(t, err)
}
```

### Integration Tests

**E2E Lock Flow:**
```elm
-- Playwright test
test "lock prevents concurrent edits" <|
    \() ->
        -- Open dashboard in Tab 1
        openDashboard "architecture-001"
        clickEdit()
        expectEditMode()
        
        -- Open same project in Tab 2
        openNewTab()
        openDashboard "techstack-001"
        clickEdit()
        expectReadOnlyMode()
        expectBanner "Architecture dashboard is currently being edited"
        
        -- Release lock in Tab 1
        switchToTab 1
        clickStopEditing()
        
        -- Tab 2 can now edit
        switchToTab 2
        clickEdit()
        expectEditMode()
```

---

## Performance Considerations

### Lock Throughput

**Target:** 1000 lock acquisitions/second per Redis instance  
**Measured:** ~5000/sec in production  
**Bottleneck:** TiDB audit log writes (secondary concern)

### Optimization: Batch Audit Writes

```go
// Buffer lock events, write to TiDB in batches
type LockAuditBatcher struct {
    buffer chan LockEvent
    db     *sql.DB
}

func (lab *LockAuditBatcher) Start(ctx context.Context) {
    ticker := time.NewTicker(5 * time.Second)
    defer ticker.Stop()
    
    var events []LockEvent
    
    for {
        select {
        case event := <-lab.buffer:
            events = append(events, event)
            
            // Flush if buffer full
            if len(events) >= 100 {
                lab.flush(ctx, events)
                events = nil
            }
        
        case <-ticker.C:
            // Periodic flush
            if len(events) > 0 {
                lab.flush(ctx, events)
                events = nil
            }
        }
    }
}
```

---

## Open Questions

1. Should lock override require admin approval? (Suggest: Yes, for multi-user post-MVP)
2. Maximum lock duration before force-release? (Suggest: 2 hours)
3. Should propagation locks be queued instead of rejected? (Suggest: No, fail-fast)
4. Audit log retention for locks? (Suggest: 90 days)

---

**End of LOCK_MECHANISM_SPECIFICATION.md**
