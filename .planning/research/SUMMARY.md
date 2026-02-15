# Project Research Summary

**Project:** Automated Blog Post Generation System (n8n-based)
**Domain:** Content automation, n8n workflow orchestration, pillar-cluster SEO
**Researched:** 2026-02-15
**Confidence:** HIGH for n8n/WordPress integration, MEDIUM for AI model selection and Turkish language optimization

## Executive Summary

This is an n8n-powered automated blog content generation system using pillar-cluster SEO strategy for Turkish-language blogs. The recommended architecture separates topic planning from content generation using two primary workflows orchestrated via Google Sheets state management. The system leverages OpenRouter API for multi-model AI access, Google Imagen for visual generation, and WordPress REST API for publishing.

The core insight from research is that **workflow separation is critical** — topic planning runs weekly while content generation runs daily, preventing wasteful execution and isolating failures. The stack optimizes for cost-efficiency (under $5/month for 30 posts) while maintaining quality through multi-stage validation gates. Google Sheets serves as the central state store, enabling manual intervention, multi-site coordination, and audit trails without database complexity.

The primary risks are: (1) n8n default parameter values causing silent runtime failures, (2) OpenRouter rate limit exhaustion during batch operations, and (3) Turkish HTML encoding corruption in WordPress. All three are mitigable through explicit parameter configuration, rate limiting with exponential backoff, and dedicated Markdown-to-HTML conversion with UTF-8 encoding. Starting with draft-mode publishing and phased feature rollout (core pipeline → automation → quality controls → multi-site) minimizes production risks.

## Key Findings

### Recommended Stack

The stack uses n8n native nodes wherever possible to maximize reliability and minimize custom code maintenance. OpenRouter provides model flexibility without vendor lock-in, supporting Claude 3.5 Sonnet for content generation, Perplexity for research, and GPT-4 Turbo for comparison testing. Google Imagen offers cost-effective image generation with existing credits, while WordPress REST API enables direct publishing without plugin dependencies.

**Core technologies:**
- **n8n (1.x latest)**: Workflow orchestration — self-hosted at ai.mokadijital.com with MCP integration, visual debugging, and built-in error handling
- **OpenRouter API**: Unified AI model access — pay-per-use pricing ($0.10-0.29/post), no model lock-in, supports Claude/GPT-4/Perplexity
- **Google Imagen 3**: Image generation — lower cost than DALL-E ($0.02-0.04/image), existing credits available, high quality output
- **WordPress REST API v2**: Content publishing — native authentication, no plugins required, well-documented endpoints for posts/media/metadata
- **Google Sheets API v4**: State management — simple tracking, manual override capability, multi-site configuration store, execution coordination

**Critical tools:**
- **marked.js**: Markdown to HTML conversion with Turkish character support
- **Mermaid.js + CLI**: Code-based diagram generation for infographics
- **Yoast SEO / RankMath**: WordPress SEO metadata via REST API fields

**Cost estimate:** $3-5/month for 30 posts (5 pillar + 25 cluster), scales to $35-45/month at 300 posts.

### Expected Features

Research reveals a clear distinction between table stakes features (users expect) and competitive differentiators (set product apart). The MVP must nail the core automation pipeline before adding intelligence.

**Must have (table stakes):**
- Keyword research and pillar-cluster topic generation — foundation of SEO strategy
- AI-powered content writing with configurable length (1500-3000+ words) — core automation value
- SEO optimization (meta title/description, URL slugs, keyword targeting) — non-negotiable for blog content
- Featured image generation with alt text — visual content expected
- WordPress integration with draft/publish control — publishing is the end goal
- Internal linking structure (cluster → pillar) — pillar-cluster model requirement
- Google Sheets tracking (status, publication log, error log) — visibility and audit trail
- Manual and automated triggers — flexibility for testing and production
- Multi-site support — critical for agency/enterprise use

**Should have (competitive differentiators):**
- Turkish language optimization (readability scoring, prompt engineering) — market-specific advantage
- SEO validation/scoring — quality assurance before publish
- Scheduled publishing with content calendar — set-and-forget operation
- Contextual internal linking (not just hardcoded patterns) — SEO value maximization
- Multi-pass content refinement — quality improvement over single-shot generation

**Defer (v2+):**
- Fact-checking integration — very high complexity, can validate manually initially
- Competitor content analysis — high complexity, manual research works for v1
- Advanced topic research (trending detection, gap analysis) — need content portfolio first
- Analytics integration (GA4, Search Console tracking) — no content to track yet
- A/B testing headlines/formats — requires traffic volume
- LSI keywords, schema markup, SERP optimization — marginal SEO value, premature optimization
- Multi-language support beyond Turkish — market expansion feature

### Architecture Approach

The architecture uses multi-workflow separation with Google Sheets as central state coordinator. Workflow 1 (Topic Planner) runs weekly to generate pillar-cluster topic hierarchies and populate the content queue. Workflow 2 (Content Generator) runs daily (or on-demand via webhook), reading from the queue to research, outline, write, generate images, and publish to WordPress. All workflows update shared state in Google Sheets, enabling coordination, manual intervention, and audit trails.

**Major components:**
1. **Topic Planner Workflow (weekly)** — keyword research via OpenRouter, pillar-cluster mapping logic, Google Sheets queue population, topic priority assignment
2. **Content Generator Workflow (daily/on-demand)** — queue reading, AI content pipeline (research → outline → write), image generation, quality validation, WordPress publishing, internal link injection
3. **Google Sheets State Store** — topics_queue (pending/in_progress/published status), content_tracking (metadata and WordPress IDs), internal_links (bidirectional link tracking), sites_config (multi-site configuration), error_log (centralized error tracking)
4. **WordPress Publishing Layer** — REST API integration for posts/media, draft vs publish mode per site, SEO metadata injection via Yoast/RankMath fields
5. **Error Handler Workflow (optional)** — centralized error logging, retry orchestration with exponential backoff, admin alerts via Slack/email

**Data flow:** Topic Planning writes to queue → Content Generator reads queue item → AI generates content → Images generated → Quality checks → WordPress publishing → Status updated in Sheets → Internal links injected into existing posts.

**Key architectural patterns:**
- **Append-only operations** for Google Sheets to prevent race conditions
- **Split in Batches** (3-5 items) with rate limiting (2s delays) for API calls
- **Multi-stage validation** (minimal → full → workflow) before deployment
- **Configuration-driven publishing** from sites_config sheet for multi-site support
- **Circuit breaker pattern** for external API failures (track error counts, implement cooldown)

### Critical Pitfalls

1. **n8n Default Parameter Values Cause Runtime Failures** — The #1 source of failures is relying on default node parameters. Nodes that appear valid in the builder fail at runtime with cryptic errors. **Prevention:** Explicitly set ALL parameters, use `validate_node({mode: 'full', profile: 'runtime'})` before deployment, never trust defaults even for "optional" fields. Test with real data, not placeholders. Address in Phase 1 (Foundation).

2. **OpenRouter Rate Limits and Model Quota Exhaustion** — Batch content generation hits per-model rate limits (429 errors), causing partial failures and inconsistent output when auto-switching to fallback models. **Prevention:** Implement exponential backoff retry logic (2s, 4s, 8s), add rate limiting between API calls, use Split in Batches with small sizes (3-5), monitor token usage, validate JSON schema after each response to catch model switches. Address in Phase 2-3 (Content Generator + Quality Controls).

3. **WordPress HTML Formatting Corruption** — AI-generated Markdown with Turkish characters (ı, ş, ğ, ü, ö, ç) breaks when converted to WordPress HTML, resulting in garbled published content. **Prevention:** Add dedicated Markdown→HTML conversion node using marked.js, set UTF-8 encoding explicitly (`Content-Type: application/json; charset=utf-8`), validate Turkish characters with regex, test with Turkish samples before production. Address in Phase 2 (Content Generator MVP).

4. **Google Sheets Concurrent Access and Race Conditions** — Multiple workflows updating same sheet simultaneously cause overwrites, duplicate rows, and data loss. **Prevention:** Use append-only operations (never update by row number), implement row-level locking with dedicated Lock column, add workflow execution IDs to each row, use atomic read-modify-write operations. Address in Phase 1 (Foundation) and Phase 4 (Multi-Site).

5. **SEO Keyword Cannibalization Between Pillar and Cluster Posts** — AI naturally generates overlapping keywords across posts, causing Google to see competing pages for same query, resulting in ALL pages ranking lower. **Prevention:** Maintain keyword mapping in Google Sheets, validate each title/H1 against existing content, use keyword exclusion lists in prompts, implement automated conflict detection before publish. Address in Phase 1 (Topic Planner) and Phase 3 (Quality Controls).

## Implications for Roadmap

Based on research, the build order must respect technical dependencies (Google Sheets → content generation → publishing → linking) and risk mitigation (start simple, validate quality, then scale). The architecture research clearly identifies 8 phases with specific prerequisites.

### Phase 1: Foundation & Infrastructure
**Rationale:** All workflows depend on Google Sheets state management and n8n credentials. Must establish data schemas, validation protocols, and configuration management before any automation.

**Delivers:**
- Google Sheets setup (all 5 sheets: topics_queue, content_tracking, internal_links, sites_config, error_log)
- n8n credentials configured (OpenRouter, Google Sheets, Google Imagen, WordPress)
- Basic workflow shells (Topic Planner and Content Generator structure)
- Validation protocol established (no default parameters, full node validation)

**Addresses:**
- Foundation for all table stakes features
- Configuration-driven architecture (no hardcoded values)

**Avoids:**
- Pitfall #1: Default parameter values (establish validation protocol)
- Pitfall #4: Race conditions (design append-only schema)
- Pitfall #12: Hardcoded configuration (build Config sheet first)

**Research needed:** None — standard n8n setup and Google Sheets schema design.

---

### Phase 2: Topic Planning Workflow
**Rationale:** Content generation requires topics in queue. Topic planning has fewer dependencies (just OpenRouter API) and lower risk than full content pipeline.

**Delivers:**
- Keyword research via OpenRouter (Perplexity API with Turkish prompts)
- Pillar-cluster topic identification and relationship mapping
- Priority assignment logic
- Google Sheets queue population
- Test with 10-20 sample topics for target niche

**Addresses:**
- Table stakes: Keyword research, pillar topic identification, cluster generation
- Differentiator: Turkish language optimization in prompts

**Avoids:**
- Pitfall #6: Keyword cannibalization (implement keyword segregation from start)
- Pitfall #2: Rate limits (basic retry logic for API calls)

**Research needed:** Minimal — OpenRouter API prompting patterns for Turkish SEO keywords.

---

### Phase 3: Content Generation Core
**Rationale:** Core automation pipeline without publishing enables testing of AI quality, validation gates, and error handling in safe environment (no public content risk).

**Delivers:**
- Queue management (read pending topics, update status to in_progress)
- AI content pipeline (Perplexity research → Claude outline → GPT-4 writing)
- Markdown to HTML conversion with Turkish character support
- Quality validation (word count, heading structure, basic checks)
- Test with 5 sample topics, review quality before proceeding

**Addresses:**
- Table stakes: AI-powered content writing, structured format, meta title/description
- Differentiator: Multi-pass architecture foundation (research → outline → write)

**Avoids:**
- Pitfall #3: HTML corruption (build robust Markdown→HTML converter)
- Pitfall #9: JSON parsing failures (add extraction/validation layer)
- Pitfall #5: AI hallucination (test Turkish language quality in controlled environment)
- Pitfall #15: Turkish prompt engineering (develop comprehensive prompts)

**Research needed:** Medium — Prompt engineering for Turkish content quality, testing multiple models for output comparison.

---

### Phase 4: Image Generation & Media Upload
**Rationale:** Images depend on content being generated (need article topics for prompts). Media upload is separate from publishing, allowing isolation of image pipeline failures.

**Delivers:**
- Google Imagen API integration with Turkish prompt generation
- Image download and storage
- WordPress media library upload (separate from post creation)
- Featured image assignment
- Alt text generation
- Test with sample articles

**Addresses:**
- Table stakes: Featured image generation, image alt text

**Avoids:**
- Pitfall #8: Media upload failures (separate from post creation, implement retry)

**Research needed:** Low — Google Imagen API is well-documented, WordPress media upload is standard.

---

### Phase 5: Internal Linking System
**Rationale:** Internal linking requires existing published content to link between. This phase implements the pillar-cluster SEO architecture.

**Delivers:**
- Link analysis (query related content from Google Sheets)
- Pillar-cluster relationship detection
- Link insertion logic (cluster → pillar, pillar → clusters)
- Backlink injection into existing posts
- Bidirectional link tracking in internal_links sheet
- Validation of link graph

**Addresses:**
- Table stakes: Internal linking structure (pillar-cluster model requirement)
- Differentiator: Contextual anchor text (foundation for future enhancement)

**Avoids:**
- Keyword conflicts during linking (cross-reference with cannibalization detection)

**Research needed:** Medium — Custom implementation, needs testing with real content graph to validate algorithm.

---

### Phase 6: WordPress Publishing Integration
**Rationale:** Publishing is the end goal, but comes after all content generation and quality validation is proven. Start with draft mode for safety.

**Delivers:**
- WordPress REST API integration (posts, categories, tags)
- Draft vs publish mode configuration (from sites_config sheet)
- SEO metadata injection (Yoast/RankMath fields)
- Status tracking in content_tracking sheet
- End-to-end test: topic → content → images → links → publish
- Verify on mokadijital.com/blog

**Addresses:**
- Table stakes: WordPress integration, draft/publish control, SEO optimization
- Differentiator: Configurable publishing per site

**Avoids:**
- Publishing low-quality content (draft mode first, manual review queue)

**Research needed:** Low — WordPress REST API is well-documented and stable.

---

### Phase 7: Error Handling & Resilience
**Rationale:** Applies to entire system, but only testable after full pipeline exists. Critical for production reliability.

**Delivers:**
- Error Trigger nodes in all workflows
- Error classification logic (rate limit vs timeout vs invalid response)
- Retry logic with exponential backoff
- Error logging to error_log sheet
- Admin notifications (Slack/email optional)
- Manual review queue for failed content

**Addresses:**
- Table stakes: Error logging (visibility requirement)
- Differentiator: Self-healing workflows

**Avoids:**
- Pitfall #2: Rate limit failures (comprehensive retry with backoff)
- Pitfall #11: Silent failures (error notifications)

**Research needed:** None — n8n error handling patterns are standard.

---

### Phase 8: Multi-Site Support & Scaling
**Rationale:** Expands to multiple WordPress sites, leveraging configuration-driven architecture established in Phase 1.

**Delivers:**
- Configuration-driven publishing (read from sites_config sheet)
- Switch node for site routing
- Site-specific credentials and validation
- Per-site daily post limits
- Site activation toggles
- Test with second site to validate isolation

**Addresses:**
- Table stakes: Multi-site support (critical for agency use)
- Differentiator: Centralized multi-site management

**Avoids:**
- Pitfall #4: Race conditions (implement full locking for concurrent sites)
- Pitfall #7: Memory limits (test batch sizes with multiple sites)
- Pitfall #10: Credential chaos (establish naming conventions)

**Research needed:** Low — Configuration pattern established in Phase 1, just expanding scope.

---

### Phase Ordering Rationale

**Dependency chain:**
- Google Sheets schema → all other phases (Phase 1 must be first)
- Topic queue → content generation (Phase 2 before Phase 3)
- Content generation → images (Phase 3 before Phase 4)
- Published content → internal linking (Phase 5 after Phase 6)
- Full pipeline → error handling (Phase 7 after core features)
- Single-site working → multi-site (Phase 8 last)

**Risk mitigation:**
- Start with data layer (Sheets) before automation logic
- Prove content quality (Phase 3) before publishing (Phase 6)
- Test with draft mode before auto-publish
- Validate single site before multi-site complexity

**Pitfall avoidance:**
- Establish validation protocol (Phase 1) before building complex workflows
- Build Turkish HTML conversion (Phase 3) before first publish
- Implement keyword segregation (Phase 2) before generating content at scale
- Add error handling (Phase 7) before production deployment

### Research Flags

**Phases needing deeper research during planning:**
- **Phase 3 (Content Generation Core):** Turkish prompt engineering for consistent quality — will need iterative testing and refinement with multiple models
- **Phase 5 (Internal Linking System):** Link insertion algorithm and contextual anchor text generation — custom implementation with limited prior art
- **Phase 7 (Error Handling):** OpenRouter rate limit specifics and optimal retry strategies — need to verify current 2026 API quotas

**Phases with standard patterns (skip research-phase):**
- **Phase 1 (Foundation):** Google Sheets schema and n8n credentials — well-documented, established patterns
- **Phase 2 (Topic Planning):** OpenRouter API integration — standard REST API calls
- **Phase 4 (Image Generation):** Google Imagen and WordPress media — official documentation available
- **Phase 6 (WordPress Publishing):** REST API v2 integration — mature, stable API with extensive documentation
- **Phase 8 (Multi-Site):** Configuration-driven architecture — pattern established in Phase 1

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | n8n capabilities verified from CLAUDE.md, OpenRouter and WordPress REST APIs are stable and well-documented, cost estimates based on current pricing |
| Features | MEDIUM | Table stakes features validated against content automation domain knowledge, but Turkish market specifics may reveal additional requirements |
| Architecture | HIGH | Multi-workflow separation and Google Sheets state management are proven patterns, n8n MCP documentation provides authoritative node details |
| Pitfalls | HIGH | Critical pitfalls identified from n8n architecture warnings (CLAUDE.md explicitly warns about default parameters), WordPress/AI integration patterns, and Turkish language considerations |

**Overall confidence: HIGH** for technical implementation, **MEDIUM** for market-fit assumptions (Turkish SEO requirements, content quality expectations).

### Gaps to Address

Research identified several areas needing validation during implementation:

- **OpenRouter model pricing and quotas:** Pricing may have changed since training data (Jan 2025), verify current rates for Claude 3.5 Sonnet, GPT-4 Turbo, and Perplexity models before finalizing budget. Also confirm per-model rate limits to calibrate batch sizes.

- **Turkish language prompt engineering:** No external sources consulted due to tool limitations. Will need A/B testing during Phase 3 to compare Claude vs GPT-4 for Turkish content quality, formality level, and natural phrasing. Establish quality baseline with first 10 posts before scaling.

- **Google Sheets API rate limits at scale:** Research assumes <100 posts/month volume. If scaling to 300+ posts, may hit read/write quotas. Monitor in Phase 7-8 and plan migration to PostgreSQL/Airtable if needed.

- **Internal linking algorithm effectiveness:** Keyword similarity algorithm for contextual linking is custom implementation. Needs validation with real content graph (20+ posts) to assess SEO value vs simpler hardcoded patterns.

- **WordPress custom field support:** Assumed Yoast SEO meta fields accessible via REST API. Verify during Phase 1 that mokadijital.com WordPress installation exposes required fields, or adjust to use RankMath alternative.

- **Google Imagen API availability and credits:** Research assumes existing Google Cloud credits. Confirm credit balance and Imagen 3 API access before Phase 4, have DALL-E 3 fallback ready if quota exhausted.

**Mitigation strategy:** All gaps have clear validation points in phase structure. None are blockers for starting Phase 1-2. Address incrementally as phases progress, with fallback options identified in STACK.md.

## Sources

### Primary (HIGH confidence)
- **CLAUDE.md (project file)** — n8n MCP integration patterns, OpenRouter API usage, critical warnings about default parameter values, n8n-specific best practices
- **n8n official documentation** (referenced in STACK.md) — node configurations, workflow patterns, error handling
- **WordPress REST API v2 documentation** (referenced in STACK.md) — post/media endpoints, authentication methods
- **OpenRouter API documentation** (referenced in STACK.md) — model selection, pricing, request formats
- **Google Sheets API v4 documentation** (referenced in STACK.md) — read/write operations, authentication

### Secondary (MEDIUM confidence)
- **SEO pillar-cluster strategy** (industry standard practice) — content architecture patterns, internal linking requirements
- **Content automation domain knowledge** (training data, pre-Jan 2025) — workflow separation patterns, state management approaches, quality validation gates
- **Turkish language NLP considerations** (training data, pre-Jan 2025) — character encoding issues, LLM performance on Turkish, prompt engineering patterns

### Tertiary (LOW confidence)
- **OpenRouter pricing estimates** — based on Jan 2025 training data, may have changed, verify before production
- **Google Imagen API capabilities** — limited community documentation compared to DALL-E, rely on official Google Cloud docs
- **Turkish SEO market specifics** — inferred from general SEO practices, may need local expert validation

### Research Limitations
- **No external web search performed** — all research based on training data (current through Jan 2025) and provided project context
- **API pricing subject to change** — cost estimates may not reflect current 2026 rates
- **Turkish language testing needed** — prompt engineering and quality assessment cannot be validated without live testing

---

*Research completed: 2026-02-15*

*Ready for roadmap: YES*

*Recommended next step: Create roadmap using 8-phase structure with dependencies as outlined. Flag Phase 3 and Phase 5 for additional research during detailed planning.*
