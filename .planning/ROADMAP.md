# Roadmap: Otomatik Blog Post Uretim Sistemi

## Overview

This roadmap delivers an automated Turkish-language blog content generation system powered by n8n workflows. Starting with foundation infrastructure and data schemas, we build topic planning and content generation workflows using OpenRouter AI, integrate Google Imagen for visual content, implement WordPress publishing via MCP Server, and add internal linking to complete the pillar-cluster SEO architecture. The system scales to multi-site support with comprehensive error handling and quality validation throughout.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Foundation & Infrastructure** - Google Sheets schema, n8n credentials, workflow shells
- [ ] **Phase 2: Topic Planning Workflow** - Pillar-cluster keyword research and queue management
- [ ] **Phase 3: Content Generation Core** - AI content pipeline with quality validation
- [ ] **Phase 4: Image Generation & Media Upload** - Google Imagen integration and WordPress media
- [ ] **Phase 5: WordPress Publishing Integration** - Draft/publish mode with SEO metadata
- [ ] **Phase 6: Internal Linking System** - Bidirectional pillar-cluster link injection
- [ ] **Phase 7: Error Handling & Resilience** - Retry logic, error logging, notifications
- [ ] **Phase 8: Multi-Site Support & Scaling** - Configuration-driven multi-site publishing

## Phase Details

### Phase 1: Foundation & Infrastructure
**Goal**: Establish data schemas, credentials, and validation protocols that all workflows depend on
**Depends on**: Nothing (first phase)
**Requirements**: WF-01, DATA-01, DATA-04, PUB-06
**Success Criteria** (what must be TRUE):
  1. Google Sheets workbook exists with 5 configured sheets (topics_queue, content_tracking, internal_links, sites_config, error_log)
  2. n8n credentials configured and validated for OpenRouter API, Google Sheets API, Google Imagen API, and WordPress MCP Server
  3. Workflow shells for Topic Planner and Content Generator exist in n8n with explicit parameter configuration
  4. Validation protocol established (no default parameters, full node validation before deployment)
**Plans**: TBD

Plans:
- [ ] 01-01: [TBD during phase planning]

### Phase 2: Topic Planning Workflow
**Goal**: Generate pillar-cluster topic hierarchies with SEO keywords and populate content queue
**Depends on**: Phase 1
**Requirements**: TOPIC-01, TOPIC-02, TOPIC-03, TOPIC-04, TOPIC-05, TOPIC-06, TOPIC-07, WF-02, WF-03, WF-04
**Success Criteria** (what must be TRUE):
  1. User can trigger topic planning workflow via webhook or manual trigger with niche and target audience inputs
  2. Workflow generates 5-10 pillar topics with 4-8 cluster topics each using OpenRouter API
  3. Each topic in Google Sheets includes target keyword, search intent (informational/transactional/navigational), LSI keywords, and publish date
  4. Topics organized in pillar-cluster hierarchy with parent relationship tracking
  5. Keyword cannibalization prevented (no duplicate primary keywords across topics)
**Plans**: TBD

Plans:
- [ ] 02-01: [TBD during phase planning]

### Phase 3: Content Generation Core
**Goal**: AI-powered research, outline, and writing pipeline with Turkish language quality validation
**Depends on**: Phase 2
**Requirements**: CONT-01, CONT-02, CONT-03, CONT-04, CONT-05, CONT-06, CONT-07, CONT-08, QA-01, QA-02
**Success Criteria** (what must be TRUE):
  1. Workflow reads pending topics from Google Sheets queue and updates status to in_progress
  2. AI performs research, generates outline, and writes complete article using OpenRouter API (Perplexity for research, Claude/GPT-4 for writing)
  3. Pillar posts contain 2500-3500 words, cluster posts contain 1200-1800 words
  4. Each article includes SEO-optimized meta title (50-60 chars), meta description (150-160 chars), and URL slug
  5. Content passes quality checks (minimum word count, minimum 3 H2 headings) before proceeding
  6. Turkish characters (i, s, g, u, o, c with diacritics) render correctly in HTML output
**Plans**: TBD

Plans:
- [ ] 03-01: [TBD during phase planning]

### Phase 4: Image Generation & Media Upload
**Goal**: Automated featured image creation and WordPress media library integration
**Depends on**: Phase 3
**Requirements**: IMG-01, IMG-02, IMG-03, QA-03
**Success Criteria** (what must be TRUE):
  1. Featured image generated for each article using Google Imagen API with Turkish prompt
  2. Images uploaded to WordPress media library via MCP Server
  3. Each image includes SEO-friendly alt text automatically generated from article context
  4. Quality check validates image existence before publishing
**Plans**: TBD

Plans:
- [ ] 04-01: [TBD during phase planning]

### Phase 5: WordPress Publishing Integration
**Goal**: Publish content to WordPress with configurable draft/publish modes and SEO metadata
**Depends on**: Phase 4
**Requirements**: PUB-01, PUB-02, PUB-03, PUB-04, PUB-05, DATA-02, QA-05
**Success Criteria** (what must be TRUE):
  1. Content published to WordPress via MCP Server as post with featured image
  2. Post status (draft or publish) controlled by sites_config sheet configuration
  3. Categories and tags automatically assigned based on topic metadata
  4. SEO metadata (title, description, focus keyword) saved with post
  5. WordPress post ID and URL recorded in content_tracking Google Sheet
  6. End-to-end test completes: topic input to published post visible at mokadijital.com/blog
**Plans**: TBD

Plans:
- [ ] 05-01: [TBD during phase planning]

### Phase 6: Internal Linking System
**Goal**: Bidirectional pillar-cluster linking with contextual anchor text
**Depends on**: Phase 5
**Requirements**: LINK-01, LINK-02, LINK-03, LINK-04, QA-04
**Success Criteria** (what must be TRUE):
  1. Cluster posts automatically link to parent pillar post with contextual anchor text
  2. Pillar posts automatically link to all related cluster posts
  3. When new cluster publishes, parent pillar post updates to include new cluster link (bidirectional linking)
  4. Related sibling cluster posts link to each other where contextually relevant
  5. Link graph tracked in internal_links Google Sheet for validation
**Plans**: TBD

Plans:
- [ ] 06-01: [TBD during phase planning]

### Phase 7: Error Handling & Resilience
**Goal**: Comprehensive error recovery with retry logic, logging, and notifications
**Depends on**: Phase 6
**Requirements**: WF-03, WF-04, DATA-03
**Success Criteria** (what must be TRUE):
  1. API failures trigger exponential backoff retry (2s, 4s, 8s delays, max 3 attempts)
  2. Rate limit errors (429 responses) handled with cooldown before retry
  3. All errors logged to error_log Google Sheet with classification (rate limit, timeout, invalid response)
  4. Failed content operations save to draft status with error details for manual review
  5. Admin receives notification (optional Slack/email) for critical failures
**Plans**: TBD

Plans:
- [ ] 07-01: [TBD during phase planning]

### Phase 8: Multi-Site Support & Scaling
**Goal**: Configuration-driven publishing to multiple WordPress sites with centralized management
**Depends on**: Phase 7
**Requirements**: PUB-06
**Success Criteria** (what must be TRUE):
  1. Workflow reads site configuration from sites_config sheet (URL, credentials, publish mode)
  2. Content routes to correct WordPress site using Switch node logic
  3. Each site maintains independent credentials and configuration
  4. Daily post limits enforced per site to prevent spam
  5. Site activation toggles enable/disable publishing per site without workflow changes
  6. Second test site validates multi-site isolation (content publishes correctly to both sites)
**Plans**: TBD

Plans:
- [ ] 08-01: [TBD during phase planning]

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation & Infrastructure | 0/TBD | Not started | - |
| 2. Topic Planning Workflow | 0/TBD | Not started | - |
| 3. Content Generation Core | 0/TBD | Not started | - |
| 4. Image Generation & Media Upload | 0/TBD | Not started | - |
| 5. WordPress Publishing Integration | 0/TBD | Not started | - |
| 6. Internal Linking System | 0/TBD | Not started | - |
| 7. Error Handling & Resilience | 0/TBD | Not started | - |
| 8. Multi-Site Support & Scaling | 0/TBD | Not started | - |

---
*Roadmap created: 2026-02-15*
*Last updated: 2026-02-15 (initial creation)*
