# Feature Landscape

**Domain:** Automated Blog/Content Generation Systems (n8n-based, Pillar-Cluster SEO)
**Researched:** 2026-02-15
**Confidence:** MEDIUM (based on established patterns, n8n capabilities, and SEO automation domain knowledge)

## Table Stakes

Features users expect. Missing = product feels incomplete.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| **Topic/Keyword Research** |
| Manual topic input | Users need to seed the system | Low | Accept niche/target audience input |
| Keyword list generation | Core of SEO content strategy | Medium | Generate related keywords from seed topic |
| Pillar topic identification | Foundation of pillar-cluster model | Medium | Identify broad pillar topics in niche |
| Cluster topic generation | Creates content network around pillars | Medium | Generate 5-20 cluster topics per pillar |
| Search volume awareness | Prevents targeting zero-traffic keywords | Medium | Basic keyword research data integration |
| **Content Generation** |
| AI-powered content writing | Core automation value proposition | Medium | LLM integration (OpenRouter) required |
| Configurable content length | Different content types need different lengths | Low | Word count targets (500-3000+ words) |
| Structured content format | SEO and readability requirement | Medium | H2/H3 headings, paragraphs, lists |
| Meta title generation | SEO fundamental | Low | 50-60 character optimized titles |
| Meta description generation | SEO fundamental | Low | 150-160 character summaries |
| Slug/URL generation | Basic CMS requirement | Low | SEO-friendly URL slugs |
| **SEO Optimization** |
| Keyword targeting | Core SEO requirement | Medium | Primary keyword placement in title/H1/meta |
| Internal linking structure | Pillar-cluster model requirement | High | Link clusters to pillars, pillars to each other |
| SEO scoring/validation | Quality gate before publishing | Medium | Basic on-page SEO checks |
| **Image Management** |
| Featured image generation | Visual content is expected | Medium | AI image generation (Imagen) |
| Image alt text generation | SEO and accessibility requirement | Low | Descriptive alt text for images |
| **Publishing** |
| WordPress integration | Dominant CMS for blogs | Medium | WordPress REST API connection |
| Draft vs publish control | Users need review capability | Low | Configurable per site |
| Post scheduling | Content calendar management | Medium | Queue posts, publish on schedule |
| Multi-site support | Agencies/enterprises need this | Medium | Manage multiple WordPress sites |
| **Data Tracking** |
| Content status tracking | Users need visibility | Medium | Track what's generated/published |
| Publication log | Audit trail requirement | Low | When/where content published |
| Error logging | Debugging and reliability | Medium | Track failures in pipeline |
| **Workflow Control** |
| Manual trigger capability | On-demand generation | Low | Run workflows manually |
| Automated scheduling | Set-and-forget operation | Low | Cron-based triggers |
| Stop/pause capability | Control over automation | Low | Pause workflows without deletion |

## Differentiators

Features that set product apart. Not expected, but valued.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| **Advanced Topic Research** |
| Competitor content analysis | Better topic selection | High | Scrape/analyze competitor blogs |
| Trending topic detection | Timely, relevant content | High | Monitor trends in niche |
| Topic gap analysis | Find untapped opportunities | High | What competitors haven't covered |
| Semantic keyword clustering | More sophisticated SEO | High | Group keywords by intent |
| **Content Quality** |
| Multi-pass content refinement | Higher quality output | High | Generate → Review → Refine loop |
| Brand voice consistency | Enterprise requirement | High | Fine-tuned prompts per site |
| Fact-checking integration | Trust and accuracy | Very High | Verify claims against sources |
| Plagiarism detection | Legal/ethical requirement | Medium | Check against existing content |
| Readability scoring | User experience focus | Medium | Flesch-Kincaid, grade level |
| **Advanced SEO** |
| LSI keyword integration | Advanced on-page SEO | Medium | Latent semantic indexing keywords |
| Schema markup generation | Rich snippets advantage | Medium | JSON-LD structured data |
| Content freshness tracking | Update old content | Medium | Identify posts needing refresh |
| SERP feature optimization | Featured snippet targeting | High | Format for position zero |
| **Smart Internal Linking** |
| Contextual anchor text | Natural link building | High | Relevant, varied anchor text |
| Link opportunity detection | Maximize internal link value | High | Find best linking opportunities |
| Automatic link insertion | True automation | High | Insert links in existing content |
| Link graph visualization | Strategy visibility | Medium | See content relationship map |
| **Advanced Image Features** |
| Multiple images per post | Richer visual content | Medium | Header + in-content images |
| Image style consistency | Brand coherence | Medium | Consistent style parameters |
| Image compression/optimization | Performance optimization | Medium | Reduce file sizes automatically |
| Stock photo integration | Alternative to AI generation | Low | Unsplash/Pexels APIs |
| **Publishing Intelligence** |
| Optimal publishing time | Maximize engagement | Medium | Analytics-based timing |
| Content calendar view | Strategic planning | Medium | Visualize publication schedule |
| Category/tag automation | Content organization | Low | Auto-assign based on topic |
| Excerpt generation | Theme compatibility | Low | Custom excerpts for archives |
| **Quality Assurance** |
| Pre-publish preview | Confidence before publishing | Medium | Render preview of final post |
| Broken link checking | Quality control | Medium | Validate external links |
| Duplicate content detection | Avoid cannibalization | Medium | Check against own content |
| Human review queue | Hybrid automation | Medium | Flag posts for manual review |
| **Multi-Language Support** |
| Content translation | Market expansion | High | Beyond Turkish-only |
| Language-specific SEO | International SEO | High | hreflang, local keywords |
| **Analytics Integration** |
| Performance tracking | ROI visibility | Medium | GA4/Search Console integration |
| A/B testing | Optimization capability | High | Test headlines, content formats |
| Content ROI reporting | Business justification | Medium | Traffic/conversion per post |
| **Workflow Features** |
| Content templates | Consistency and efficiency | Medium | Reusable content structures |
| Custom prompt library | Flexibility per use case | Low | Save/reuse effective prompts |
| Conditional logic | Smart automation | Medium | Different paths for different content types |
| Webhook integrations | Extensibility | Low | Trigger external systems |

## Anti-Features

Features to explicitly NOT build. Common mistakes in this domain.

| Anti-Feature | Why Avoid | What to Do Instead |
|--------------|-----------|-------------------|
| **Spammy Generation** |
| Unlimited bulk generation | Creates low-quality spam | Enforce rate limits, quality gates |
| No human oversight option | Legal/brand risk | Always provide review capability |
| Auto-publish without review | High-risk for new system | Default to draft, opt-in to auto-publish |
| **Over-Automation** |
| Fully autonomous niche selection | Requires domain expertise | User provides niche/audience |
| Automatic site creation | Out of scope, complex | Assume WordPress sites exist |
| Automated social media posting | Scope creep | Focus on blog content first |
| Email newsletter automation | Different domain | WordPress publishing only |
| **Complex UI** |
| Custom web dashboard in v1 | Premature, resource-intensive | Use Google Sheets for tracking |
| Visual workflow builder | n8n already provides this | Leverage n8n UI |
| In-app analytics | Reinventing the wheel | Link to Google Analytics |
| **Fragile SEO Tactics** |
| Keyword stuffing automation | Against Google guidelines | Natural keyword integration |
| Automated backlinking | Black hat SEO | Internal linking only |
| Content spinning | Plagiarism risk | Original AI generation |
| Hidden text/cloaking | Penalty risk | Legitimate optimization only |
| **Premature Optimization** |
| Multi-model AI selection | Complexity without clear value | OpenRouter API with single model |
| Advanced NLP processing | Over-engineering | Use LLM capabilities directly |
| Custom image generation training | Very high complexity | Use Imagen API as-is |
| ML-based topic prediction | Overkill for v1 | Rule-based or simple algorithms |
| **Scope Creep** |
| Video content generation | Different medium entirely | Text/images only |
| Podcast script generation | Out of scope | Blog posts only |
| E-commerce product descriptions | Different use case | Editorial content focus |
| Landing page generation | Different format/purpose | Blog posts only |

## Feature Dependencies

```
Content Generation Flow:
Manual Topic Input → Keyword Research → Topic Generation → Content Writing → SEO Optimization → Image Generation → Publishing

Critical Dependencies:
- Internal Linking REQUIRES Content Generation (need existing content to link to)
- Multi-Pass Refinement REQUIRES Content Generation (need initial content)
- Pillar-Cluster Linking REQUIRES Topic Classification (know which are pillars vs clusters)
- Scheduling REQUIRES Publishing Integration (need WordPress connection)
- Status Tracking REQUIRES Data Storage (Google Sheets)
- Error Logging REQUIRES Workflow Execution (something to log)

Phase Ordering Implications:
1. Basic generation pipeline BEFORE advanced features
2. Single-site publishing BEFORE multi-site
3. Manual triggers BEFORE automation
4. Draft mode BEFORE auto-publish
5. Basic internal linking BEFORE advanced link intelligence
```

## MVP Recommendation

For MVP, prioritize:

### MUST HAVE (Table Stakes)
1. **Topic Planning Workflow**
   - Manual niche/audience input
   - Pillar topic identification (3-5 pillars)
   - Cluster topic generation (5-10 per pillar)
   - Google Sheets tracking

2. **Content Generation Workflow**
   - AI content writing (OpenRouter)
   - Structured format (title, H2/H3, paragraphs)
   - Meta title/description
   - Featured image (Imagen)
   - Image alt text
   - SEO slug generation

3. **Publishing**
   - WordPress API integration
   - Draft mode (default)
   - Single site support
   - Manual trigger

4. **Basic Internal Linking**
   - Cluster → Pillar links
   - Pillar → Pillar links
   - Hardcoded anchor text patterns

5. **Tracking**
   - Google Sheets status tracking
   - Publication log
   - Basic error logging

### SHOULD HAVE (Key Differentiators)
1. **Multi-site support** - Core value for agencies
2. **Configurable draft/publish** - Trust building
3. **Scheduled publishing** - True automation
4. **Basic SEO validation** - Quality assurance
5. **Turkish language optimization** - Specific market need

### DEFER TO POST-MVP

**Content Quality** (complexity vs value)
- Multi-pass refinement: High complexity, value uncertain until basic generation proven
- Fact-checking: Very high complexity, niche-dependent value
- Plagiarism detection: Medium complexity, can validate manually initially
- Readability scoring: Medium complexity, nice-to-have

**Advanced SEO** (premature optimization)
- LSI keywords: Medium complexity, marginal SEO value
- Schema markup: Medium complexity, can add manually if needed
- SERP feature optimization: High complexity, requires significant traffic first
- Content freshness tracking: Medium complexity, need content portfolio first

**Advanced Topic Research** (can be manual initially)
- Competitor analysis: High complexity, can research manually
- Trending topics: High complexity, can identify manually
- Topic gap analysis: High complexity, requires competitor data
- Semantic clustering: High complexity, simple lists work for v1

**Analytics Integration** (need content first)
- Performance tracking: Medium complexity, no content to track yet
- A/B testing: High complexity, need traffic first
- ROI reporting: Medium complexity, premature for v1

**Advanced Images** (diminishing returns)
- Multiple images per post: Medium complexity, featured image sufficient
- Image optimization: Medium complexity, WordPress handles this
- Stock photo integration: Low complexity but adds decision complexity

**Smart Internal Linking** (need content graph first)
- Contextual anchor text: High complexity, need content corpus
- Link opportunity detection: High complexity, requires analysis
- Automatic link insertion: High complexity, risky without testing
- Link graph visualization: Medium complexity, nice-to-have

## Complexity vs Value Matrix

### HIGH VALUE + LOW/MEDIUM COMPLEXITY (build first)
- Manual topic input
- Keyword list generation
- Basic content generation
- WordPress publishing
- Featured image generation
- Google Sheets tracking
- Multi-site support
- Draft/publish toggle

### HIGH VALUE + HIGH COMPLEXITY (build strategically)
- Internal linking automation (phased approach)
- SEO validation (start simple, enhance)
- Post scheduling (n8n native capability)
- Error handling (build incrementally)

### LOW VALUE + HIGH COMPLEXITY (skip for v1)
- Fact-checking integration
- Competitor content analysis
- A/B testing
- Custom analytics

### LOW VALUE + LOW COMPLEXITY (add opportunistically)
- Category/tag automation
- Excerpt generation
- Custom prompt library
- Stock photo integration

## Feature Maturity Path

### Phase 1: Core Pipeline
Focus: Prove content generation works end-to-end
- Topic planning (manual input → pillar/cluster identification)
- Content generation (AI writing → meta → images)
- WordPress publishing (draft mode, single site)
- Basic tracking (status in Google Sheets)

### Phase 2: Automation & Scale
Focus: Reduce manual intervention, support multiple sites
- Multi-site support
- Scheduled publishing
- Basic internal linking (cluster → pillar)
- Configurable draft/publish per site
- Error handling improvements

### Phase 3: Quality & Intelligence
Focus: Improve content quality and SEO effectiveness
- SEO validation/scoring
- Multi-pass content refinement
- Advanced internal linking (contextual)
- Readability optimization
- Link graph visibility

### Phase 4: Analytics & Optimization
Focus: Measure results, optimize performance
- Performance tracking (GA4/Search Console)
- Content freshness detection
- ROI reporting
- A/B testing capability
- Optimization recommendations

## Turkish Language Considerations

**Table Stakes for Turkish Content:**
- Turkish character support (ğ, ü, ş, ı, ö, ç)
- Turkish SEO keyword research (Turkish search behavior)
- Turkish grammar/syntax in LLM prompts
- Turkish meta descriptions (different optimal length patterns)

**Differentiators for Turkish Market:**
- Turkish readability scoring (adapted metrics)
- Turkish-specific SEO rules (different from English)
- Local search optimization (Turkey-specific)
- Turkish brand voice templates

**Note:** Most LLMs (OpenRouter models) handle Turkish well, but prompts must explicitly specify language. Imagen API supports Turkish text in prompts.

## n8n-Specific Feature Opportunities

**Leverage n8n Strengths:**
- Visual workflow debugging (built-in)
- Error retry logic (native capability)
- Webhook triggers (extensibility point)
- Scheduled triggers (cron-based automation)
- Data transformation nodes (flexible processing)
- HTTP request nodes (API integrations)
- Conditional routing (workflow logic)
- Loop/batch processing (handle multiple topics)

**Avoid in n8n:**
- Complex UI rendering (not n8n's strength)
- Real-time collaboration (use Sheets for this)
- Advanced data visualization (external tools better)
- User authentication (workflow-level, not app-level)

## OpenRouter API Features

**Available Capabilities:**
- Model selection (Claude, GPT-4, etc.)
- Token usage tracking
- Cost monitoring
- Rate limiting
- Multiple providers

**Use for MVP:**
- Single model selection (simplify)
- Basic error handling
- Cost tracking in Sheets

**Defer:**
- Multi-model comparison
- Automatic model selection
- Advanced rate limit management

## Google Imagen API Features

**Available Capabilities:**
- Text-to-image generation
- Image editing/inpainting
- Style control
- Safety filters
- Multiple aspect ratios

**Use for MVP:**
- Basic text-to-image (blog header)
- Single aspect ratio (featured image)
- Safety filters (default)

**Defer:**
- Multiple images per post
- Custom style training
- Image editing/variations
- Advanced aspect ratio handling

## Sources

**Knowledge Base:**
- Automated content generation system patterns (training data, pre-Jan 2025)
- n8n workflow automation capabilities (training data, pre-Jan 2025)
- Pillar-cluster SEO strategy principles (established SEO practice)
- WordPress REST API documentation (training data, pre-Jan 2025)
- OpenRouter API capabilities (training data, pre-Jan 2025)
- Google Imagen API features (training data, pre-Jan 2025)

**Confidence Note:**
This research is based on established patterns in content automation, SEO, and workflow automation domains. No external verification was performed due to tool limitations. Key assumptions:
- n8n capabilities are standard (HTTP, scheduling, error handling)
- WordPress REST API remains stable
- OpenRouter/Imagen APIs function as documented
- Pillar-cluster SEO model remains valid practice

**Verification Recommended:**
- OpenRouter API current model availability and pricing (check docs)
- Google Imagen API current capabilities and limits (check docs)
- n8n latest version features (check changelog)
- WordPress REST API authentication methods (check docs)
