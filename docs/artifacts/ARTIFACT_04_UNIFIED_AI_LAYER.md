# Unified AI Layer Specification

**Document Type:** Core Architecture  
**Version:** 1.1.0  
**Date:** 2026-01-01T00:00:00Z  
**Dependencies:** ARTIFACT_03_THE_BIN_SPECIFICATION.md, ARTIFACT_05_DATA_MODEL.md, ARTIFACT_20_METADATA_GOVERNANCE_DASHBOARD.md

---

## Overview

The Unified AI Layer provides **just-in-time context and reasoning** across all dashboards via a **provider-agnostic interface**.

**MVP Default:** Claude Sonnet 4.5  
**Future:** Multi-provider support (local models, specialized providers)

---

## Provider Interface

```rust
#[async_trait]
pub trait AIProvider {
    async fn reason(&self, prompt: String, budget: TokenBudget) -> Result<AIResponse, AIError>;
    async fn generate_code(&self, spec: CodeSpec, budget: TokenBudget) -> Result<String, AIError>;
    async fn analyze_context(&self, context: ProjectContext) -> Result<ContextSummary, AIError>;
    fn name(&self) -> &str;
    fn supports_tools(&self) -> bool;
}

pub struct ClaudeProvider {
    api_key: String,
    base_url: String,
    model: String, // "claude-sonnet-4-20250514"
}

#[async_trait]
impl AIProvider for ClaudeProvider {
    async fn reason(&self, prompt: String, budget: TokenBudget) -> Result<AIResponse, AIError> {
        let request = build_claude_request(prompt, budget);
        let response = self.http_client.post(&self.base_url)
            .json(&request)
            .send()
            .await?;
        parse_claude_response(response).await
    }
    
    fn name(&self) -> &str { "claude-sonnet-4" }
    fn supports_tools(&self) -> bool { true }
}
```

---

## Timestamp Normalization in AI Context

**Challenge:** AI models may generate timestamps in various formats (local times, offset-based, inconsistent formatting). The Metadata Governance Dashboard requires all timestamps to use Zulu (UTC) format with `Z` suffix.

**Solution:** Post-process all AI responses to normalize timestamps before storage or display.

### Normalization Implementation

```rust
use chrono::{DateTime, Utc, SecondsFormat, FixedOffset};
use regex::{Regex, Captures};
use tracing::warn;

/// Timestamp normalizer for AI-generated content
pub struct TimestampNormalizer {
    /// Regex for ISO 8601 timestamps with timezone offset
    offset_regex: Regex,
    /// Regex for ISO 8601 timestamps without timezone (assumed local)
    naive_regex: Regex,
    /// Regex for non-standard date formats
    informal_regex: Regex,
}

impl TimestampNormalizer {
    pub fn new() -> Self {
        Self {
            // Matches: 2026-01-01T14:30:45+05:30 or 2026-01-01T14:30:45-08:00
            offset_regex: Regex::new(
                r"\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(\.\d{1,6})?[+-]\d{2}:\d{2}"
            ).unwrap(),
            
            // Matches: 2026-01-01T14:30:45 (no timezone - dangerous!)
            naive_regex: Regex::new(
                r"\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(\.\d{1,6})?(?![Z+-])"
            ).unwrap(),
            
            // Matches: January 1, 2026 or 01/01/2026 (informal formats)
            informal_regex: Regex::new(
                r"(?i)(January|February|March|April|May|June|July|August|September|October|November|December)\s+\d{1,2},?\s+\d{4}|\d{1,2}/\d{1,2}/\d{4}"
            ).unwrap(),
        }
    }
    
    /// Convert all timestamps in content to Zulu format
    pub fn normalize(&self, content: String) -> NormalizationResult {
        let mut result = content.clone();
        let mut conversions = Vec::new();
        
        // Step 1: Convert offset-based timestamps to Zulu
        result = self.offset_regex.replace_all(&result, |caps: &Captures| {
            let original = &caps[0];
            match DateTime::parse_from_rfc3339(original) {
                Ok(dt) => {
                    let zulu = dt.with_timezone(&Utc)
                        .to_rfc3339_opts(SecondsFormat::Millis, true);
                    conversions.push(TimestampConversion {
                        original: original.to_string(),
                        normalized: zulu.clone(),
                        conversion_type: ConversionType::OffsetToZulu,
                    });
                    zulu
                }
                Err(_) => original.to_string(),
            }
        }).to_string();
        
        // Step 2: Flag naive timestamps (no timezone) as warnings
        for caps in self.naive_regex.captures_iter(&content) {
            let original = &caps[0];
            conversions.push(TimestampConversion {
                original: original.to_string(),
                normalized: format!("{}Z", original), // Assume UTC
                conversion_type: ConversionType::NaiveAssumedUtc,
            });
        }
        result = self.naive_regex.replace_all(&result, |caps: &Captures| {
            format!("{}Z", &caps[0]) // Append Z suffix
        }).to_string();
        
        // Step 3: Log informal timestamps but don't convert
        for caps in self.informal_regex.captures_iter(&content) {
            warn!(
                "AI response contains informal timestamp '{}' - consider structured format",
                &caps[0]
            );
        }
        
        NormalizationResult {
            content: result,
            conversions,
            has_informal_timestamps: self.informal_regex.is_match(&content),
        }
    }
}

#[derive(Debug, Clone)]
pub struct NormalizationResult {
    pub content: String,
    pub conversions: Vec<TimestampConversion>,
    pub has_informal_timestamps: bool,
}

#[derive(Debug, Clone)]
pub struct TimestampConversion {
    pub original: String,
    pub normalized: String,
    pub conversion_type: ConversionType,
}

#[derive(Debug, Clone, PartialEq)]
pub enum ConversionType {
    OffsetToZulu,      // +05:30 → Z
    NaiveAssumedUtc,   // No TZ → assumed UTC with Z
    Unchanged,         // Already Zulu
}
```

### AI Provider Integration

```rust
use crate::metadata::{MetadataValidator, ValidationSeverity};
use chrono::{Utc, SecondsFormat};

/// Enhanced ClaudeProvider with timestamp normalization
impl AIProvider for ClaudeProvider {
    async fn complete(&self, request: AIRequest) -> Result<AIResponse, AIError> {
        // Step 1: Call the AI API
        let mut response = self.call_api(request).await?;
        
        // Step 2: Normalize all timestamps in AI response content
        let normalizer = TimestampNormalizer::new();
        let normalization = normalizer.normalize(response.content.clone());
        response.content = normalization.content;
        
        // Step 3: Log conversions for audit trail
        if !normalization.conversions.is_empty() {
            info!(
                "Normalized {} timestamps in AI response",
                normalization.conversions.len()
            );
            for conv in &normalization.conversions {
                debug!(
                    "Timestamp converted: '{}' → '{}' ({:?})",
                    conv.original, conv.normalized, conv.conversion_type
                );
            }
        }
        
        // Step 4: Set response metadata with Zulu timestamp
        response.created_at = Utc::now().to_rfc3339_opts(SecondsFormat::Millis, true);
        response.normalized_timestamps = normalization.conversions.len();
        
        // Step 5: Optional metadata validation (log violations, don't block)
        if let Some(validator) = &self.metadata_validator {
            match validator.validate_content(&response.content).await {
                Ok(result) if result.has_warnings() => {
                    warn!("AI response has metadata warnings: {:?}", result.warnings());
                }
                Err(e) => {
                    warn!("Metadata validation error (non-blocking): {}", e);
                }
                _ => {}
            }
        }
        
        Ok(response)
    }
    
    fn normalize_timestamps(&self, content: String) -> String {
        let normalizer = TimestampNormalizer::new();
        normalizer.normalize(content).content
    }
}

/// Extended AIResponse with timestamp metadata
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct AIResponse {
    pub content: String,
    pub created_at: String,              // Always Zulu: 2026-01-01T14:30:45.123Z
    pub model: String,
    pub tokens_used: u32,
    pub normalized_timestamps: usize,    // Count of timestamps normalized
    pub finish_reason: FinishReason,
}
```

### Validation Against Metadata Standards

```rust
/// Integration with Metadata Governance Dashboard
pub struct MetadataAwareAIProvider {
    inner_provider: Box<dyn AIProvider>,
    validator: MetadataValidator,
    db: TiDBClient,
}

impl MetadataAwareAIProvider {
    /// Validate AI response against active metadata standards
    pub async fn validate_and_log(&self, response: &AIResponse) -> Result<(), MetadataError> {
        // Query active timestamp standards
        let standards = self.db.query(
            "SELECT * FROM metadata_standards WHERE category = 'temporal' AND is_active = TRUE"
        ).await?;
        
        for standard in standards {
            let regex = Regex::new(&standard.pattern_regex)?;
            
            // Find all timestamps in response
            for timestamp in extract_timestamps(&response.content) {
                if !regex.is_match(&timestamp) {
                    // Log violation (but don't block AI responses)
                    self.db.execute(
                        "INSERT INTO metadata_violations 
                         (id, standard_id, resource_type, resource_id, violation_value, 
                          suggested_fix, status, detected_at)
                         VALUES (?, ?, 'ai_response', ?, ?, ?, 'open', ?)",
                        &[
                            &Uuid::new_v4().to_string(),
                            &standard.id,
                            &response.id,
                            &timestamp,
                            &format!("Convert to Zulu format: {}Z", timestamp),
                            &Utc::now().to_rfc3339(),
                        ]
                    ).await?;
                }
            }
        }
        
        Ok(())
    }
}
```

### Test Cases for Timestamp Normalization

```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_offset_to_zulu_conversion() {
        let normalizer = TimestampNormalizer::new();
        let input = "Meeting scheduled for 2026-01-01T14:30:45+05:30 in Mumbai.";
        let result = normalizer.normalize(input.to_string());
        
        assert!(result.content.contains("2026-01-01T09:00:45.000Z"));
        assert_eq!(result.conversions.len(), 1);
        assert_eq!(result.conversions[0].conversion_type, ConversionType::OffsetToZulu);
    }
    
    #[test]
    fn test_naive_timestamp_gets_z_suffix() {
        let normalizer = TimestampNormalizer::new();
        let input = "Created at 2026-01-01T14:30:45 without timezone.";
        let result = normalizer.normalize(input.to_string());
        
        assert!(result.content.contains("2026-01-01T14:30:45Z"));
        assert_eq!(result.conversions[0].conversion_type, ConversionType::NaiveAssumedUtc);
    }
    
    #[test]
    fn test_already_zulu_unchanged() {
        let normalizer = TimestampNormalizer::new();
        let input = "Deployed at 2026-01-01T14:30:45.123Z successfully.";
        let result = normalizer.normalize(input.to_string());
        
        assert_eq!(result.content, input);
        assert!(result.conversions.is_empty());
    }
    
    #[test]
    fn test_multiple_timestamps_in_content() {
        let normalizer = TimestampNormalizer::new();
        let input = "Start: 2026-01-01T09:00:00+00:00, End: 2026-01-01T17:00:00-08:00";
        let result = normalizer.normalize(input.to_string());
        
        // Both should be Zulu format
        assert!(result.content.contains("Z"));
        assert!(!result.content.contains("+00:00"));
        assert!(!result.content.contains("-08:00"));
        assert_eq!(result.conversions.len(), 2);
    }
    
    #[test]
    fn test_informal_timestamp_flagged() {
        let normalizer = TimestampNormalizer::new();
        let input = "Released on January 1, 2026 with great fanfare.";
        let result = normalizer.normalize(input.to_string());
        
        assert!(result.has_informal_timestamps);
        // Content unchanged (informal timestamps not auto-converted)
        assert!(result.content.contains("January 1, 2026"));
    }
}
```

---

## Context Management

**Challenge:** Maintain conversation continuity across dashboard switches

**Solution:** Sliding Window Context

```rust
pub struct ContextWindow {
    project_id: String,
    conversation_history: VecDeque<Message>,
    max_tokens: usize,  // 200K for Claude
    current_tokens: usize,
}

impl ContextWindow {
    pub fn add_message(&mut self, msg: Message) {
        let tokens = estimate_tokens(&msg);
        
        // Evict oldest messages if over budget
        while self.current_tokens + tokens > self.max_tokens {
            if let Some(removed) = self.conversation_history.pop_front() {
                self.current_tokens -= estimate_tokens(&removed);
            } else {
                break;
            }
        }
        
        self.conversation_history.push_back(msg);
        self.current_tokens += tokens;
    }
    
    pub fn get_messages_for_request(&self, dashboard: &str) -> Vec<Message> {
        // Include system prompt + recent history + dashboard-specific context
        let mut messages = vec![build_system_prompt(dashboard)];
        messages.extend(self.conversation_history.iter().cloned());
        messages
    }
}
```

**Persistence:** Context window saved to TiDB on dashboard switch

---

## Token Budget Enforcement

```rust
pub struct TokenBudget {
    session_limit: u32,      // 100,000 tokens/hour
    session_used: Arc<AtomicU32>,
    dashboard_limit: u32,    // 10,000 tokens/request
    tool_call_limit: u32,    // 4,000 tokens/call
}

impl TokenBudget {
    pub fn consume(&self, tokens: u32) -> Result<(), BudgetExceeded> {
        let current = self.session_used.fetch_add(tokens, Ordering::SeqCst);
        if current + tokens > self.session_limit {
            Err(BudgetExceeded {
                limit: self.session_limit,
                requested: tokens,
                remaining: self.session_limit.saturating_sub(current),
            })
        } else {
            Ok(())
        }
    }
    
    pub fn reset_session(&self) {
        self.session_used.store(0, Ordering::SeqCst);
    }
}
```

**UI Integration:**
```elm
-- Display token budget in dashboard header
viewTokenBudget : TokenBudget -> Html Msg
viewTokenBudget budget =
    div [ class "token-budget-indicator" ]
        [ text <| String.fromInt budget.remaining ++ " tokens remaining"
        , progress [ value (String.fromInt budget.used), max (String.fromInt budget.limit) ] []
        ]
```

---

## Token Usage Tracking (TiDB Integration)

All AI requests are logged to the `token_usage` table for analytics and cost tracking.

```rust
use sqlx::TiDB;

/// Log token usage to TiDB for analytics
pub async fn log_token_usage(
    db: &TiDBClient,
    request: &AIRequest,
    response: &AIResponse,
) -> Result<(), DbError> {
    let usage = TokenUsageRecord {
        id: Uuid::new_v4().to_string(),
        provider: response.model.clone(),
        input_tokens: request.estimated_tokens,
        output_tokens: response.tokens_used,
        total_tokens: request.estimated_tokens + response.tokens_used,
        cost_cents: calculate_cost(&response.model, request.estimated_tokens, response.tokens_used),
        dashboard_context: request.dashboard.clone(),
        recorded_at: Utc::now().to_rfc3339_opts(SecondsFormat::Micros, true),
    };
    
    db.execute(
        "INSERT INTO token_usage 
         (id, provider, input_tokens, output_tokens, total_tokens, 
          cost_cents, dashboard_context, recorded_at)
         VALUES (?, ?, ?, ?, ?, ?, ?, ?)",
        &[
            &usage.id,
            &usage.provider,
            &usage.input_tokens,
            &usage.output_tokens,
            &usage.total_tokens,
            &usage.cost_cents,
            &usage.dashboard_context,
            &usage.recorded_at,
        ]
    ).await?;
    
    Ok(())
}

/// Calculate cost in cents based on provider pricing
fn calculate_cost(model: &str, input_tokens: u32, output_tokens: u32) -> u32 {
    match model {
        "claude-sonnet-4-20250514" => {
            // $3/M input, $15/M output (example pricing)
            let input_cost = (input_tokens as f64 / 1_000_000.0) * 300.0;
            let output_cost = (output_tokens as f64 / 1_000_000.0) * 1500.0;
            (input_cost + output_cost) as u32
        }
        _ => 0, // Unknown model
    }
}
```

---

## Guardrail Validation via AI

**Example: Single Responsibility Principle Check**

```rust
pub async fn validate_srp(change: &CodeChange, provider: &dyn AIProvider) -> Result<SRPResult, AIError> {
    let prompt = format!(
        "Analyze if this change violates Single Responsibility Principle:\n\n\
         File: {}\n\
         Changes:\n{}\n\n\
         Respond with JSON: {{\"violates\": bool, \"reason\": str, \"suggestions\": [str]}}",
        change.file_path,
        change.diff
    );
    
    let response = provider.reason(prompt, TokenBudget::from(4000)).await?;
    parse_srp_result(&response.content)
}

#[derive(Deserialize)]
struct SRPResult {
    violates: bool,
    reason: String,
    suggestions: Vec<String>,
}
```

**Integration with The Bin:**
- AI validation runs as part of guardrail checks
- Results displayed in Bin detail view
- User can override with justification (logged for audit)

---

## Offline Caching Strategy

**Problem:** AI unavailable offline

**Solution:** Cache common responses

```rust
pub struct AICache {
    cache: Arc<RwLock<HashMap<String, CachedResponse>>>,
    ttl: Duration,  // 7 days
}

pub struct CachedResponse {
    prompt_hash: String,
    response: AIResponse,
    created_at: SystemTime,
    hit_count: u32,
}

impl AICache {
    pub async fn get_or_fetch(
        &self, 
        prompt: &str, 
        provider: &dyn AIProvider,
        budget: TokenBudget
    ) -> Result<AIResponse, AIError> {
        let hash = hash_prompt(prompt);
        
        // Try cache first
        if let Some(cached) = self.cache.read().await.get(&hash) {
            if cached.created_at.elapsed().unwrap() < self.ttl {
                return Ok(cached.response.clone());
            }
        }
        
        // Fetch from provider
        let response = provider.reason(prompt.to_string(), budget).await?;
        
        // Update cache
        self.cache.write().await.insert(hash.clone(), CachedResponse {
            prompt_hash: hash,
            response: response.clone(),
            created_at: SystemTime::now(),
            hit_count: 0,
        });
        
        Ok(response)
    }
}
```

**Cache Storage:** IndexedDB (browser) + TiDB (server)

---

## Dashboard-Specific Prompts

**Template:**

```yaml
dashboard_prompts:
  vibe_coding_inception:
    system: |
      You are assisting with project inception phase.
      Help user define: scope, goals, constraints, success criteria.
      Ask clarifying questions. Suggest SMART objectives.
    
  vibe_coding_implementation:
    system: |
      You are assisting with implementation phase.
      Review code for: clarity, correctness, test coverage.
      Suggest refactorings aligned with project guardrails.
```

**Dynamic Injection:**

```rust
fn build_system_prompt(dashboard: &str) -> Message {
    let template = load_prompt_template(dashboard);
    let project_context = load_project_context();
    
    Message {
        role: Role::System,
        content: format!(
            "{}\n\nProject Context:\n{}\n\nActive Guardrails:\n{}",
            template.system,
            project_context.summary,
            format_guardrails(&project_context.guardrails)
        ),
    }
}
```

---

## Tool Use (Future)

**Example: GitHub Integration**

```rust
pub struct GitHubTool;

#[async_trait]
impl AITool for GitHubTool {
    fn name(&self) -> &str { "github_create_issue" }
    
    fn schema(&self) -> ToolSchema {
        ToolSchema {
            name: "github_create_issue".to_string(),
            description: "Create a GitHub issue in the project repository".to_string(),
            parameters: json!({
                "type": "object",
                "properties": {
                    "title": { "type": "string" },
                    "body": { "type": "string" },
                    "labels": { "type": "array", "items": { "type": "string" } }
                },
                "required": ["title", "body"]
            }),
        }
    }
    
    async fn execute(&self, params: serde_json::Value) -> Result<ToolResult, ToolError> {
        let title = params["title"].as_str().ok_or(ToolError::InvalidParams)?;
        let body = params["body"].as_str().ok_or(ToolError::InvalidParams)?;
        
        // Call GitHub API
        let issue_url = github_api::create_issue(title, body).await?;
        
        Ok(ToolResult {
            success: true,
            output: json!({ "issue_url": issue_url }),
        })
    }
}
```

**Claude Tool Use Pattern:**

```json
{
  "model": "claude-sonnet-4-20250514",
  "max_tokens": 4000,
  "tools": [
    {
      "name": "github_create_issue",
      "description": "Create a GitHub issue",
      "input_schema": {
        "type": "object",
        "properties": {
          "title": { "type": "string" },
          "body": { "type": "string" }
        },
        "required": ["title", "body"]
      }
    }
  ],
  "messages": [
    { "role": "user", "content": "Create an issue to track bug #123" }
  ]
}
```

---

## Performance Optimization

**Streaming Responses:**

```rust
pub async fn reason_streaming(
    &self, 
    prompt: String, 
    callback: impl Fn(String) + Send + Sync
) -> Result<AIResponse, AIError> {
    let stream = self.http_client
        .post(&self.base_url)
        .json(&build_request(prompt))
        .send()
        .await?
        .bytes_stream();
    
    let mut accumulated = String::new();
    
    while let Some(chunk) = stream.next().await {
        let text = parse_sse_chunk(chunk?)?;
        accumulated.push_str(&text);
        callback(text);  // Call UI update callback
    }
    
    Ok(AIResponse { content: accumulated })
}
```

**Elm Integration:**

```elm
-- Subscription for streaming AI responses
subscriptions : Model -> Sub Msg
subscriptions model =
    aiResponseStream AIChunkReceived
```

---

## Security Considerations

**API Key Management:**
- Never stored in Elm code or localStorage
- Fetched from Go API on session start
- Rotated monthly (automated)
- Stored encrypted in `api_configurations` table

**PII Filtering:**

```rust
pub fn sanitize_context(context: &str) -> String {
    let mut sanitized = context.to_string();
    
    // Redact emails
    sanitized = regex::Regex::new(r"[\w\.-]+@[\w\.-]+\.\w+")
        .unwrap()
        .replace_all(&sanitized, "[EMAIL REDACTED]")
        .to_string();
    
    // Redact API keys
    sanitized = regex::Regex::new(r"(api[_-]?key|token)[\s:=]+[\w-]+")
        .unwrap()
        .replace_all(&sanitized, "[API_KEY REDACTED]")
        .to_string();
    
    sanitized
}
```

---

## Graceful Degradation

**Provider Unavailable:**

```rust
pub async fn reason_with_fallback(
    &self, 
    prompt: String, 
    budget: TokenBudget
) -> Result<AIResponse, AIError> {
    // Try primary provider (Claude)
    match self.primary_provider.reason(prompt.clone(), budget).await {
        Ok(response) => Ok(response),
        Err(e) if e.is_rate_limit() => {
            // Try cached response
            if let Some(cached) = self.cache.get(&hash_prompt(&prompt)).await {
                return Ok(cached.response);
            }
            
            // Fallback to secondary provider (local model)
            self.secondary_provider.reason(prompt, budget).await
        },
        Err(e) => Err(e),
    }
}
```

**Degraded Experience:**
- AI assistance disabled
- Show "AI unavailable" banner
- Queue AI requests for retry

---

## Cross-References

| Artifact | Integration Point |
|----------|-------------------|
| ARTIFACT_03 (The Bin) | Metadata validation for AI responses |
| ARTIFACT_05 (Data Model) | `token_usage`, `api_configurations` tables |
| ARTIFACT_06 (Master Control) | AI Cortex panel embedding |
| ARTIFACT_13 (Security) | API key encryption, PII filtering |
| ARTIFACT_20 (Metadata) | Timestamp normalization standards |

---

## Changelog

### v1.1.0 (2026-01-01T00:00:00Z)
- **Added:** Timestamp Normalization in AI Context section
- **Added:** `TimestampNormalizer` struct with Zulu conversion
- **Added:** Integration with Metadata Governance Dashboard
- **Added:** Token usage tracking with TiDB
- **Added:** Test cases for timestamp normalization
- **Added:** Cross-references table
- **Updated:** Dependencies to include ARTIFACT_20_METADATA_GOVERNANCE_DASHBOARD.md
- **Updated:** Date format to Zulu (Z suffix)

### v1.0.0 (2026-01-01)
- Initial specification release

---

**End of ARTIFACT_04_UNIFIED_AI_LAYER.md**
