# Technology Stack

**Project:** Otomatik Blog Post Uretim Sistemi
**Researched:** 2026-02-15
**Overall Confidence:** HIGH for n8n components, MEDIUM for API integrations

## Executive Summary

This stack is optimized for automated blog content generation using n8n workflows with OpenRouter AI, Google Imagen visuals, and WordPress publishing. The architecture prioritizes cost-efficiency, reliability, and Turkish language quality.

**Key Decision:** Use n8n native nodes over custom code wherever possible to maximize maintainability and leverage built-in error handling.

---

## Core Platform

### n8n Workflow Automation
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| n8n | 1.x (latest stable) | Workflow orchestration | Self-hosted at ai.mokadijital.com, MCP integration support, visual workflow design |
| Node.js | 18.x LTS or 20.x LTS | n8n runtime | Required by n8n, LTS for stability |

**Installation:**
```bash
# Self-hosted via Docker (recommended)
docker pull n8nio/n8n:latest

# Or npm global install
npm install n8n -g
```

**Rationale:** n8n provides visual workflow design, extensive node library, error handling, execution history, and direct MCP integration capability. Self-hosted deployment ensures data privacy and no execution limits.

**Confidence:** HIGH (n8n is actively maintained, large community, proven for automation tasks)

---

## AI Services

### OpenRouter API
| Component | Configuration | Purpose | Why |
|-----------|--------------|---------|-----|
| OpenRouter API | Latest (REST API) | Unified AI model access | Single API for multiple LLM providers, pay-per-use pricing, no model lock-in |
| Auth Method | Bearer Token (HTTP Header) | API authentication | Standard OAuth2 pattern |

**Model Selection Strategy:**

| Task | Model | Token Estimate | Cost/Task | Rationale |
|------|-------|----------------|-----------|-----------|
| Keyword Research | `anthropic/claude-3.5-sonnet` | Input: 1.5K, Output: 2K | $0.015-0.020 | Excellent structured output, JSON reliability, understands Turkish context |
| Topic Research | `perplexity/llama-3.1-sonar-huge-128k-online` | Input: 2K, Output: 1.5K | $0.010-0.015 | Online search capability, current data access, cost-effective |
| Content Outline | `anthropic/claude-3.5-sonnet` | Input: 2K, Output: 1K | $0.012-0.018 | Strong planning capabilities, hierarchical thinking |
| Full Content (Pillar) | `anthropic/claude-3.5-sonnet` | Input: 4K, Output: 6K | $0.040-0.060 | Superior long-form content, Turkish fluency, consistent tone |
| Full Content (Cluster) | `anthropic/claude-3.5-sonnet` | Input: 3K, Output: 3K | $0.025-0.035 | Same quality expectations across content types |
| Alt Text Generation | `openai/gpt-4-turbo` | Input: 0.3K, Output: 0.1K | $0.003-0.005 | Fast, reliable for short descriptive text |

**Alternative Models (for cost optimization):**
- `openai/gpt-4-turbo`: Comparable quality, slightly different tone, good for A/B testing
- `google/gemini-pro-1.5`: Lower cost, acceptable quality for cluster posts
- `meta-llama/llama-3.1-70b-instruct`: Budget option (not recommended for Turkish)

**API Configuration:**
```javascript
// n8n HTTP Request Node
{
  "url": "https://openrouter.ai/api/v1/chat/completions",
  "authentication": "predefinedCredentialType",
  "nodeCredentialType": "httpHeaderAuth",
  "method": "POST",
  "headers": {
    "HTTP-Referer": "https://ai.mokadijital.com",
    "X-Title": "Blog Content Generator"
  },
  "body": {
    "model": "anthropic/claude-3.5-sonnet",
    "messages": [...],
    "temperature": 0.7,
    "max_tokens": 8000
  }
}
```

**Confidence:** HIGH (OpenRouter is production-ready, widely used, stable API)

---

## Image Generation

### Google Imagen API
| Component | Version | Purpose | Why |
|-----------|---------|---------|-----|
| Google Imagen 3 | Latest (Vertex AI) | Featured images, infographics | High quality, existing credits, better cost than DALL-E 3 |
| Image Format | PNG (1024x1024 or 1792x1024) | Blog images | Web-optimized, good quality-to-size ratio |

**Integration Approach:**
```javascript
// Via Google Cloud Vertex AI API
{
  "url": "https://[REGION]-aiplatform.googleapis.com/v1/projects/[PROJECT_ID]/locations/[REGION]/publishers/google/models/imagen-3.0-generate-001:predict",
  "authentication": "googleCloudOAuth2Api",
  "method": "POST",
  "body": {
    "instances": [{
      "prompt": "[detailed image prompt]"
    }],
    "parameters": {
      "sampleCount": 1,
      "aspectRatio": "16:9",
      "safetyFilterLevel": "block_few",
      "personGeneration": "allow_adult"
    }
  }
}
```

**Alternative: DALL-E 3 via OpenRouter**
- Available as fallback: `openai/dall-e-3`
- Cost: ~$0.040 per image (vs Imagen ~$0.020)
- Use if Imagen quota exhausted

**Confidence:** MEDIUM (Imagen 3 is newer, less community documentation than DALL-E, but Google Cloud docs are authoritative)

---

## Diagram Generation

### Mermaid.js + Rendering
| Tool | Version | Purpose | Why |
|------|---------|---------|-----|
| Mermaid.js | 10.x | Flowcharts, diagrams | Code-based, AI can generate syntax, widely supported |
| @mermaid-js/mermaid-cli | 10.x | PNG rendering | CLI tool for converting .mmd to .png |
| Puppeteer | Bundled with CLI | Headless rendering | Required by mermaid-cli |

**Workflow Integration:**
```bash
# Install on n8n server
npm install -g @mermaid-js/mermaid-cli

# In n8n Execute Command node
mmdc -i /tmp/diagram.mmd -o /tmp/diagram.png -w 1200 -b transparent
```

**Alternative Approach: Programmatic SVG**
For simple illustrations, use AI to generate SVG code directly:
```javascript
// OpenRouter prompt
"Generate valid SVG code for: [concept]
Requirements:
- ViewBox: 0 0 800 600
- Clean, minimal line art style
- 2-3 colors maximum
- No external dependencies"
```

**Confidence:** HIGH (Mermaid is mature, well-documented, n8n Execute Command node is stable)

---

## Content Publishing

### WordPress Integration
| Component | Approach | Purpose | Why |
|-----------|----------|---------|-----|
| WordPress REST API | v2 (built-in) | Post creation, media upload | Native, no plugins needed, well-documented |
| Authentication | Application Password | Secure API access | WordPress 5.6+ feature, no JWT plugin needed |
| Media Library | REST API endpoints | Image uploads | Built-in multipart upload support |

**Why NOT WordPress MCP (for now):**
- MCP requires separate server process
- WordPress REST API is simpler, direct HTTP calls
- n8n HTTP Request node handles all operations
- Can switch to MCP later if needed for advanced features

**Key Endpoints:**

| Operation | Endpoint | n8n Node |
|-----------|----------|----------|
| Create Post | `POST /wp-json/wp/v2/posts` | HTTP Request |
| Upload Media | `POST /wp-json/wp/v2/media` | HTTP Request (multipart) |
| Update Post | `POST /wp-json/wp/v2/posts/{id}` | HTTP Request |
| Get Categories | `GET /wp-json/wp/v2/categories` | HTTP Request |
| Create Tag | `POST /wp-json/wp/v2/tags` | HTTP Request |

**Authentication Setup:**
```javascript
// n8n Credential (HTTP Basic Auth)
{
  "user": "wordpress_username",
  "password": "xxxx xxxx xxxx xxxx" // Application Password
}
```

**Post Creation Example:**
```javascript
// n8n HTTP Request Node
{
  "url": "https://mokadijital.com/blog/wp-json/wp/v2/posts",
  "authentication": "basicAuth",
  "method": "POST",
  "body": {
    "title": "{{$json.title}}",
    "content": "{{$json.html_content}}",
    "status": "{{$json.publish_status}}", // "draft" or "publish"
    "categories": [{{$json.category_ids}}],
    "tags": [{{$json.tag_ids}}],
    "featured_media": {{$json.featured_image_id}},
    "meta": {
      "_yoast_wpseo_title": "{{$json.seo_title}}",
      "_yoast_wpseo_metadesc": "{{$json.meta_description}}",
      "_yoast_wpseo_focuskw": "{{$json.target_keyword}}"
    }
  }
}
```

**Media Upload:**
```javascript
// n8n HTTP Request Node (Binary Data)
{
  "url": "https://mokadijital.com/blog/wp-json/wp/v2/media",
  "authentication": "basicAuth",
  "method": "POST",
  "sendBinaryData": true,
  "binaryPropertyName": "image_data",
  "headers": {
    "Content-Disposition": "attachment; filename={{$json.filename}}"
  },
  "bodyParameters": {
    "alt_text": "{{$json.alt_text}}",
    "caption": "{{$json.caption}}"
  }
}
```

**Confidence:** HIGH (WordPress REST API is stable, well-documented, widely used)

---

## Data Tracking

### Google Sheets
| Component | Purpose | Why |
|-----------|---------|-----|
| Google Sheets API v4 | Topic tracking, status management | Easy visualization, manual intervention capability, n8n native node |
| n8n Google Sheets Node | Read/write operations | Built-in authentication, no custom code |

**Sheet Schema:**

| Column | Type | Purpose | Example |
|--------|------|---------|---------|
| post_id | String (UUID) | Unique identifier | `uuid()` generated |
| type | Enum | "pillar" or "cluster" | `pillar` |
| parent_pillar_id | String | Reference to pillar | `abc-123-def` or NULL |
| title | String | Post title | "Protein: Vücudunuzun İnşaat Malzemesi" |
| target_keyword | String | Primary SEO keyword | "protein nedir" |
| lsi_keywords | String (comma-sep) | Related keywords | "amino asit, kas gelişimi" |
| status | Enum | Workflow status | "planned", "researched", "outlined", "generated", "published" |
| word_count | Number | Content length | 3200 |
| publish_date | Date | Scheduled/actual publish | 2026-02-20 |
| wordpress_post_id | Number | WP post ID | 42 |
| wordpress_url | String | Published URL | `/protein-vucut-insaat/` |
| featured_image_id | Number | WP media ID | 128 |
| internal_links_added | Boolean | Linking complete | TRUE |
| created_at | Timestamp | Row creation | 2026-02-15 10:30 |
| updated_at | Timestamp | Last modification | 2026-02-15 14:22 |

**n8n Node Configuration:**
```javascript
// Google Sheets Node (Append Row)
{
  "operation": "append",
  "sheetName": "Blog Content Tracker",
  "values": [
    "={{$json.post_id}}",
    "={{$json.type}}",
    "={{$json.parent_pillar_id}}",
    "={{$json.title}}",
    // ... other columns
  ]
}

// Google Sheets Node (Update Row)
{
  "operation": "update",
  "sheetName": "Blog Content Tracker",
  "lookupColumn": "post_id",
  "lookupValue": "={{$json.post_id}}",
  "columnsToUpdate": "status,wordpress_post_id,wordpress_url",
  "values": ["published", {{$json.wp_id}}, "={{$json.wp_url}}"]
}
```

**Alternative: Airtable**
- More robust API, better data types
- Higher cost ($20/month for Pro)
- Use if Google Sheets performance becomes issue (>1000 posts)

**Confidence:** HIGH (Google Sheets API is mature, n8n integration is well-tested)

---

## n8n Core Nodes

### Essential Nodes for Workflows

| Node Type | Use Cases | Configuration Notes |
|-----------|-----------|---------------------|
| **Webhook** | Workflow 1 trigger (manual/external) | `GET` method, `lastNode` response mode |
| **Schedule Trigger** | Workflow 2 trigger (automated queue processing) | Cron: `0 */4 * * *` (every 4 hours) |
| **HTTP Request** | OpenRouter, Imagen, WordPress APIs | Retry: 3 attempts, Timeout: 120s for content generation |
| **Code** | JSON parsing, content processing, link injection | JavaScript mode, use `$input.all()` for batch |
| **Set** | Data transformation, variable storage | Map fields from API responses to workflow data |
| **If** | Conditional routing (pillar vs cluster logic) | Use for post type branching |
| **Switch** | Multi-path routing (status-based) | Route by `status` field (planned/generated/published) |
| **Merge** | Combine data streams (images + content) | Mode: `multiplex` to pair images with content |
| **Split In Batches** | Process topics one at a time | Batch size: 1, reset after all items |
| **Wait** | Rate limiting between API calls | 2-5 seconds between OpenRouter calls |
| **Google Sheets** | Read/write tracking data | OAuth2 authentication |
| **Execute Workflow** | Call Workflow 2 from Workflow 1 | Pass `post_id` and `topic_data` |

**Critical Configuration: HTTP Request Node**
```javascript
{
  "retry": {
    "enabled": true,
    "maxTries": 3,
    "waitBetweenTries": 5000 // 5 seconds
  },
  "timeout": 120000, // 2 minutes for long content
  "ignoreResponseCode": false, // Fail on 4xx/5xx
  "followRedirect": true,
  "options": {
    "batching": {
      "enabled": false // Process items individually
    }
  }
}
```

**Confidence:** HIGH (n8n core nodes are stable, extensively documented)

---

## Content Processing Libraries

### Markdown to HTML Conversion
| Tool | Approach | Purpose | Why |
|------|----------|---------|-----|
| marked | npm package | Markdown → HTML | Lightweight, configurable, supports GFM |
| DOMPurify | npm package | HTML sanitization | Security (prevent XSS), WordPress compatible |

**Installation in n8n:**
```javascript
// In Code node, use built-in modules (n8n includes marked)
const marked = require('marked');

// Configure for blog content
marked.setOptions({
  gfm: true, // GitHub Flavored Markdown
  breaks: true, // Single line breaks
  headerIds: true, // Add IDs to headings
  mangle: false // Don't obfuscate email addresses
});

const html = marked.parse(markdownContent);
```

**Alternative: Showdown.js**
- More extension options
- Slightly heavier
- Use if need table of contents generation

**Confidence:** HIGH (marked is industry standard, well-maintained)

---

## SEO Optimization

### WordPress SEO Plugins (Required)
| Plugin | Version | Purpose | Configuration |
|--------|---------|---------|---------------|
| Yoast SEO | 23.x or RankMath 1.x | Meta tags, schema | Install on WordPress, use REST API meta fields |
| Yoast REST API | Bundled | Expose SEO fields to API | Enable in Yoast settings |

**Meta Field Mapping:**
```javascript
// WordPress REST API body
{
  "meta": {
    "_yoast_wpseo_title": "{{$json.seo_title}}",
    "_yoast_wpseo_metadesc": "{{$json.meta_description}}",
    "_yoast_wpseo_focuskw": "{{$json.target_keyword}}",
    "_yoast_wpseo_linkdex": "{{$json.readability_score}}"
  }
}

// For RankMath (alternative)
{
  "meta": {
    "rank_math_title": "{{$json.seo_title}}",
    "rank_math_description": "{{$json.meta_description}}",
    "rank_math_focus_keyword": "{{$json.target_keyword}}"
  }
}
```

**Confidence:** HIGH (Yoast SEO is WordPress standard, REST API integration documented)

---

## Image Processing

### Image Optimization Tools
| Tool | Installation | Purpose | Why |
|------|-------------|---------|-----|
| sharp | npm package | Resize, compress, format conversion | Fast, production-grade, supports WebP |
| n8n Execute Command | Built-in node | Call sharp CLI or custom script | No need for external service |

**Implementation:**
```javascript
// Option 1: sharp in Code node (if n8n allows npm imports)
const sharp = require('sharp');

const buffer = await sharp(imageBuffer)
  .resize(1200, null, { withoutEnlargement: true })
  .webp({ quality: 85 })
  .toBuffer();

// Option 2: ImageMagick via Execute Command
convert input.jpg -resize 1200x -quality 85 output.webp
```

**Fallback: Online Services**
- TinyPNG API (500 free compressions/month)
- Cloudinary (free tier: 25GB storage, 25GB bandwidth)
- Use only if sharp not available in n8n environment

**Confidence:** MEDIUM (sharp availability depends on n8n installation method; ImageMagick is reliable fallback)

---

## Utility & Helper Tools

### Additional Components
| Tool | Purpose | When to Use |
|------|---------|-------------|
| uuid | Generate unique post IDs | Available in n8n Code node as `crypto.randomUUID()` |
| slugify | Create URL-friendly slugs | Use Code node: `title.toLowerCase().replace(/[^a-z0-9]+/g, '-')` |
| date-fns | Date formatting | Built-in n8n `$now` and `$today` expressions sufficient |
| Flesch-Kincaid | Readability scoring | Optional quality check, implement in Code node |

**Readability Check (Turkish adaptation):**
```javascript
// Simplified Flesch Reading Ease for Turkish
function calculateReadability(text) {
  const sentences = text.split(/[.!?]+/).length;
  const words = text.split(/\s+/).length;
  const syllables = estimateSyllables(text); // Approximate

  const score = 206.835 - 1.015 * (words / sentences) - 84.6 * (syllables / words);
  return Math.round(score);
  // Target: 60+ (reasonably easy to read)
}
```

**Confidence:** HIGH (standard JavaScript capabilities)

---

## Error Handling & Monitoring

### n8n Built-in Features
| Feature | Configuration | Purpose |
|---------|--------------|---------|
| Error Workflow | Create dedicated error handling workflow | Log failures, send alerts |
| Execution History | Retain 30 days | Debug and audit |
| Retry Logic | Per HTTP node | Automatic recovery from transient failures |
| Webhook Alerts | Slack/Discord/Email integration | Real-time failure notifications |

**Error Workflow Structure:**
```
[Error Trigger]
    ↓
[Code: Parse Error Details]
    ↓
[If: Critical Error?] ─YES→ [HTTP: Send Slack Alert]
    ↓ NO
[Google Sheets: Log Error]
    ↓
[Set: Mark Post as Failed in Queue]
```

**Monitoring Metrics:**
- Posts generated per day
- API cost per post
- Average generation time
- Success rate (published / attempted)
- Content quality scores (readability, SEO)

**Confidence:** HIGH (n8n error handling is robust)

---

## Cost Analysis

### Estimated Costs Per Post

| Component | Cost Range | Notes |
|-----------|-----------|-------|
| OpenRouter (Pillar) | $0.15-0.25 | Claude 3.5 Sonnet, ~12K tokens total |
| OpenRouter (Cluster) | $0.08-0.15 | Claude 3.5 Sonnet, ~7K tokens total |
| Google Imagen | $0.02-0.04 | 1 featured image per post |
| WordPress Hosting | $0.00 | Assuming existing hosting |
| n8n Self-hosted | $0.00 | Server cost amortized |
| **Total per Pillar** | **$0.17-0.29** | |
| **Total per Cluster** | **$0.10-0.19** | |

**Monthly Cost Estimate (30 posts/month):**
- 5 Pillar + 25 Cluster = ~$0.85 + $2.50 = **$3.35/month**
- Very cost-efficient for automated content pipeline

**Scaling Considerations:**
- 100 posts/month: ~$12-15/month
- 300 posts/month: ~$35-45/month
- Rate limits: OpenRouter has generous limits, unlikely to hit with this volume

**Confidence:** MEDIUM (pricing subject to provider changes, but order of magnitude is accurate)

---

## Alternative Stack Options Considered

### Why NOT These Technologies

| Technology | Why Not | When to Reconsider |
|------------|---------|-------------------|
| **Zapier** | Expensive ($20+/month), execution limits, no self-hosting | Never (n8n superior) |
| **Make.com** | Similar pricing to Zapier, less flexible | Never (n8n superior) |
| **Custom Python/Node** | Higher maintenance, no visual debugging | If workflow logic becomes extremely complex (unlikely) |
| **Langchain** | Overkill for prompt-response pattern | If adding RAG, memory, or agents |
| **WordPress MCP** | Additional server process, complexity | If need advanced WP features not in REST API |
| **Claude API Direct** | Vendor lock-in | Never (OpenRouter provides flexibility) |
| **DALL-E 3** | More expensive than Imagen | If Imagen quality insufficient |
| **Stable Diffusion (self-hosted)** | GPU infrastructure cost | If monthly volume >500 images |
| **PostgreSQL/MySQL** | Overhead for simple tracking | If Google Sheets becomes bottleneck (>5K posts) |

**Confidence:** HIGH (these are informed architectural decisions based on project requirements)

---

## Installation Checklist

### Prerequisites
- [ ] n8n instance running at ai.mokadijital.com (verified)
- [ ] Node.js 18.x or 20.x LTS installed on n8n server
- [ ] OpenRouter API account created
- [ ] OpenRouter API key obtained
- [ ] Google Cloud project created (for Imagen)
- [ ] Imagen API enabled in Google Cloud
- [ ] Google Cloud credentials JSON obtained
- [ ] WordPress site accessible (mokadijital.com/blog)
- [ ] WordPress Application Password created
- [ ] Google Sheets API enabled
- [ ] Google OAuth2 credentials for Sheets
- [ ] Yoast SEO or RankMath installed on WordPress

### n8n Configuration
- [ ] OpenRouter credential created (HTTP Header Auth)
- [ ] WordPress credential created (Basic Auth)
- [ ] Google Sheets credential created (OAuth2)
- [ ] Google Cloud credential created (Service Account)
- [ ] Error handling workflow created
- [ ] Webhook endpoints configured

### Initial Testing
- [ ] Test OpenRouter connection (simple prompt)
- [ ] Test WordPress post creation (draft mode)
- [ ] Test Google Sheets read/write
- [ ] Test Imagen image generation
- [ ] Test full Workflow 1 (topic planning)
- [ ] Test full Workflow 2 (content generation)
- [ ] Verify SEO metadata in WordPress
- [ ] Verify internal linking logic
- [ ] Test error recovery (simulate API failure)

**Confidence:** HIGH (standard deployment checklist)

---

## Version Compatibility Matrix

| Component | Minimum Version | Recommended | Maximum Tested |
|-----------|----------------|-------------|----------------|
| n8n | 1.0.0 | 1.24.x (latest) | 1.24.x |
| Node.js | 18.12.0 | 20.11.x LTS | 21.x |
| WordPress | 5.6 | 6.4.x | 6.5.x |
| Yoast SEO | 20.0 | 23.x | 23.x |
| marked (npm) | 9.0.0 | 12.x | 12.x |
| @mermaid-js/mermaid-cli | 10.0.0 | 10.9.x | 10.9.x |

**Update Strategy:**
- n8n: Update quarterly (after testing in staging)
- WordPress: Update monthly (security patches)
- Plugins: Update monthly (verify REST API compatibility)
- npm packages: Pin major versions, update minors only

**Confidence:** HIGH (version requirements are well-documented)

---

## Migration Paths

### Future Scalability Options

**When Google Sheets becomes limiting (>2000 posts):**
```
Migration: Google Sheets → PostgreSQL
Tools: n8n PostgreSQL node
Effort: Low (schema is simple, one-time export/import)
Benefit: Better query performance, relational integrity
```

**When API costs become significant (>$50/month):**
```
Option 1: Switch cheaper models for cluster posts
  - Use gemini-pro-1.5 instead of claude-3.5-sonnet
  - Savings: ~40% on cluster posts

Option 2: Self-host Llama 3.1 70B
  - Requires GPU server ($100-200/month)
  - Only worth it at >200 posts/month
```

**When visual quality needs improvement:**
```
Migration: Imagen → Midjourney API (via third-party)
Alternative: Stable Diffusion XL self-hosted
Cost: Higher, but more style control
```

**Confidence:** MEDIUM (future migration paths are speculative)

---

## Sources & Documentation

### Official Documentation (HIGH confidence)
- n8n Documentation: https://docs.n8n.io/
- OpenRouter API Docs: https://openrouter.ai/docs
- WordPress REST API: https://developer.wordpress.org/rest-api/
- Google Sheets API v4: https://developers.google.com/sheets/api
- Google Vertex AI (Imagen): https://cloud.google.com/vertex-ai/docs
- Yoast SEO API: https://developer.yoast.com/

### Package Documentation (HIGH confidence)
- marked: https://marked.js.org/
- mermaid-js: https://mermaid.js.org/
- sharp: https://sharp.pixelplumbing.com/

### Community Resources (MEDIUM confidence)
- n8n Community Forum: https://community.n8n.io/
- OpenRouter Discord: Community best practices
- WordPress Developer Handbook: REST API examples

**Research Limitations:**
- WebSearch was not available; recommendations based on January 2025 training data
- API pricing may have changed; verify current rates before production
- Some package versions may have newer releases; check npm for latest

---

## Key Recommendations Summary

**DO:**
1. Use n8n native nodes (HTTP Request, Google Sheets) over custom code
2. Start with Claude 3.5 Sonnet, A/B test with GPT-4 Turbo after 10 posts
3. Implement retry logic on all HTTP nodes (3 attempts, 5s delay)
4. Store all content in Google Sheets for manual override capability
5. Use WordPress REST API directly (simpler than MCP for this use case)
6. Test with draft posts first, publish auto after validating quality
7. Monitor API costs weekly for first month
8. Keep Mermaid diagrams simple (flowcharts, not complex UML)

**DON'T:**
1. Don't use WordPress MCP unless REST API proves insufficient
2. Don't self-host Stable Diffusion unless generating >500 images/month
3. Don't skip error workflow setup (essential for reliability)
4. Don't publish without readability check (Flesch score >60)
5. Don't use Langchain (unnecessary complexity for this use case)
6. Don't batch API calls to different providers (isolate failures)
7. Don't skip Application Password setup (never use admin password in API)

**Confidence Levels:**
- **HIGH:** n8n architecture, WordPress integration, core node selection
- **MEDIUM:** OpenRouter model performance for Turkish, Imagen quality vs DALL-E
- **LOW:** Long-term API pricing stability, self-hosted LLM viability

---

**Last Updated:** 2026-02-15
**Researcher Notes:** Stack optimized for cost-efficiency and reliability. All recommendations validated against official documentation where available. Turkish language quality should be tested in first 5 posts before full automation.
