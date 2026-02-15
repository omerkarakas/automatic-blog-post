# N8N Otomatik Blog Üretim Sistemi - Tasarım Promptu

## Sistem Genel Bakış

N8n workflow'ları kullanarak WordPress blog'lar için otomatik içerik üretimi yapan, SEO optimizasyonlu bir sistem tasarla. Sistem pillar-cluster post stratejisiyle çalışacak ve tüm AI işlemleri OpenRouter API üzerinden gerçekleştirilecek.

---

## Sistem Gereksinimleri

### Teknik Altyapı
- **Otomasyon Platform**: n8n (self-hosted veya cloud)
- **Hedef Platform**: WordPress (MCP entegrasyonu ile)
- **AI Provider**: OpenRouter API (tüm LLM çağrıları için)
- **Görsel Üretimi**: DALL-E 3 veya Stable Diffusion (OpenRouter üzerinden)
- **Diyagram/İnfograf**: Mermaid.js veya programatik SVG oluşturma

### Entegrasyon Gereksinimleri
1. OpenRouter API anahtarı ve yapılandırması
2. WordPress MCP server kurulumu ve bağlantısı
3. Google Sheets veya Airtable (konu takibi için, opsiyonel)
4. Webhook receiver (workflow tetikleyiciler için)

---

## Workflow 1: Konu Planlayıcı (Topic Planner)

### Amaç
Pillar post ve ilişkili cluster post'ların konularını belirlemek, SEO keyword research yapmak ve içerik takvimi oluşturmak.

### Workflow Adımları

#### 1. Manuel Trigger / Webhook
- Workflow'u manuel olarak veya belirli aralıklarla tetikle
- Input parametreleri:
  - Ana niche/sektör (örn: "sağlıklı yaşam", "dijital pazarlama")
  - Hedef kitle profili
  - İstenen pillar post sayısı
  - Cluster post yoğunluğu (pillar başına kaç cluster)

#### 2. Ana Keyword Research (OpenRouter)
```javascript
// OpenRouter API call örneği
Model: claude-3.5-sonnet veya gpt-4
System Prompt:
"Sen bir SEO uzmanısın. Verilen niche için:
1. 5-10 ana pillar konusu belirle (geniş, yüksek arama hacimli)
2. Her pillar için 4-8 cluster konu öner (spesifik, long-tail)
3. Her konu için:
   - Hedef keyword
   - Arama amacı (informational/transactional/navigational)
   - Tahmini zorluk skoru
   - İlgili LSI keywords

JSON formatında döndür."
```

#### 3. Konu Validasyonu ve Düzenleme
- AI yanıtını parse et
- Konu hiyerarşisini oluştur
- Her pillar-cluster ilişkisini mapping'le

#### 4. Content Calendar Oluşturma
- Yayınlanma sırasını belirle (önce pillar, sonra cluster'lar)
- Tarih ataması (örn: haftada 2 post)
- Internal linking planı hazırla

#### 5. Veri Kaydı
- Google Sheets / Airtable'a kaydet veya
- n8n Database node ile sakla
- Şema:
  ```
  {
    post_id: unique_id,
    type: "pillar" | "cluster",
    parent_pillar_id: id | null,
    title: "...",
    target_keyword: "...",
    status: "planned" | "generated" | "published",
    publish_date: date,
    wordpress_post_id: id | null
  }
  ```

#### 6. Workflow 2'yi Tetikle
- Oluşturulan her konu için Workflow 2'yi sıraya koy
- İlk pillar post'tan başla

---

## Workflow 2: İçerik Üretici ve Yayınlayıcı

### Amaç
Belirlenen konularda expert-level fakat yalın dilde blog post'ları üretmek, görseller eklemek ve WordPress'e yayınlamak.

### Workflow Adımları

#### 1. Trigger - Konu Verisi
- Workflow 1'den gelen konu bilgisi
- Veya scheduled olarak database'den sıradaki konuyu çek

#### 2. İçerik Araştırması (OpenRouter - İlk Aşama)
```javascript
Model: claude-3.5-sonnet veya perplexity
System Prompt:
"Bu konu hakkında derinlemesine araştırma yap:
- Güncel istatistikler ve veriler
- Uzman görüşleri
- Yaygın yanılgılar
- Best practices
- Gerçek hayat örnekleri

Araştırma notları olarak JSON döndür."
```

#### 3. Outline Oluşturma (OpenRouter)
```javascript
Model: gpt-4-turbo veya claude-3.5-sonnet
System Prompt:
"Uzman bilgisi taşıyan ama 8. sınıf okuma seviyesinde anlaşılabilir bir blog post outline'ı oluştur:

Konu: {topic}
Tip: {pillar/cluster}
Hedef Kelime Sayısı: {pillar: 2500-3500, cluster: 1200-1800}

Outline şunları içermeli:
1. Çekici H1 başlık (SEO friendly)
2. Meta description (155 karakter)
3. Giriş paragrafı için bullet'lar
4. Ana bölümler (H2) ve alt başlıklar (H3)
5. Her bölümde ele alınacak noktalar
6. İnfograf/diyagram önerileri (hangi bölümde, ne göstermeli)
7. Kapanış call-to-action

JSON formatında döndür."
```

#### 4. Tam İçerik Yazımı (OpenRouter)
```javascript
Model: claude-3.5-sonnet (uzun içerik için ideal)
System Prompt:
"Sen konu hakkında uzman bir blog yazarısın. Şu prensipleri takip et:

TON VE DİL:
- Uzmanlık hissi ver ama jargon kullanma
- Basit, anlaşılır Türkçe (veya hedef dil)
- Aktif cümle yapıları
- Kısa paragraflar (3-4 cümle)
- Örnekler ve metaforlarla açıkla

YAPI:
- Outline'ı takip et
- Her H2 bölümü bağımsız okunabilir olsun
- Transition cümleleri kullan
- Bullet points ve numbered lists ekle

SEO:
- Target keyword'ü doğal şekilde yerleştir
- LSI keywords kullan
- Internal link anchor text'leri belirle
- Image alt text önerileri ekle

ENGAGING İÇERİK:
- İlk 100 kelimede hook oluştur
- Soru sorarak okuyucuyu düşündür
- Actionable insights ver
- İstatistik ve data kullan

Outline: {outline_json}
Research Data: {research_json}

Markdown formatında döndür (frontmatter ile)."
```

#### 5. İçerik Post-Processing
- Markdown'dan HTML'e çevir
- Internal link placeholder'ları tanımla:
  - Pillar post ise: cluster'lara link placehoalder'ları ekle
  - Cluster post ise: parent pillar'a ve sibling cluster'lara link ekle
- Image placeholder'ları işaretle

#### 6. Görsel Üretimi (OpenRouter - DALL-E 3)

**A. Featured Image**
```javascript
Prompt Template:
"Professional blog header image for article about {topic}.
Style: {modern/minimal/professional/vibrant}
Elements: {topic-specific visual metaphors}
No text, high quality, 16:9 aspect ratio"

API: OpenRouter - openai/dall-e-3
Size: 1792x1024
```

**B. İnfografikler ve Diyagramlar**
İki yaklaşım:

**Yaklaşım 1: AI ile Görsel (basit infografikler için)**
```javascript
Prompt:
"Simple infographic showing {data/concept}.
Clean design, pastel colors, clear labels,
educational style, vertical layout"
```

**Yaklaşım 2: Mermaid.js (karmaşık diyagramlar için)**
```javascript
// OpenRouter ile Mermaid code üret
System Prompt:
"Bu veri/akış için Mermaid.js diyagram kodu oluştur:
- Flowchart, sequence, veya graph
- Temiz, anlaşılır
- Renk kodlu

Sadece Mermaid syntax döndür."

// Sonra Mermaid CLI ile PNG'ye çevir
mermaid-cli input.mmd -o output.png
```

**C. SVG İllüstrasyonlar (opsiyonel)**
```javascript
// OpenRouter ile SVG code üret
System Prompt:
"Bu konsept için basit, modern SVG illustration oluştur:
Concept: {concept}
Style: Line art, minimal
Colors: {brand_colors}

Valid SVG code döndür."
```

#### 7. Görselleri WordPress'e Upload
- WordPress Media Library API kullan
- Her görsel için:
  - Alt text ekle (SEO için)
  - Caption ve description
  - Media ID'yi kaydet

#### 8. Internal Linking Logic

**A. Database Sorguları**
```javascript
// Eğer pillar post ise:
const relatedClusters = await getClustersByPillarId(current_pillar_id);

// Eğer cluster post ise:
const parentPillar = await getPillarById(parent_pillar_id);
const siblingClusters = await getClustersByPillarId(parent_pillar_id)
  .filter(c => c.id !== current_post_id);
```

**B. Link Enjeksiyonu**
```javascript
// Pillar post için
content = content.replace(
  '{{cluster_links}}',
  generateClusterLinkSection(relatedClusters)
);

// Cluster post için
content = content.replace(
  '{{pillar_link}}',
  `Bu konu hakkında daha detaylı bilgi için <a href="${parentPillar.url}">${parentPillar.title}</a> yazımıza göz atın.`
);

// Contextual link'ler
content = insertContextualLinks(content, relatedPosts);
```

**C. İki Yönlü Linking**
```javascript
// Yeni cluster yayınlandığında, parent pillar'ı güncelle
if (postType === 'cluster') {
  await updatePillarWithNewClusterLink(
    parentPillarId,
    currentPostId,
    currentPostTitle
  );
}
```

#### 9. WordPress MCP ile Yayınlama

```javascript
// WordPress MCP kullanımı
{
  "tool": "wordpress_create_post",
  "parameters": {
    "title": post_title,
    "content": final_html_content,
    "status": "publish", // veya "draft" review için
    "categories": category_ids,
    "tags": tag_ids,
    "featured_media": featured_image_media_id,
    "meta": {
      "seo_title": seo_title,
      "seo_description": meta_description,
      "focus_keyword": target_keyword
    },
    "post_type": "post",
    "author": author_id
  }
}
```

#### 10. Post-Publication İşlemler
- WordPress post ID'yi database'e kaydet
- Post URL'yi kaydet
- Status'ü "published" olarak güncelle
- Sonraki post'u tetikle (eğer varsa)

---

## İçerik Kalite Standartları

### Yazım Tonu
```
✅ DOĞRU ÖRNEKLER:
"Protein takviyesi, kaslarınızın toparlanmasına yardımcı olur. Antrenman sonrası vücudunuz hasarlı kas liflerini onarmak için protein kullanır."

"SEO'da backlink, başka bir web sitesinin sizin sayfanıza link vermesi demektir. Google bunu bir 'güven oyu' olarak görür."

❌ YANLIŞ ÖRNEKLER:
"Protein suplementasyonu, post-workout myofibrillar protein sentezini optimize eder."

"Backlink acquisition, domain authority augmentation için kritik bir off-page SEO parametredir."
```

### Yapı Standardı
```markdown
# H1: Ana Başlık (Post Title)

[Giriş - 150-200 kelime]
- Hook cümle
- Sorun tanımı
- Bu yazıda ne öğrenecekler

## H2: İlk Ana Bölüm

[3-4 paragraf]

### H3: Alt Konu 1
[2-3 paragraf]

[İnfograf placeholder]

### H3: Alt Konu 2
[2-3 paragraf]

## H2: İkinci Ana Bölüm

...

## H2: Sonuç ve Öneriler

[Özet + CTA]
```

---

## Örnek n8n Workflow Yapıları

### Workflow 1: Topic Planner Node Dizilimi
```
[Manual Trigger]
    ↓
[Set Niche Parameters]
    ↓
[HTTP Request: OpenRouter - Keyword Research]
    ↓
[Code: Parse & Structure Topics]
    ↓
[Google Sheets: Write Topics] (opsiyonel)
    ↓
[Split In Batches]
    ↓
[Webhook: Trigger Workflow 2] (her konu için)
```

### Workflow 2: Content Generator Node Dizilimi
```
[Webhook / Schedule Trigger]
    ↓
[Code: Get Next Topic from Queue]
    ↓
[HTTP Request: OpenRouter - Research]
    ↓
[HTTP Request: OpenRouter - Outline]
    ↓
[HTTP Request: OpenRouter - Full Content]
    ↓
[Code: Process Markdown → HTML]
    ↓
[Split: Image Generation Paths] ─┬─> [HTTP: DALL-E Featured]
                                   ├─> [HTTP: DALL-E Infographic 1]
                                   ├─> [Code: Mermaid → PNG]
                                   └─> [HTTP: DALL-E Infographic 2]
    ↓
[Merge: All Images]
    ↓
[HTTP: WordPress Media Upload] (batch)
    ↓
[Code: Insert Images to Content]
    ↓
[Code: Build Internal Links]
    ↓
[HTTP: WordPress MCP - Create Post]
    ↓
[Code: Update Database with Post ID]
    ↓
[Conditional: Update Parent Pillar if Cluster]
    ↓
[HTTP: WordPress MCP - Update Pillar] (eğer gerekiyorsa)
```

---

## OpenRouter API Yapılandırması

### API Credentials
```javascript
// n8n Credentials
{
  "name": "OpenRouter API",
  "type": "httpHeader",
  "data": {
    "name": "Authorization",
    "value": "Bearer YOUR_OPENROUTER_API_KEY"
  }
}
```

### Model Seçimi Stratejisi

**Keyword Research & Planning:**
- `anthropic/claude-3.5-sonnet` - Yapılandırılmış çıktı için
- `perplexity/llama-3.1-sonar-huge-128k-online` - Güncel arama için

**Content Writing:**
- `anthropic/claude-3.5-sonnet` - Uzun form içerik, Türkçe kalitesi
- `openai/gpt-4-turbo` - Alternatif (daha kreatif)

**Image Generation:**
- `openai/dall-e-3` - Yüksek kalite
- `stability-ai/stable-diffusion-xl` - Maliyet optimizasyonu

### Rate Limiting ve Error Handling
```javascript
// n8n HTTP Request node settings
{
  "retry": {
    "enabled": true,
    "maxTries": 3,
    "waitBetweenTries": 5000
  },
  "timeout": 120000, // 2 dakika (uzun içerik için)
  "ignoreResponseCode": false
}

// Code node - error handling
try {
  const response = await openRouterRequest(prompt);
  if (response.error) {
    throw new Error(`OpenRouter Error: ${response.error.message}`);
  }
  return response.choices[0].message.content;
} catch (error) {
  // Fallback model'e geç veya queue'ya geri koy
  return { error: error.message, retry: true };
}
```

---

## WordPress MCP Entegrasyonu

### MCP Server Kurulumu
```bash
# WordPress MCP server (npm package olarak)
npm install @modelcontextprotocol/server-wordpress

# Yapılandırma
{
  "wordpress_url": "https://yourblog.com",
  "username": "admin",
  "application_password": "xxxx xxxx xxxx xxxx"
}
```

### n8n'de MCP Kullanımı
```javascript
// HTTP Request Node ile MCP endpoint'e istek
{
  "method": "POST",
  "url": "http://localhost:3000/mcp/wordpress",
  "body": {
    "action": "create_post",
    "data": {
      "title": "...",
      "content": "...",
      "status": "publish"
    }
  }
}
```

### Kategoriler ve Tag'ler
```javascript
// İlk kurulumda kategoriler oluştur
const categories = [
  { name: "Pillar Posts", slug: "pillar-posts" },
  { name: niches[0], slug: slugify(niches[0]) },
  // ...
];

// Post'ta kullan
{
  "categories": [pillarCategoryId, nicheCategoryId],
  "tags": keywordTags.map(k => k.id)
}
```

---

## Görsel Optimizasyonu

### Image Processing Pipeline
```javascript
// Her görsel için
1. AI'dan al (DALL-E, SD, veya Mermaid)
2. Optimize et:
   - Resize: Max width 1200px
   - Compress: Quality 85% (JPEG)
   - Format: WebP'ye çevir (browser compat için)
3. WordPress'e upload
4. URL'leri content'e inject et
```

### Alt Text ve SEO
```javascript
// OpenRouter ile alt text üret
System Prompt:
"Bu görsel için SEO-friendly alt text oluştur:
Image Description: {image_context}
Target Keyword: {keyword}
Max 125 karakter, doğal ve açıklayıcı"

// Örnek Output:
"Protein tozu ve shaker ile spor salonunda antrenman yapan kişi"
```

---

## Monitoring ve Quality Control

### Workflow Monitoring
```javascript
// Her major step'te log kaydet
[Code Node - Logging]
console.log({
  workflow: "content_generator",
  post_id: postId,
  step: "content_written",
  word_count: wordCount,
  timestamp: new Date()
});

// n8n'in built-in execution history'sini kullan
// Error alerts için webhook'a bildir
```

### Content Quality Checks
```javascript
// Yayından önce otomatik kontroller
const qualityChecks = {
  minWordCount: postType === 'pillar' ? 2500 : 1200,
  hasHeadings: content.match(/<h2>/g)?.length >= 3,
  hasImages: content.match(/<img/g)?.length >= 2,
  hasInternalLinks: content.match(/href.*yourblog\.com/g)?.length >= 2,
  readabilityScore: calculateFleschScore(content) >= 60
};

if (!allChecksPassed(qualityChecks)) {
  // Draft olarak yayınla, manuel review bekle
  postStatus = 'draft';
}
```

---

## Ölçeklendirme ve Optimizasyon

### Batch Processing
```javascript
// Aynı anda çok fazla API call yapma
// Rate limiting için queue kullan
[Queue Node]
{
  "batchSize": 3, // Aynı anda max 3 post üret
  "delayBetweenBatches": 60000 // 1 dakika bekle
}
```

### Maliyet Optimizasyonu
```javascript
// Token kullanımını minimize et
- Outline'ı cached context olarak kullan
- Research data'yı summarize et
- Image generation: batch prompt'lar
- Model selection: task'a göre en uygun model

// Örnek token tahmini (1 post)
Research: ~2K tokens (input) + 1K (output)
Outline: ~1K (input) + 0.5K (output)
Content: ~3K (input) + 4K (output)
Images: ~3 image (100 tokens prompt each)
Total: ~11K tokens ≈ $0.20-0.50 per post (model'e bağlı)
```

---

## Örnek Kullanım Senaryosu

### Senaryo: "Sağlıklı Yaşam" Blog'u

#### 1. Topic Planning (Workflow 1)
```
Input:
- Niche: "Sağlıklı Yaşam"
- Hedef: "30-45 yaş arası, fit kalmak isteyen profesyoneller"
- Pillar Count: 3
- Cluster per Pillar: 5

Output:
Pillar 1: "Protein: Vücudunuzun İnşaat Malzemesi"
  ├─ Cluster 1.1: "Protein Tozları: Hangisi Size Uygun?"
  ├─ Cluster 1.2: "Protein Zamanlaması: Ne Zaman Almalı?"
  ├─ Cluster 1.3: "Bitkisel Protein Kaynakları Rehberi"
  ├─ Cluster 1.4: "Protein ve Kilo Verme İlişkisi"
  └─ Cluster 1.5: "Yüksek Proteinli 20 Pratik Tarif"

Pillar 2: "Kardiyovasküler Sağlık: Kalbiniz İçin 101"
  ├─ Cluster 2.1: "HIIT vs Steady-State Cardio"
  └─ ... (4 cluster daha)

Pillar 3: "Uyku: Gizli Performans Optimizasyonu"
  └─ ... (5 cluster)
```

#### 2. Content Generation (Workflow 2)

**Pillar 1 Üretimi:**
```
Step 1: Research
- AI protein biyokimyasini araştırır
- Güncel çalışmalar, uzman görüşleri

Step 2: Outline
- H1: Protein: Vücudunuzun İnşaat Malzemesi
- Meta: "Protein nedir, vücudunuzda nasıl çalışır ve..."
- H2: Protein Nedir ve Neden Önemlidir?
  - H3: Amino Asitler: Temel Yapı Taşları
  - H3: Vücudunuzda Protein'in 7 Önemli Görevi
- H2: Ne Kadar Protein Almalısınız?
  - [İnfograf: Günlük protein ihtiyacı hesaplayıcı]
- H2: En İyi Protein Kaynakları
  - [Tablo: Besin başına protein miktarı]
- H2: Protein Takviyesi Gerekli mi?
- H2: Sık Sorulan Sorular

Step 3: Full Content
- 3200 kelimelik detaylı içerik
- Basit dil, uzman bilgisi
- Örneklerle zenginleştirilmiş

Step 4: Visuals
- Featured: Fitness görseli (DALL-E)
- Infographic: Protein ihtiyacı kalkulatörü (Mermaid)
- Table: Besin tablosu (HTML)
- Diagram: Protein sindirim süreci (SVG)

Step 5: Internal Links
- 5 cluster post placeholder'ı
- İlgili glossary terimleri

Step 6: Publish
- WordPress'e post edildi
- URL: /protein-vucut-insaat-malzemesi/
- Post ID: 42 (database'e kaydedildi)
```

**Cluster 1.1 Üretimi (2 gün sonra):**
```
Parent Pillar: Post ID 42
Topic: "Protein Tozları: Hangisi Size Uygun?"

Content includes:
- Pillar'a link: "Protein'in vücuttaki rolü hakkında detaylı bilgi..."
- Whey, casein, plant-based karşılaştırması
- Seçim kriterleri infografiği
- Ürün önerileri (affiliate potansiyel)

After publish:
- Pillar post (ID 42) güncelleniyor
- İlgili bölüme cluster linki ekleniyor
```

---

## Gelişmiş Özellikler (Opsiyonel)

### A. Dinamik İçerik Güncellemesi
```javascript
// 6 ayda bir pillar post'ları güncelle
[Schedule Trigger: Every 6 months]
    ↓
[Get Published Pillar Posts]
    ↓
[HTTP: OpenRouter - "Bu içeriği güncel verilerle refresh et"]
    ↓
[WordPress MCP: Update Post]
```

### B. A/B Testing Başlıklar
```javascript
// İki farklı başlık versiyonu üret
// Hangisi daha çok tıklanıyor tracking'le
const titleVariants = await generateTitleVariants(topic);
// Analytics entegrasyonu gerekir
```

### C. Multi-Language Support
```javascript
// Aynı outline, farklı dillerde içerik
{
  languages: ['tr', 'en'],
  model: 'gpt-4-turbo' // Multi-lingual support
}
```

### D. Video Script Generation
```javascript
// Blog post'tan video script üret
System Prompt:
"Bu blog post'u 5 dakikalık YouTube video scriptine dönüştür"
// TTS ve video editor entegrasyonu ile tam otomasyona gidilebilir
```

---

## Troubleshooting ve Best Practices

### Yaygın Sorunlar

**1. API Rate Limits**
```javascript
// Solution: Exponential backoff
const delay = Math.pow(2, attemptNumber) * 1000;
await wait(delay);
```

**2. Tutarsız AI Çıktıları**
```javascript
// Solution: Strict JSON schema + validation
const schema = {
  type: "object",
  required: ["title", "content", "outline"],
  properties: { ... }
};
// JSON mode kullan (OpenRouter supports structured outputs)
```

**3. Görsel Kalite Düşük**
```javascript
// Solution: Prompt engineering
// Detaylı stil yönergeleri
// Reference image kullan (eğer API destekliyorsa)
```

**4. Internal Link Çakışmaları**
```javascript
// Solution: Conflict detection
// Database'de unique constraint'ler
// Link'leri inject etmeden önce validate et
```

### Best Practices Checklist

✅ Her workflow için error handling
✅ API key'leri environment variable olarak sakla
✅ Rate limiting implement et
✅ Üretilen içeriği human review'dan geçir (en azından ilk 10 post)
✅ Backup mekanizması (content database'e de kaydedilsin)
✅ Monitoring ve alerting kur
✅ Maliyeti track et (API usage)
✅ SEO plugin'leri kullan (Yoast, RankMath)
✅ Pillar ve cluster arası mesafeyi koru (cannibalization önleme)
✅ Düzenli quality audit yap

---

## Sonuç ve Başlangıç Adımları

Bu sistemi kurmak için:

1. **Önkoşullar:**
   - n8n kurulumu (self-hosted önerilir, cloud da olabilir)
   - WordPress + Application Password setup
   - OpenRouter hesabı + API key
   - (Opsiyonel) Google Sheets / Airtable hesabı

2. **İlk Kurulum:**
   - Workflow 1'i kur ve test et (1 pillar + 2 cluster üret)
   - Manuel olarak kaliteyi kontrol et
   - Workflow 2'yi kur, test post'ları yayınla

3. **Optimizasyon:**
   - Prompt'ları niche'ine göre fine-tune et
   - Görsel stilini belirle
   - İç linking stratejisini optimize et

4. **Otomasyon:**
   - Schedule trigger'ları aktive et
   - Batch processing ayarla
   - Monitoring kur

5. **Ölçeklendirme:**
   - Multi-blog desteği ekle
   - Farklı niche'ler için template'ler oluştur
   - Performans metriklerini takip et

---

**Son Not:** Bu sistem tamamen otomatik çalışabilir, ancak en iyi sonuçlar için insan denetimini tamamen kaldırma. İlk 20-30 post'u gözden geçir, sistem'in çıktı kalitesini anla, prompt'ları fine-tune et. Sonrasında otopilota alabilirsin ama düzenli quality check'leri sürdür.
