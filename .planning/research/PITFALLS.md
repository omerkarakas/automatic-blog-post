# Domain Pitfalls: n8n-Based Automated Blog Content Generation

**Domain:** Automated blog content generation with n8n workflows
**Researched:** 2026-02-15
**Confidence:** HIGH (based on n8n architecture, OpenRouter API specifications, WordPress REST API, and AI content generation patterns)

## Critical Pitfalls

Mistakes that cause rewrites, data loss, or major production issues.

### Pitfall 1: n8n Default Parameter Values Cause Runtime Failures

**What goes wrong:** Nodes configured with missing required parameters fail silently or produce incorrect results at runtime, even though they appear valid in the workflow builder.

**Why it happens:** n8n's workflow builder doesn't validate all required parameters before execution. The CLAUDE.md explicitly warns: "Default parameter values are the #1 source of runtime failures."

**Real-world example:**
```json
// This LOOKS valid but FAILS at runtime
{
  "resource": "message",
  "operation": "post",
  "text": "Blog published"
}

// Missing required 'select' and 'channelId' parameters
// Fails with cryptic error during execution
```

**Consequences:**
- Workflows succeed in test but fail in production
- Silent failures (no error notification)
- Partial executions that corrupt data state
- Hours spent debugging seemingly-valid configurations

**Prevention:**
- ALWAYS explicitly set ALL parameters that control node behavior
- Use `validate_node({mode: 'full', profile: 'runtime'})` before deployment
- Never rely on default values, even for "optional" parameters
- Test with real data, not placeholder values

**Detection:**
- Workflow succeeds in manual test but fails when triggered automatically
- Error messages mention "missing required field" despite field appearing configured
- Inconsistent behavior between executions

**Phase mapping:** Address in Phase 1 (Foundation) - establish validation protocol before building complex workflows.

---

### Pitfall 2: OpenRouter API Rate Limits and Model Quota Exhaustion

**What goes wrong:** Batch content generation hits OpenRouter rate limits, causing workflow failures mid-execution. Worse, model quotas exhaust without warning, switching to fallback models with different output formats.

**Why it happens:**
- OpenRouter imposes per-model rate limits (requests/min and tokens/min)
- Different models have different quotas
- No built-in queuing in n8n for API calls
- Batch operations (e.g., "generate 20 cluster posts") make rapid sequential calls

**Real-world scenario:**
```
Topic Planner generates 15 cluster topics → Content Generator loops through all 15
- Calls 1-10: Claude 3.5 Sonnet (success)
- Call 11: Rate limit hit (429 error)
- Calls 12-15: Auto-fallback to GPT-4 Turbo
- GPT-4 returns different JSON structure
- WordPress publishing fails due to schema mismatch
```

**Consequences:**
- Partial content generation (some posts created, others failed)
- Inconsistent content quality (mixed models)
- Data corruption in Google Sheets (partial updates)
- Wasted API credits on failed executions
- Difficult to resume from failure point

**Prevention:**
- Implement exponential backoff with retry logic (n8n's built-in retry with 2s, 4s, 8s delays)
- Add rate limiting node between AI calls (Sleep node with calculated delays)
- Use n8n's "Split in Batches" node with small batch sizes (3-5 items)
- Set execution timeout protection (max 5 minutes per API call)
- Monitor token usage per execution (log in Google Sheets)
- Configure model-specific fallback chains with schema validation

**Implementation pattern:**
```
Loop Items → Sleep (2s) → HTTP Request (OpenRouter)
→ IF (status=429) → Sleep (60s) → Retry
→ IF (status=200) → Validate JSON Schema → Continue
```

**Detection:**
- Execution history shows 429 HTTP errors
- Google Sheets has gaps in generated content IDs
- Content quality suddenly changes mid-batch
- Error notifications mention "rate limit exceeded"

**Phase mapping:**
- Phase 2 (Content Generator MVP): Implement basic retry logic
- Phase 3 (Quality Controls): Add schema validation and fallback chains
- Phase 4 (Multi-Site): Scale batch processing with advanced queuing

---

### Pitfall 3: WordPress HTML Formatting Corruption

**What goes wrong:** AI-generated content contains valid Markdown but breaks when converted to WordPress HTML. Special characters, code blocks, lists, and Turkish characters render incorrectly or cause publishing failures.

**Why it happens:**
- AI models generate Markdown (Claude, GPT-4)
- WordPress expects sanitized HTML
- n8n's HTTP Request node doesn't auto-convert Markdown
- Turkish characters (ı, ş, ğ, ü, ö, ç) require UTF-8 encoding
- WordPress REST API has strict HTML sanitization rules

**Real-world example:**
```markdown
AI generates:
"İçerik pazarlama stratejisi **çok önemli**. Liste:
- Anahtar kelime araştırması
- Backlink analizi"

WordPress receives (if not properly encoded):
"Ã‡erik pazarlama stratejisi **Ã§ok Ã¶nemli**. Liste:
- Anahtar kelime araÅŸtÄ±rmasÄ±
- Backlink analizi"

Result: Garbled content, broken formatting, reader complaints
```

**Consequences:**
- Published content unreadable
- SEO damaged (garbled text in meta descriptions)
- Manual editing required (defeats automation purpose)
- Brand reputation damage
- Google indexing issues

**Prevention:**
- Add dedicated Markdown→HTML conversion node (use Code node with marked.js)
- Set UTF-8 encoding explicitly in HTTP Request headers: `Content-Type: application/json; charset=utf-8`
- Sanitize HTML before WordPress API call (remove unsupported tags)
- Validate Turkish characters with regex: `/[ıİğĞüÜşŞöÖçÇ]/u`
- Test with Turkish content samples during development
- Use WordPress preview endpoint before final publish

**Implementation:**
```javascript
// Code node: Markdown to WordPress HTML
const marked = require('marked');

// Configure for WordPress compatibility
marked.setOptions({
  gfm: true, // GitHub Flavored Markdown
  breaks: true, // Convert \n to <br>
  sanitize: false, // We'll sanitize separately
});

let html = marked($json.content);

// Sanitize for WordPress
html = html
  .replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '') // Remove scripts
  .replace(/<iframe\b[^<]*(?:(?!<\/iframe>)<[^<]*)*<\/iframe>/gi, ''); // Remove iframes

return {html};
```

**Detection:**
- WordPress preview shows broken formatting
- Published posts contain � characters
- Turkish characters display as Latin equivalents
- Lists render as plain text
- Bold/italic formatting missing

**Phase mapping:** Phase 2 (Content Generator MVP) - establish HTML conversion before first publish.

---

### Pitfall 4: Google Sheets Concurrent Access and Race Conditions

**What goes wrong:** Multiple workflow executions try to update the same Google Sheet simultaneously, causing data overwrites, duplicate rows, or API errors.

**Why it happens:**
- Topic Planner and Content Generator may run concurrently
- Google Sheets API has row-level locking issues
- n8n doesn't provide built-in concurrency control
- Multiple sites publishing simultaneously create contention

**Real-world scenario:**
```
Time 10:00:00 - Topic Planner starts, reads Sheet (last row: 50)
Time 10:00:02 - Content Generator triggered for Site A, reads Sheet (last row: 50)
Time 10:00:03 - Topic Planner appends row 51 (new topic)
Time 10:00:05 - Content Generator appends row 51 (new post) - OVERWRITES topic
Time 10:00:06 - Both workflows report success
Result: Lost topic data, duplicate row 51, broken tracking
```

**Consequences:**
- Data loss (overwritten rows)
- Duplicate content entries
- Broken topic→post linking
- Audit trail corruption
- Manual reconciliation required

**Prevention:**
- Use Google Sheets "Append" operation (never "Update by row number")
- Implement row-level locking with dedicated "Lock" sheet column
- Add workflow execution ID to each row (prevents confusion)
- Use atomic operations: Read → Modify → Write in single node
- Implement optimistic locking: check timestamp before update
- Consider dedicated queue sheet for multi-workflow coordination

**Locking pattern:**
```
1. Read current max ID from Sheet
2. Write "LOCK:workflow-id:timestamp" to Lock column
3. Wait 1 second
4. Read Lock column again
5. IF Lock contains our workflow-id → Proceed
6. ELSE → Sleep 5s → Retry from step 1
7. After update → Clear Lock column
```

**Detection:**
- Sheets contain duplicate IDs
- Row counts don't match expected workflow executions
- Timestamps show overlapping write operations
- Error logs show "concurrent modification" or "row already updated"

**Phase mapping:**
- Phase 1 (Foundation): Establish append-only pattern
- Phase 4 (Multi-Site): Implement full locking protocol

---

### Pitfall 5: AI Hallucination and Factual Errors in Turkish Content

**What goes wrong:** AI models generate confident-sounding but factually incorrect content, especially for Turkish-specific topics, company names, statistics, or technical terms.

**Why it happens:**
- Claude and GPT-4 have less Turkish training data than English
- Models "fill in gaps" with plausible-sounding invented facts
- Turkish technical terminology often borrowed from English (inconsistent translation)
- No built-in fact-checking in LLM workflows
- Perplexity (grounded search) may still hallucinate citations

**Real-world examples:**
```
Hallucination type 1 - Invented statistics:
"Türkiye'de SEO pazarı 2023'te 450 milyon TL büyüklüğe ulaştı"
(No such statistic exists; model invented plausible number)

Hallucination type 2 - Wrong company names:
"Google'ın Türkiye ofisi İstanbul Maslak'ta bulunuyor"
(Google Turkey office is in different location; model guessed)

Hallucination type 3 - Fake tool features:
"WordPress 6.4'te otomatik AI içerik üretimi özelliği eklendi"
(No such feature; model extrapolated from trends)
```

**Consequences:**
- Published misinformation damages credibility
- SEO penalties for thin/incorrect content
- Legal liability for false claims
- Reader complaints and lost trust
- Manual fact-checking required (negates automation value)

**Prevention:**
- Use Perplexity API for factual claims (provides citations)
- Implement multi-model validation: Claude generates → GPT-4 fact-checks
- Add human review queue for statistical claims
- Maintain Turkish terminology glossary (consistent terms)
- Set conservative temperature (0.3-0.5) for factual content
- Use prompt engineering: "Only include verifiable facts. If uncertain, omit."
- Add disclaimer: "AI-generated content, verify important claims"

**Fact-checking workflow pattern:**
```
1. Claude generates content
2. Extract claims with regex (numbers, dates, company names)
3. Perplexity API: verify each claim
4. IF claim has citation → Keep
5. IF claim unsupported → Flag for review OR remove
6. Log all fact-check results in Google Sheets
```

**Detection:**
- Content contains statistics without sources
- Turkish technical terms inconsistent across posts
- Reader comments point out errors
- Competitors cite your content as misinformation example

**Phase mapping:**
- Phase 3 (Quality Controls): Implement fact-checking pipeline
- Phase 5 (Monitoring): Add post-publish validation

---

### Pitfall 6: SEO Keyword Cannibalization Between Pillar and Cluster Posts

**What goes wrong:** Pillar post and cluster posts target overlapping keywords, causing them to compete in search results. Google can't determine which page to rank, resulting in ALL pages ranking lower.

**Why it happens:**
- AI models naturally generate related keywords across posts
- Pillar-cluster strategy requires careful keyword segregation
- No automated keyword conflict detection
- Internal linking creates false equivalence signals
- Turkish language has fewer synonym variations (higher cannibalization risk)

**Real-world scenario:**
```
Pillar post: "SEO Nedir? Kapsamlı Rehber"
Target keyword: "SEO nedir"

Cluster post 1: "SEO Teknikleri ve Stratejileri"
AI-generated title includes: "SEO Nedir ve Nasıl Uygulanır?"
CONFLICT: Both target "SEO nedir"

Cluster post 2: "Anahtar Kelime Araştırması Rehberi"
AI-generated intro: "SEO nedir, öncelikle anahtar kelime araştırması..."
CONFLICT: Pillar keyword in cluster intro

Result: Google sees 3 pages about "SEO nedir", ranks none in top 10
```

**Consequences:**
- Lower search rankings for all related content
- Wasted content production effort
- Confused site architecture
- Internal link value diluted
- Difficult to fix without republishing/redirects

**Prevention:**
- Maintain keyword mapping in Google Sheets (Pillar → Cluster assignments)
- Validate each generated title/H1 against existing content
- Use keyword exclusion lists in AI prompts: "Do NOT use these exact phrases: [pillar keywords]"
- Implement automated conflict detection before publish
- Use semantic variation prompts: "Use synonyms and long-tail variations only"
- Set strict internal linking rules: Cluster → Pillar only (never Cluster ↔ Cluster)

**Conflict detection implementation:**
```javascript
// Code node: Detect keyword cannibalization
const newTitle = $json.title.toLowerCase();
const newH1 = $json.h1.toLowerCase();
const existingKeywords = $node["Google Sheets"].json.map(row =>
  row.target_keyword.toLowerCase()
);

const conflicts = existingKeywords.filter(kw =>
  newTitle.includes(kw) || newH1.includes(kw)
);

if (conflicts.length > 0) {
  return {
    conflict: true,
    conflicting_keywords: conflicts,
    action: "REJECT - Regenerate with different angle"
  };
}

return {conflict: false};
```

**Detection:**
- Google Search Console shows multiple URLs for same query
- Pillar posts not ranking despite quality content
- Search impressions spread across similar posts
- Ahrefs/SEMrush shows keyword conflicts
- Internal search brings up duplicate topics

**Phase mapping:**
- Phase 1 (Topic Planner): Establish keyword segregation rules
- Phase 3 (Quality Controls): Implement automated conflict detection
- Phase 5 (Monitoring): Track keyword rankings per post

---

## Moderate Pitfalls

Mistakes that cause delays, technical debt, or require refactoring.

### Pitfall 7: n8n Memory Limits in Large Batch Operations

**What goes wrong:** Workflows processing large batches (20+ posts) exceed n8n's memory limits, causing execution crashes without clear error messages.

**Why it happens:**
- n8n stores entire execution data in memory
- Each AI response (2000-4000 tokens) accumulates
- Image generation adds binary data to memory
- No automatic memory cleanup between loop iterations

**Prevention:**
- Use "Split in Batches" with max 5 items per batch
- Clear execution data with Code node: `return [{minimal: 'data'}];`
- Process images separately (don't store in main workflow memory)
- Set workflow-level memory limits
- Use sub-workflow execution for isolated memory spaces

**Detection:**
- Workflow stops mid-execution without error
- Execution history shows "Unknown error"
- Small batches succeed, large batches fail
- Memory usage alerts in n8n instance

**Phase mapping:** Phase 4 (Multi-Site) - address when scaling batch sizes.

---

### Pitfall 8: WordPress Media Upload Failures and Orphaned Images

**What goes wrong:** Image generation succeeds but WordPress upload fails, leaving orphaned images and posts without featured images.

**Why it happens:**
- WordPress media API requires different authentication than post API
- Image size limits (default 2MB, configurable)
- Network timeouts for large images
- MIME type mismatches
- No retry logic for media uploads

**Prevention:**
- Upload media BEFORE creating post (separate API calls)
- Validate image size before upload (<1.5MB recommended)
- Set proper Content-Type headers
- Implement retry with exponential backoff
- Store media IDs in Google Sheets for recovery
- Use WordPress's image optimization (resize on upload)

**Recovery pattern:**
```
1. Generate image → Save to temporary storage (URL)
2. Upload to WordPress Media Library → Get media ID
3. Create post with featured_media: media_id
4. IF upload fails → Log image URL in Sheets → Set flag "pending_image"
5. Retry workflow: Query Sheets for "pending_image" → Attempt upload
```

**Detection:**
- Posts published without featured images
- WordPress media library has gaps in IDs
- Google Sheets has image URLs but no media IDs

**Phase mapping:** Phase 2 (Content Generator MVP) - implement robust media handling.

---

### Pitfall 9: OpenRouter JSON Response Parsing Failures

**What goes wrong:** AI models occasionally return malformed JSON, breaking downstream nodes that expect structured data.

**Why it happens:**
- Models sometimes include explanatory text before/after JSON
- JSON escaping issues with Turkish characters (quotes in content)
- Inconsistent JSON formatting across models
- No schema enforcement in prompt alone

**Real-world examples:**
```
Expected:
{"title": "SEO Rehberi", "content": "..."}

Model returns:
"Here's the JSON you requested:
{"title": "SEO Rehberi", "content": "..."}
Hope this helps!"

OR:

{"title": "SEO "Kapsamlı" Rehberi", "content": "..."}
// Unescaped quotes break parsing
```

**Prevention:**
- Use JSON schema in system prompt with explicit examples
- Add JSON extraction Code node: regex `/\{[\s\S]*\}/` to find JSON block
- Validate with try-catch before downstream nodes
- Use Perplexity's structured output mode (enforces JSON)
- Set response format parameter in OpenRouter request: `response_format: {"type": "json_object"}`
- Escape Turkish quotes in pre-processing

**Extraction + Validation pattern:**
```javascript
// Code node: Extract and validate JSON
let raw = $json.response;

// Extract JSON block (handles surrounding text)
const jsonMatch = raw.match(/\{[\s\S]*\}/);
if (!jsonMatch) {
  return {error: "No JSON found", raw};
}

let parsed;
try {
  parsed = JSON.parse(jsonMatch[0]);
} catch (e) {
  // Try fixing common issues
  let fixed = jsonMatch[0]
    .replace(/[\u201C\u201D]/g, '"') // Smart quotes → regular
    .replace(/[\u2018\u2019]/g, "'") // Smart apostrophes
    .replace(/\n/g, '\\n'); // Newlines

  try {
    parsed = JSON.parse(fixed);
  } catch (e2) {
    return {error: "Invalid JSON", raw, attempted_fix: fixed};
  }
}

// Validate required fields
if (!parsed.title || !parsed.content) {
  return {error: "Missing required fields", parsed};
}

return parsed;
```

**Detection:**
- Workflow errors mention "Unexpected token" or "JSON parse error"
- Some executions succeed, others fail with same prompt
- Error logs show raw AI responses with malformed JSON

**Phase mapping:** Phase 2 (Content Generator MVP) - establish robust parsing before scaling.

---

### Pitfall 10: Credential Management Chaos in Multi-Site Setup

**What goes wrong:** Managing credentials for multiple WordPress sites, Google Sheets, OpenRouter, and Google Imagen becomes complex, leading to wrong credentials used for wrong site.

**Why it happens:**
- n8n stores credentials globally (not per-workflow)
- Naming ambiguity ("WordPress Site 1" vs "Site 1 WordPress")
- Copy-paste errors when duplicating workflows
- No validation that credential matches target site

**Prevention:**
- Establish naming convention: `{service}_{site_domain}` (e.g., `wordpress_siteA.com`)
- Store credential-to-site mapping in Google Sheets
- Add validation node: confirm WordPress credential matches target URL
- Use n8n's credential testing before workflow runs
- Document all credentials in `.planning/CREDENTIALS.md`
- Implement environment-based credential selection (dev vs production)

**Validation pattern:**
```javascript
// Code node: Validate credential matches site
const targetSite = $json.site_url; // From Google Sheets
const credentialName = $node["WordPress"].credential.name;

if (!credentialName.includes(targetSite.replace(/https?:\/\//, '').split('/')[0])) {
  return {
    error: "CREDENTIAL_MISMATCH",
    expected_site: targetSite,
    credential_used: credentialName,
    action: "STOP_EXECUTION"
  };
}

return {validated: true};
```

**Detection:**
- Posts published to wrong WordPress site
- Error: "Authentication failed" despite valid credentials
- Google Sheets for Site A contains Site B data

**Phase mapping:** Phase 4 (Multi-Site) - establish credential management before adding second site.

---

## Minor Pitfalls

Mistakes that cause annoyance but are easily fixable.

### Pitfall 11: Missing Error Notifications

**What goes wrong:** Workflows fail silently, and team doesn't know content wasn't published until manually checking.

**Prevention:**
- Add error trigger workflow (catches all failed executions)
- Send notifications via Slack/Email on failure
- Log all errors to dedicated Google Sheet
- Set up monitoring dashboard

**Phase mapping:** Phase 2 (Content Generator MVP).

---

### Pitfall 12: Hardcoded Values Instead of Configuration

**What goes wrong:** Site URLs, API endpoints, model names hardcoded in workflows, making multi-site deployment tedious.

**Prevention:**
- Store all configuration in Google Sheets "Config" tab
- Read config at workflow start
- Use variables throughout workflow
- Document all configurable values

**Phase mapping:** Phase 1 (Foundation).

---

### Pitfall 13: No Content Preview Before Publishing

**What goes wrong:** Low-quality content published directly to production, requiring manual cleanup.

**Prevention:**
- Add IF node: Check "auto_publish" flag from Google Sheets
- Default to draft mode, manual review optional
- Implement preview generation (save HTML to file, share link)

**Phase mapping:** Phase 2 (Content Generator MVP).

---

### Pitfall 14: Execution History Clutter

**What goes wrong:** n8n execution history fills with thousands of executions, making debugging difficult.

**Prevention:**
- Set execution retention policy (30 days)
- Archive important executions before deletion
- Use workflow tags for filtering

**Phase mapping:** Phase 5 (Monitoring).

---

### Pitfall 15: Turkish Language Prompt Engineering Gaps

**What goes wrong:** AI generates content in English-influenced Turkish, mixing formal/informal tone, or using unnatural phrasing.

**Prevention:**
- Provide Turkish writing style guide in system prompt
- Include Turkish content examples in few-shot prompts
- Specify formality level (formal blog vs conversational)
- Test prompts with multiple models (Claude often better for Turkish than GPT-4)

**Phase mapping:** Phase 3 (Quality Controls).

---

## Phase-Specific Warnings

| Phase Topic | Likely Pitfall | Mitigation |
|-------------|---------------|------------|
| Phase 1: Foundation Setup | Pitfall #1: Default parameter values | Validate all n8n nodes with `mode: 'full'` before deployment |
| Phase 1: Foundation Setup | Pitfall #12: Hardcoded values | Create Config sheet before building workflows |
| Phase 2: Topic Planner | Pitfall #6: Keyword cannibalization | Implement keyword mapping before generating first topics |
| Phase 2: Content Generator | Pitfall #3: HTML formatting corruption | Build Markdown→HTML converter in MVP, test with Turkish samples |
| Phase 2: Content Generator | Pitfall #9: JSON parsing failures | Add extraction/validation layer for all AI responses |
| Phase 2: Content Generator | Pitfall #8: Media upload failures | Separate media upload from post creation, implement retry |
| Phase 3: Quality Controls | Pitfall #5: AI hallucination | Add fact-checking pipeline before scaling content volume |
| Phase 3: Quality Controls | Pitfall #15: Turkish language quality | Develop comprehensive prompt engineering guide |
| Phase 4: Multi-Site | Pitfall #4: Google Sheets race conditions | Implement locking protocol before adding second site |
| Phase 4: Multi-Site | Pitfall #7: Memory limits | Test batch sizes, implement sub-workflow isolation |
| Phase 4: Multi-Site | Pitfall #10: Credential chaos | Establish naming conventions and validation |
| Phase 5: Monitoring | Pitfall #2: OpenRouter rate limits | Monitor token usage, implement intelligent queuing |

---

## Critical Success Factors

To avoid these pitfalls, ensure each phase includes:

1. **Validation Gates**: Never deploy without full validation (`validate_node`, `validate_workflow`)
2. **Error Handling**: Every external API call wrapped in try-catch with retry logic
3. **Data Integrity**: All Google Sheets operations use append-only pattern with execution IDs
4. **Content Quality**: Multi-layer validation (schema → facts → formatting → preview)
5. **Observability**: Comprehensive logging and error notifications from day one
6. **Configuration Management**: No hardcoded values, all config in Sheets
7. **Testing Protocol**: Test with real Turkish content, full batch sizes, concurrent executions

---

## Quick Reference: Pre-Deployment Checklist

Before deploying any workflow:

- [ ] All n8n nodes validated with `mode: 'full', profile: 'runtime'`
- [ ] No default parameter values relied upon
- [ ] Error handling on all HTTP Request nodes (retry: 3x with backoff)
- [ ] Google Sheets operations use append (not update by row)
- [ ] OpenRouter responses parsed and validated (not assumed JSON)
- [ ] WordPress HTML encoding set to UTF-8
- [ ] Markdown→HTML conversion tested with Turkish characters
- [ ] Image upload separate from post creation with retry
- [ ] Keyword conflict detection active
- [ ] Error notification configured
- [ ] Execution logged to Google Sheets with workflow ID
- [ ] Preview/draft mode available (not auto-publish only)
- [ ] Rate limiting implemented for batch operations
- [ ] Memory limits tested for maximum batch size
- [ ] Credentials validated for target site

---

## Sources

**Note:** This research is based on my training knowledge (current through January 2025) of:
- n8n workflow automation architecture and common failure patterns
- OpenRouter API specifications and multi-model integration challenges
- WordPress REST API documentation and media handling requirements
- Google Sheets API rate limits and concurrent access patterns
- AI content generation best practices and hallucination mitigation
- SEO pillar-cluster strategy implementation pitfalls
- Turkish language NLP considerations in AI models

**Confidence Level: HIGH** for technical API behaviors and n8n architecture patterns, **MEDIUM** for specific 2026 rate limit values (which may have changed since my training data). Recommend verifying current OpenRouter and Google Sheets API quotas during Phase 1 implementation.

Key guidance sourced from project's CLAUDE.md, which provides authoritative n8n-MCP best practices including the critical warning about default parameter values being the #1 source of runtime failures.
