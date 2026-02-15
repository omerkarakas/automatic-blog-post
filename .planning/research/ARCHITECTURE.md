# Architecture Patterns: n8n Automated Blog Content Generation

**Domain:** Automated content generation and publishing
**Researched:** 2026-02-15
**Confidence:** MEDIUM (based on n8n patterns, content generation best practices, and project requirements)

## Executive Summary

An n8n-based automated blog generation system with pillar-cluster SEO strategy requires a **multi-workflow architecture** with clear separation of concerns. The recommended architecture uses 2-3 core workflows with shared state management via Google Sheets, event-driven triggers, and comprehensive error handling.

**Key architectural decisions:**
- **Workflow separation:** Topic Planning vs Content Generation (different execution frequencies and dependencies)
- **State management:** Google Sheets as central data store and coordination layer
- **API orchestration:** OpenRouter for AI, Google Imagen for images, WordPress for publishing
- **Error resilience:** Retry logic, circuit breakers, and manual review queues
- **Multi-site support:** Configuration-driven site selection with per-site publishing rules

## Recommended Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    n8n Workflow Orchestration                    │
└─────────────────────────────────────────────────────────────────┘
                                │
                ┌───────────────┴───────────────┐
                │                               │
        ┌───────▼────────┐            ┌────────▼─────────┐
        │  Workflow 1:   │            │   Workflow 2:    │
        │ Topic Planner  │            │    Content       │
        │   (Weekly)     │            │   Generator      │
        │                │            │    (Daily)       │
        └───────┬────────┘            └────────┬─────────┘
                │                               │
                │      ┌─────────────────┐     │
                └─────►│  Google Sheets  │◄────┘
                       │  (State Store)  │
                       └────────┬────────┘
                                │
                    ┌───────────┼───────────┐
                    │           │           │
            ┌───────▼──┐  ┌────▼────┐  ┌──▼──────┐
            │OpenRouter│  │ Imagen  │  │WordPress│
            │   API    │  │   API   │  │   API   │
            └──────────┘  └─────────┘  └─────────┘
```

### Component Boundaries

| Component | Responsibility | Trigger Type | Frequency |
|-----------|---------------|--------------|-----------|
| **Topic Planner Workflow** | Keyword research, content calendar generation, pillar-cluster mapping | Schedule Trigger | Weekly |
| **Content Generator Workflow** | Article research, outline, writing, image generation, publishing | Webhook/Schedule | Daily or on-demand |
| **Google Sheets State Store** | Topic queue, content status tracking, metadata, site configuration | N/A (data layer) | Read/Write as needed |
| **Error Handler Workflow** (optional) | Centralized error logging, retry orchestration, alerts | Webhook from other workflows | On error |

## Data Flow Architecture

### Flow 1: Topic Planning → Content Queue

```
Schedule Trigger (Weekly)
    ↓
Keyword Research (OpenRouter API)
    ↓
Content Calendar Generation
    ↓
Pillar-Cluster Relationship Mapping
    ↓
Google Sheets: Write to "Topics Queue" sheet
    ↓
[Topics ready for content generation]
```

**Google Sheets Schema - Topics Queue:**
```
| topic_id | pillar_id | topic | keywords | cluster_type | priority | status | created_date | assigned_date |
|----------|-----------|-------|----------|--------------|----------|--------|--------------|---------------|
```

### Flow 2: Content Generation → Publishing

```
Schedule Trigger (Daily) OR Webhook (Manual)
    ↓
Google Sheets: Read next topic from queue (status='pending')
    ↓
Research Phase (OpenRouter API - Perplexity for research)
    ↓
Outline Generation (OpenRouter API - Claude 3.5 Sonnet)
    ↓
Content Writing (OpenRouter API - GPT-4 Turbo)
    ↓
Internal Linking Analysis (Google Sheets: fetch related content)
    ↓
Image Generation (Google Imagen API - 2-3 images per article)
    ↓
Quality Checks (word count, headings, images, links)
    ↓
    ├─ FAIL → Google Sheets: Update status='needs_review' → Exit
    ↓
    └─ PASS → Continue
        ↓
WordPress Publishing (draft or publish based on site config)
    ↓
Google Sheets: Update status='published', add metadata
    ↓
Internal Link Injection (update pillar/cluster posts with bidirectional links)
```

**Google Sheets Schema - Content Tracking:**
```
| content_id | topic_id | pillar_id | title | wordpress_id | wordpress_url | site | status | word_count | images_count | published_date | backlinks_added |
|------------|----------|-----------|-------|--------------|---------------|------|--------|------------|--------------|----------------|-----------------|
```

### Flow 3: Multi-Site Configuration

**Google Sheets Schema - Sites Configuration:**
```
| site_id | site_name | site_url | wordpress_api_url | wordpress_auth | publish_mode | language | active |
|---------|-----------|----------|-------------------|----------------|--------------|----------|--------|
| 1 | MokaDigital Blog | mokadijital.com/blog | https://... | credential_ref | draft | tr | TRUE |
```

## Workflow Separation Strategy

### Why Multiple Workflows?

| Concern | Single Workflow | Multi-Workflow (Recommended) |
|---------|----------------|------------------------------|
| **Execution frequency** | Wasteful (planning runs daily unnecessarily) | Efficient (planning weekly, generation daily) |
| **Error isolation** | Topic planning error blocks content generation | Failures isolated to specific workflow |
| **Debugging** | Complex execution logs | Clear separation of concerns |
| **Scalability** | All-or-nothing scaling | Independent scaling per workflow |
| **Development** | Sequential development required | Parallel development possible |

### Recommended Workflow Structure

**Workflow 1: Topic Planner (topic-planner.json)**
- **Trigger:** Schedule (every Sunday at 00:00)
- **Nodes:** ~8-12 nodes
- **Purpose:** Strategic planning layer
- **Key nodes:**
  - Schedule Trigger
  - OpenRouter API (keyword research)
  - Set nodes (data transformation)
  - IF node (validate topic quality)
  - Google Sheets (write topics queue)
  - Sticky Notes (documentation)

**Workflow 2: Content Generator (content-generator.json)**
- **Trigger:** Schedule (daily at 09:00) + Webhook (manual trigger)
- **Nodes:** ~25-35 nodes
- **Purpose:** Tactical content production
- **Key nodes:**
  - Schedule Trigger + Webhook Trigger (merged)
  - Google Sheets (read queue)
  - Multiple OpenRouter API nodes (research, outline, write)
  - Google Imagen API (image generation)
  - WordPress API (publishing)
  - IF nodes (quality checks)
  - Error handler sub-workflow
  - Google Sheets (update status)

**Workflow 3: Error Handler (Optional - error-handler.json)**
- **Trigger:** Webhook (called by other workflows)
- **Nodes:** ~5-8 nodes
- **Purpose:** Centralized error management
- **Key nodes:**
  - Webhook Trigger
  - Google Sheets (log errors)
  - IF node (error severity routing)
  - HTTP Request (send alerts - Slack/email)
  - Set node (format error messages)

## n8n-Specific Architecture Patterns

### Pattern 1: Workflow Chaining via Execute Workflow Trigger

For complex operations, use sub-workflow pattern:

```
Main Workflow (Content Generator)
    ↓
Execute Workflow Trigger → "Image Generation Workflow"
    ↓
Returns: image URLs
```

**When to use:**
- Reusable logic (image generation used by multiple workflows)
- Complex error handling
- Long-running operations that might timeout

**Implementation:**
```json
{
  "name": "Execute Image Workflow",
  "type": "n8n-nodes-base.executeWorkflow",
  "parameters": {
    "source": "database",
    "workflowId": "image-generator-workflow-id",
    "waitForCompletion": true
  }
}
```

### Pattern 2: Error Handling with Error Trigger

Every workflow should have error handling:

```
[Any Node] → Error Trigger
    ↓
IF node (is_retryable?)
    ↓ TRUE
    Wait node (exponential backoff)
    ↓
    Retry logic (max 3 attempts)
    ↓ FALSE
    Google Sheets (log error)
    ↓
    Webhook (notify admin)
```

**Implementation:**
```json
{
  "name": "Error Catcher",
  "type": "n8n-nodes-base.errorTrigger",
  "continueOnFail": false
}
```

### Pattern 3: Rate Limiting with Split In Batches

For API calls (OpenRouter, Imagen, WordPress):

```
Split In Batches (batch size: 1)
    ↓
API Call
    ↓
Wait (2 seconds)
    ↓
Loop until all items processed
```

**Why:** Prevents rate limit errors, especially for OpenRouter API (varies by model)

### Pattern 4: State Machine via Google Sheets

Google Sheets as state coordinator:

```
status field values:
- 'pending' → ready for generation
- 'in_progress' → currently processing
- 'needs_review' → failed quality check
- 'published' → completed successfully
- 'error' → failed with error

Workflow reads WHERE status='pending'
Updates to status='in_progress' immediately
Updates to final status on completion/error
```

**Why:** Prevents duplicate processing, enables manual intervention, provides audit trail

### Pattern 5: Multi-Site Configuration Pattern

Use Google Sheets for configuration, not hardcoded values:

```
Read Sites Config (Google Sheets)
    ↓
Filter by active=TRUE
    ↓
Switch node (route by site_id)
    ↓
WordPress Publish (site-specific credentials)
```

**Benefits:** Add new sites without workflow changes, enable/disable sites dynamically

## Google Sheets Schema Design

### Sheet 1: topics_queue

Primary key: `topic_id` (UUID or auto-increment)

```
Columns:
- topic_id: TEXT (primary key)
- pillar_id: TEXT (foreign key, NULL if this IS a pillar)
- topic: TEXT (article topic)
- keywords: TEXT (JSON array of keywords)
- cluster_type: ENUM('pillar', 'cluster')
- priority: INTEGER (1-5, higher = more important)
- status: ENUM('pending', 'in_progress', 'needs_review', 'published', 'error')
- created_date: TIMESTAMP
- assigned_date: TIMESTAMP (when moved to in_progress)
- notes: TEXT (optional metadata)

Indexes:
- status (for efficient querying)
- pillar_id (for relationship lookups)
```

### Sheet 2: content_tracking

Primary key: `content_id` (UUID or auto-increment)

```
Columns:
- content_id: TEXT (primary key)
- topic_id: TEXT (foreign key to topics_queue)
- pillar_id: TEXT (denormalized for quick lookup)
- title: TEXT
- wordpress_id: INTEGER (WordPress post ID)
- wordpress_url: TEXT
- site_id: TEXT (foreign key to sites_config)
- status: ENUM('draft', 'published')
- word_count: INTEGER
- images_count: INTEGER
- internal_links_count: INTEGER
- published_date: TIMESTAMP
- backlinks_added: BOOLEAN
- last_updated: TIMESTAMP

Indexes:
- topic_id (for relationship to queue)
- pillar_id (for cluster content lookup)
- site_id (for per-site filtering)
```

### Sheet 3: internal_links

Tracks bidirectional links between content

```
Columns:
- link_id: TEXT (primary key)
- source_content_id: TEXT (content that contains the link)
- target_content_id: TEXT (content being linked to)
- link_type: ENUM('pillar_to_cluster', 'cluster_to_pillar', 'cluster_to_cluster')
- anchor_text: TEXT
- created_date: TIMESTAMP

Indexes:
- source_content_id (for content updates)
- target_content_id (for backlink counts)
```

### Sheet 4: sites_config

Configuration for multi-site support

```
Columns:
- site_id: TEXT (primary key)
- site_name: TEXT
- site_url: TEXT
- wordpress_api_url: TEXT
- wordpress_auth_credential: TEXT (reference to n8n credential)
- publish_mode: ENUM('draft', 'publish')
- language: TEXT (e.g., 'tr')
- active: BOOLEAN
- daily_post_limit: INTEGER
- created_date: TIMESTAMP

Default row for testing:
site_id: 'moka-blog'
site_name: 'MokaDigital Blog'
site_url: 'https://mokadijital.com/blog'
wordpress_api_url: 'https://mokadijital.com/wp-json/wp/v2'
wordpress_auth_credential: 'wordpress-moka-auth'
publish_mode: 'draft'
language: 'tr'
active: TRUE
daily_post_limit: 3
```

### Sheet 5: error_log

Centralized error tracking

```
Columns:
- error_id: TEXT (primary key)
- workflow_name: TEXT
- node_name: TEXT
- error_message: TEXT
- error_stack: TEXT
- topic_id: TEXT (if applicable)
- retry_count: INTEGER
- severity: ENUM('low', 'medium', 'high', 'critical')
- resolved: BOOLEAN
- created_date: TIMESTAMP

Indexes:
- workflow_name (for workflow-specific debugging)
- topic_id (for content-specific errors)
- severity (for alert filtering)
```

## Error Handling and Retry Architecture

### Error Classification

| Error Type | Severity | Retry Strategy | Example |
|------------|----------|----------------|---------|
| **Rate Limit** | Medium | Exponential backoff (2s, 4s, 8s) | OpenRouter 429 response |
| **API Timeout** | Medium | Linear retry (3 attempts, 5s delay) | Imagen API timeout |
| **Invalid Response** | High | No retry, log for review | Malformed JSON from AI |
| **Quota Exceeded** | Critical | Stop workflow, alert admin | Google Sheets API quota |
| **Configuration Error** | Critical | Stop workflow, alert admin | Missing WordPress credentials |

### Retry Logic Pattern

```
Try Block (n8n Error Trigger)
    ↓
API Call Node (continueOnFail: true)
    ↓
IF node (check for error)
    ↓ TRUE (error occurred)
    Set node (increment retry_count)
    ↓
    IF node (retry_count < 3?)
        ↓ TRUE
        Function node (calculate backoff: 2^retry_count seconds)
        ↓
        Wait node (backoff duration)
        ↓
        Loop back to API Call
        ↓ FALSE (max retries exceeded)
        Google Sheets (log error)
        ↓
        Webhook (call Error Handler workflow)
    ↓ FALSE (no error)
    Continue normal flow
```

### Circuit Breaker Pattern

For external APIs, implement circuit breaker to prevent cascading failures:

```
State: CLOSED (normal operation)
    ↓
Error occurs
    ↓
Increment error counter (Google Sheets or n8n static data)
    ↓
IF error_count > threshold (e.g., 5 errors in 10 minutes)
    ↓
State: OPEN (stop calling API)
    ↓
Wait for cooldown period (e.g., 5 minutes)
    ↓
State: HALF-OPEN (try one request)
    ↓
    ├─ SUCCESS → State: CLOSED
    └─ FAILURE → State: OPEN
```

**Implementation:** Use Google Sheets to track error counts and state, or use n8n's built-in static data feature.

## API Integration Architecture

### OpenRouter API Integration

**Configuration:**
- **Base URL:** `https://openrouter.ai/api/v1/chat/completions`
- **Authentication:** Bearer token (stored in n8n credentials)
- **Models used:**
  - Research: `perplexity/sonar-pro` (web search capability)
  - Outline: `anthropic/claude-3.5-sonnet`
  - Writing: `openai/gpt-4-turbo`

**Rate Limiting:**
- Per-model rate limits vary
- Implement 2-second delay between calls
- Monitor `x-ratelimit-remaining` header

**Error Handling:**
- 429 (Rate Limit): Retry with exponential backoff
- 500 (Server Error): Retry 3 times with 5s delay
- 400 (Bad Request): Log and skip, no retry

**Node Structure:**
```json
{
  "name": "OpenRouter - Research",
  "type": "n8n-nodes-base.httpRequest",
  "parameters": {
    "method": "POST",
    "url": "https://openrouter.ai/api/v1/chat/completions",
    "authentication": "predefinedCredentialType",
    "nodeCredentialType": "openRouterApi",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        {"name": "HTTP-Referer", "value": "https://mokadijital.com"},
        {"name": "X-Title", "value": "MokaDigital Blog Automation"}
      ]
    },
    "sendBody": true,
    "bodyParameters": {
      "parameters": [
        {"name": "model", "value": "perplexity/sonar-pro"},
        {"name": "messages", "value": "={{ $json.messages }}"},
        {"name": "temperature", "value": 0.7},
        {"name": "max_tokens", "value": 2000}
      ]
    },
    "options": {
      "timeout": 30000,
      "retry": {
        "enabled": true,
        "maxTries": 3,
        "waitBetweenTries": 2000
      }
    }
  }
}
```

### Google Imagen API Integration

**Configuration:**
- **Base URL:** `https://aiplatform.googleapis.com/v1/projects/{PROJECT_ID}/locations/{LOCATION}/publishers/google/models/imagen-3.0-generate-001:predict`
- **Authentication:** Google Service Account (OAuth2)
- **Image specs:**
  - Aspect ratio: 16:9 (blog featured images)
  - Number of images: 1 (featured) + 1-2 (in-content)
  - Language: Turkish prompts

**Rate Limiting:**
- Google Cloud quota-based
- Daily/monthly limits
- Implement queue if quota exceeded

**Prompt Engineering Pattern:**
```javascript
// Function node to generate image prompt
const articleTitle = $json.title;
const articleTopic = $json.topic;

const imagePrompt = `Professional, high-quality photograph for a blog article about "${articleTitle}".
Style: Modern, clean, business-appropriate.
Mood: Professional and informative.
Content: ${articleTopic} related imagery without text overlays.
Turkish cultural context where appropriate.`;

return {
  json: {
    ...input,
    imagePrompt: imagePrompt
  }
};
```

### WordPress API Integration

**Configuration:**
- **Base URL:** `{SITE_URL}/wp-json/wp/v2/posts`
- **Authentication:** Application password or JWT
- **Multi-site:** Configuration-driven from Google Sheets

**Publishing Flow:**
```
Read site config from Google Sheets
    ↓
Set node (prepare WordPress API request)
    ↓
HTTP Request (POST /wp/v2/posts)
    ↓
Response validation (check for wordpress_id)
    ↓
IF publish_mode == 'publish'
    ↓ TRUE
    HTTP Request (POST /wp/v2/posts/{id} with status='publish')
    ↓ FALSE
    Leave as draft
    ↓
Return wordpress_id and wordpress_url
```

**Request Body Structure:**
```json
{
  "title": "{{ $json.title }}",
  "content": "{{ $json.html_content }}",
  "status": "{{ $json.publish_mode }}",
  "categories": [{{ $json.category_ids }}],
  "tags": [{{ $json.tag_ids }}],
  "featured_media": "{{ $json.featured_image_id }}",
  "lang": "tr",
  "meta": {
    "pillar_id": "{{ $json.pillar_id }}",
    "content_id": "{{ $json.content_id }}"
  }
}
```

## Quality Control Architecture

### Pre-Publishing Validation Pipeline

```
Content generated
    ↓
Validation Node 1: Word Count
    ├─ Check: word_count >= 1500 (configurable)
    └─ FAIL → Set status='needs_review', reason='word_count_low'
    ↓
Validation Node 2: Headings Structure
    ├─ Check: Has H2 headings (at least 3)
    └─ FAIL → Set status='needs_review', reason='missing_headings'
    ↓
Validation Node 3: Images
    ├─ Check: images_count >= 2 (1 featured + 1 in-content)
    └─ FAIL → Set status='needs_review', reason='insufficient_images'
    ↓
Validation Node 4: Internal Links
    ├─ Check: internal_links_count >= 2 (pillar + 1 cluster)
    └─ FAIL → Set status='needs_review', reason='missing_internal_links'
    ↓
Validation Node 5: External Links
    ├─ Check: external_links_count >= 3
    └─ FAIL → Set status='needs_review', reason='missing_external_links'
    ↓
ALL PASS → Proceed to publishing
```

**Implementation with Function Node:**
```javascript
// Quality check function
const content = $json.html_content;
const wordCount = content.split(/\s+/).length;
const h2Count = (content.match(/<h2>/g) || []).length;
const imagesCount = $json.images.length;
const internalLinksCount = (content.match(/href="https:\/\/mokadijital\.com/g) || []).length;
const externalLinksCount = (content.match(/href="https?:\/\//g) || []).length - internalLinksCount;

const validations = {
  word_count: wordCount >= 1500,
  headings: h2Count >= 3,
  images: imagesCount >= 2,
  internal_links: internalLinksCount >= 2,
  external_links: externalLinksCount >= 3
};

const allPassed = Object.values(validations).every(v => v === true);

return {
  json: {
    ...input,
    quality_check: {
      passed: allPassed,
      validations: validations,
      metrics: {
        word_count: wordCount,
        h2_count: h2Count,
        images_count: imagesCount,
        internal_links_count: internalLinksCount,
        external_links_count: externalLinksCount
      }
    }
  }
};
```

## Internal Linking Architecture

### Bidirectional Linking Strategy

**Pillar Post:**
- Contains links TO all cluster posts in that pillar
- Receives backlinks FROM all cluster posts

**Cluster Post:**
- Contains link TO pillar post
- Contains links TO 1-2 related cluster posts in same pillar
- May receive backlinks FROM other cluster posts

### Implementation Flow

```
Content published
    ↓
Read pillar_id from content metadata
    ↓
Google Sheets: Query all content WHERE pillar_id = {pillar_id} AND status = 'published'
    ↓
IF current content is CLUSTER
    ├─ Get pillar post (WHERE cluster_type='pillar' AND pillar_id={pillar_id})
    ├─ Get 1-2 related cluster posts (random or by keyword similarity)
    ├─ Generate link HTML snippets
    └─ Insert links into current content
    ↓
IF current content is PILLAR
    ├─ Get all cluster posts in this pillar
    ├─ Generate table of contents or list of links
    └─ Insert links into current content
    ↓
WordPress API: Update post content with links
    ↓
Google Sheets: Log links in internal_links sheet
    ↓
FOR EACH linked post (backlink injection)
    ├─ Read existing post content from WordPress
    ├─ Insert backlink to current post (contextually or in dedicated section)
    ├─ WordPress API: Update post
    └─ Google Sheets: Log backlink in internal_links sheet
```

### Link Injection Pattern

**For new posts (during generation):**
```javascript
// Function node: Build internal links during content generation
const pillarPost = $json.pillar_post;
const relatedClusters = $json.related_clusters;

let linkHtml = `\n\n<h3>İlgili Makaleler</h3>\n<ul>\n`;
linkHtml += `<li><a href="${pillarPost.url}">${pillarPost.title}</a></li>\n`;

relatedClusters.forEach(cluster => {
  linkHtml += `<li><a href="${cluster.url}">${cluster.title}</a></li>\n`;
});

linkHtml += `</ul>\n`;

return {
  json: {
    ...input,
    internal_links_html: linkHtml
  }
};
```

**For existing posts (backlink injection):**
```javascript
// Function node: Inject backlink into existing post
const existingContent = $json.existing_post_content;
const newPost = $json.new_post;

// Find appropriate insertion point (before conclusion, or at end)
const conclusionPattern = /<h2>.*?(Sonuç|Özet).*?<\/h2>/i;
const insertionPoint = existingContent.search(conclusionPattern);

let updatedContent;
if (insertionPoint !== -1) {
  // Insert before conclusion
  updatedContent = existingContent.slice(0, insertionPoint) +
    `\n<p>Ayrıca okuyun: <a href="${newPost.url}">${newPost.title}</a></p>\n` +
    existingContent.slice(insertionPoint);
} else {
  // Append at end
  updatedContent = existingContent +
    `\n<p>Ayrıca okuyun: <a href="${newPost.url}">${newPost.title}</a></p>\n`;
}

return {
  json: {
    ...input,
    updated_content: updatedContent
  }
};
```

## Build Order and Dependencies

### Phase 1: Foundation (Week 1)
**Goal:** Basic infrastructure and state management

1. **Google Sheets Setup**
   - Create all sheets (topics_queue, content_tracking, sites_config, internal_links, error_log)
   - Set up initial configuration (sites_config with mokadijital.com)
   - Create test data for development

2. **n8n Credentials Configuration**
   - OpenRouter API credential
   - Google Sheets credential (OAuth2)
   - Google Imagen API credential (Service Account)
   - WordPress API credential (Application Password)

3. **Basic Workflow Shell**
   - Create Topic Planner workflow (triggers and basic structure)
   - Create Content Generator workflow (triggers and basic structure)
   - Test workflow activation and execution

**Dependencies:** None (foundation)

### Phase 2: Topic Planning Workflow (Week 2)
**Goal:** Automated topic research and content calendar

1. **Keyword Research Integration**
   - OpenRouter API node (Perplexity for keyword research)
   - Prompt engineering for Turkish SEO keywords
   - Test with sample niche

2. **Pillar-Cluster Logic**
   - Function node to identify pillar vs cluster topics
   - Relationship mapping logic
   - Priority assignment

3. **Google Sheets Integration**
   - Write to topics_queue sheet
   - Validate data structure
   - Test with 10-20 sample topics

**Dependencies:** Phase 1 (Google Sheets, OpenRouter credential)

### Phase 3: Content Generation Core (Week 3)
**Goal:** End-to-end content creation without publishing

1. **Queue Management**
   - Read from topics_queue (status='pending')
   - Update status to 'in_progress'
   - Handle concurrent execution prevention

2. **AI Content Pipeline**
   - Research node (Perplexity)
   - Outline node (Claude)
   - Writing node (GPT-4)
   - Turkish language prompts
   - Test with 5 sample topics

3. **Content Validation**
   - Quality check function node
   - Validation pipeline (word count, headings)
   - Test pass/fail scenarios

**Dependencies:** Phase 2 (topics in queue)

### Phase 4: Image Generation (Week 4)
**Goal:** Automated image creation and WordPress media upload

1. **Google Imagen Integration**
   - API authentication
   - Prompt generation from article content
   - Image download and storage

2. **WordPress Media Upload**
   - Upload images to WordPress media library
   - Set featured image
   - Insert images into content
   - Test with sample articles

**Dependencies:** Phase 3 (content generation)

### Phase 5: Internal Linking System (Week 5)
**Goal:** Bidirectional pillar-cluster linking

1. **Link Analysis**
   - Query related content from Google Sheets
   - Pillar-cluster relationship detection
   - Link insertion logic

2. **Backlink Injection**
   - Read existing posts from WordPress
   - Update with backlinks
   - Track in internal_links sheet

3. **Link Validation**
   - Test pillar → cluster links
   - Test cluster → pillar links
   - Verify bidirectional relationships

**Dependencies:** Phase 3 (published content exists)

### Phase 6: WordPress Publishing (Week 6)
**Goal:** Automated publishing with quality gates

1. **Publishing Logic**
   - WordPress API integration
   - Draft vs publish mode (from sites_config)
   - Category and tag assignment

2. **Status Tracking**
   - Update content_tracking sheet
   - Store WordPress ID and URL
   - Mark as published

3. **End-to-End Test**
   - Run full pipeline: topic → content → images → links → publish
   - Verify on mokadijital.com/blog
   - Test draft mode

**Dependencies:** Phase 4 (images), Phase 5 (internal links)

### Phase 7: Error Handling (Week 7)
**Goal:** Resilient workflows with retry logic

1. **Error Detection**
   - Error Trigger nodes in all workflows
   - Error classification logic
   - Severity assessment

2. **Retry Logic**
   - Exponential backoff for rate limits
   - Retry counter tracking
   - Max retry limits

3. **Error Logging**
   - Write to error_log sheet
   - Admin notifications (optional Error Handler workflow)
   - Manual review queue

**Dependencies:** All previous phases (applies to entire system)

### Phase 8: Multi-Site Support (Week 8)
**Goal:** Expand to multiple WordPress sites

1. **Configuration-Driven Publishing**
   - Read sites_config sheet
   - Switch node for site routing
   - Site-specific credentials

2. **Per-Site Limits**
   - Daily post limit enforcement
   - Site-specific publish modes
   - Site activation toggle

3. **Testing**
   - Add second test site
   - Verify isolated publishing
   - Test site switching logic

**Dependencies:** Phase 6 (publishing working for single site)

## Anti-Patterns to Avoid

### Anti-Pattern 1: Hardcoded Configuration
**What:** Storing site URLs, credentials, or settings directly in workflow nodes
**Why bad:** Requires workflow edits for every configuration change, no multi-site flexibility
**Instead:** Use Google Sheets for all configuration, n8n credentials for auth

### Anti-Pattern 2: Monolithic Workflow
**What:** Single workflow handling topic planning, content generation, and publishing
**Why bad:** Hard to debug, inefficient execution frequency, cascading failures
**Instead:** Separate workflows by concern, use Execute Workflow for composition

### Anti-Pattern 3: Synchronous External API Calls
**What:** Waiting for each API call to complete before proceeding
**Why bad:** Slow execution, timeouts, poor user experience
**Instead:** Use parallel execution where possible, async patterns, batch processing

### Anti-Pattern 4: No Error Handling
**What:** Assuming APIs always work, no retry logic
**Why bad:** Silent failures, lost content, manual recovery required
**Instead:** Error Trigger nodes, retry logic, error logging, alerts

### Anti-Pattern 5: Manual State Management
**What:** Using workflow execution data or memory for state tracking
**Why bad:** Lost on workflow restart, no persistence, no audit trail
**Instead:** Google Sheets as state store, explicit status fields

### Anti-Pattern 6: Direct Content Storage in Sheets
**What:** Storing full article HTML in Google Sheets cells
**Why bad:** Cell size limits (50,000 chars), slow reads, expensive API calls
**Instead:** Store metadata in Sheets, content in WordPress, use IDs for reference

### Anti-Pattern 7: Uncontrolled Parallel Execution
**What:** Running multiple instances of content generator without coordination
**Why bad:** Duplicate content, race conditions, quota exhaustion
**Instead:** Status-based locking (in_progress), execution limits, queue management

### Anti-Pattern 8: Missing Quality Gates
**What:** Publishing content without validation
**Why bad:** Low-quality content, SEO penalties, manual cleanup required
**Instead:** Multi-stage validation, needs_review status, admin approval for edge cases

## Scalability Considerations

| Concern | At 10 posts/month | At 100 posts/month | At 1000 posts/month |
|---------|-------------------|---------------------|----------------------|
| **Topic Planning** | Manual keyword input acceptable | Semi-automated research needed | Fully automated topic discovery required |
| **Google Sheets** | Single sheet sufficient | May need partitioning by date | Consider database (PostgreSQL, Airtable) |
| **API Rate Limits** | No special handling | Monitor quotas, implement delays | Circuit breakers, quota pools, fallback providers |
| **WordPress Performance** | Direct API calls work | Batch operations helpful | Queue-based publishing, CDN required |
| **Workflow Execution** | Schedule-based works | Add webhook triggers | Consider n8n clustering, workflow optimization |
| **Error Handling** | Manual review okay | Automated retry critical | Self-healing workflows, predictive error prevention |
| **Internal Linking** | Real-time injection works | May need async processing | Link graph database, batch link updates |
| **Image Generation** | Generate on-demand | Pre-generate and cache | Image CDN, multiple providers, fallback to stock photos |

## Advanced Patterns (Post-MVP)

### Pattern 1: Content Versioning
Store multiple versions of content in Google Sheets or separate version control system, enable rollback.

### Pattern 2: A/B Testing Headlines
Generate multiple headlines, publish variations to different sites, track performance.

### Pattern 3: Content Calendar Optimization
ML-based topic selection using historical performance data, seasonal trends.

### Pattern 4: Automated SEO Audits
Post-publishing workflow to validate SEO elements, readability scores, keyword density.

### Pattern 5: Social Media Distribution
Extend Content Generator to auto-post to Twitter, LinkedIn with content excerpts.

### Pattern 6: Performance Monitoring Dashboard
Real-time dashboard (Google Data Studio or custom) showing content pipeline status.

## Technology Stack Integration

| Technology | Purpose | Configuration Location | Credentials |
|------------|---------|------------------------|-------------|
| **n8n** | Workflow orchestration | ai.mokadijital.com | n8n instance admin |
| **OpenRouter API** | AI content generation | n8n credentials | OpenRouter API key |
| **Google Imagen API** | Image generation | n8n credentials | Google Cloud Service Account |
| **Google Sheets API** | State management | n8n credentials | Google OAuth2 |
| **WordPress API** | Content publishing | Google Sheets (sites_config) | WordPress Application Password |

## Monitoring and Observability

### Key Metrics to Track

**In Google Sheets (automated):**
- Topics in queue (by status)
- Content generation rate (posts per day)
- Success rate (published vs needs_review)
- Error rate (by error type)
- Average time per content piece
- Internal links created (per post, total)

**In n8n (built-in):**
- Workflow execution count
- Workflow execution duration
- Node failure rate
- Execution queue depth

**In WordPress (manual/plugin):**
- Post views
- SEO rankings (requires external tool)
- Backlink counts

### Alerting Strategy

**Critical (immediate alert):**
- Workflow execution failure > 3 times in 1 hour
- API quota exceeded
- WordPress publishing error

**Medium (daily digest):**
- Content in needs_review status
- Quality check failures
- API rate limit warnings

**Low (weekly report):**
- Content generation statistics
- Internal linking metrics
- Topic queue health

## Confidence Assessment

**Overall confidence: MEDIUM**

| Aspect | Confidence | Reasoning |
|--------|-----------|-----------|
| Workflow separation strategy | HIGH | Based on n8n best practices, clear separation of concerns |
| Google Sheets schema | HIGH | Standard data modeling, proven patterns for state management |
| Error handling patterns | MEDIUM | General best practices, but n8n-specific retry mechanisms need validation |
| Internal linking logic | MEDIUM | Custom implementation, needs testing with real content graph |
| API integration details | HIGH | Standard REST API patterns, well-documented endpoints |
| Build order | HIGH | Logical dependency chain, incremental delivery |
| Multi-site architecture | MEDIUM | Configuration-driven approach sound, but needs testing with multiple sites |

**Gaps and assumptions:**
- Assumed n8n version supports all referenced node types (validate with actual instance)
- Google Sheets API rate limits may require adjustment at scale (100+ posts/month)
- Internal linking algorithm (keyword similarity) not fully specified
- WordPress REST API v2 capabilities assumed (custom fields, media upload)

## Sources

**Based on:**
- n8n official documentation and MCP tool descriptions (provided in CLAUDE.md)
- Standard workflow automation patterns for content generation systems
- RESTful API integration best practices
- Google Sheets as state store pattern (common in no-code automation)
- WordPress REST API v2 documentation (standard capabilities)
- Pillar-cluster SEO content strategy (industry standard practice)

**Confidence notes:**
- n8n-specific patterns: HIGH (based on provided MCP documentation)
- Content generation workflow design: MEDIUM (general best practices, not n8n-specific sources)
- Internal linking algorithm: LOW (custom implementation, needs validation)
- API integration specifics: MEDIUM (standard REST patterns, but provider-specific details need verification)

---

**Next steps for validation:**
1. Verify all referenced n8n nodes exist in the ai.mokadijital.com instance
2. Confirm Google Sheets API quota (reads/writes per day)
3. Test OpenRouter API rate limits for selected models
4. Validate WordPress REST API capabilities (custom fields, media uploads)
5. Prototype internal linking logic with sample content graph
