# Requirements: Otomatik Blog Post Uretim Sistemi

**Defined:** 2026-02-15
**Core Value:** Bir niche ve hedef kitle verildiginde, SEO uyumlu pillar-cluster yapisinda blog iceriklerini arastirmadan yayinlamaya kadar tamamen otomatik uretmek ve WordPress'e yayinlamak.

## v1 Requirements

### Konu Planlama (TOPIC)

- [ ] **TOPIC-01**: Kullanici niche/sektor ve hedef kitle profilini manuel olarak girebilir
- [ ] **TOPIC-02**: Verilen niche icin 5-10 pillar konu otomatik belirlenir (OpenRouter API)
- [ ] **TOPIC-03**: Her pillar icin 4-8 cluster konu otomatik uretilir
- [ ] **TOPIC-04**: Her konu icin hedef keyword, arama amaci (informational/transactional/navigational) ve LSI keyword'ler uretilir
- [ ] **TOPIC-05**: Konu hiyerarsisi olusturulur (pillar-cluster mapping)
- [ ] **TOPIC-06**: Icerik takvimi olusturulur (yayin sirasi: once pillar, sonra cluster'lar)
- [ ] **TOPIC-07**: Tum konu verileri Google Sheets'e kaydedilir (post_id, type, parent_pillar_id, title, target_keyword, status, publish_date)

### Icerik Uretimi (CONT)

- [ ] **CONT-01**: Her konu icin AI tabanli derinlemesine arastirma yapilir (guncel veriler, uzman gorusleri, best practices)
- [ ] **CONT-02**: Arastirma verilerine dayanarak yapilandirilmis outline uretilir (H1, meta description, H2/H3 bolumleri, kapsamlar)
- [ ] **CONT-03**: Outline ve arastirma verileriyle tam icerik yazilir (uzman bilgisi, basit Turkce, aktif cumleler, kisa paragraflar)
- [ ] **CONT-04**: Pillar post'lar 2500-3500 kelime, cluster post'lar 1200-1800 kelime olur
- [ ] **CONT-05**: Her post icin SEO-friendly meta title (50-60 karakter) uretilir
- [ ] **CONT-06**: Her post icin meta description (150-160 karakter) uretilir
- [ ] **CONT-07**: SEO-friendly URL slug otomatik uretilir
- [ ] **CONT-08**: Hedef keyword icerik icinde dogal sekilde yerlestirilir (title, H1, meta, govde)

### Gorsel Uretimi (IMG)

- [ ] **IMG-01**: Her post icin Google Imagen API ile featured image uretilir
- [ ] **IMG-02**: Uretilen gorseller WordPress medya kutuphanesine yuklenir
- [ ] **IMG-03**: Her gorsel icin SEO-friendly alt text otomatik uretilir

### Internal Linking (LINK)

- [ ] **LINK-01**: Cluster post'lar parent pillar post'a otomatik link icerir
- [ ] **LINK-02**: Pillar post'lar iliskili cluster post'lara otomatik link icerir
- [ ] **LINK-03**: Yeni cluster yayinlandiginda parent pillar post guncellenerek yeni cluster linki eklenir (iki yonlu linking)
- [ ] **LINK-04**: Kardes cluster post'lar arasinda iliskili linkler eklenir

### Yayinlama (PUB)

- [ ] **PUB-01**: Icerik WordPress MCP Server uzerinden post olarak olusturulur
- [ ] **PUB-02**: Gorseller WordPress MCP Server uzerinden medya kutuphanesine yuklenir
- [ ] **PUB-03**: Post durumu site bazinda konfigure edilebilir (draft veya publish)
- [ ] **PUB-04**: Kategori ve tag'ler otomatik atanir
- [ ] **PUB-05**: SEO meta verileri (title, description, focus keyword) post ile birlikte kaydedilir
- [ ] **PUB-06**: Sistem birden fazla WordPress sitesi icin konfigure edilebilir (multi-site)

### Veri Takibi (DATA)

- [ ] **DATA-01**: Google Sheets'te konu durumu takip edilir (planned/generated/published)
- [ ] **DATA-02**: Yayinlanan post'un WordPress post ID ve URL'si Google Sheets'te guncellenir
- [ ] **DATA-03**: Hata durumlarinda hata logu Google Sheets'e kaydedilir
- [ ] **DATA-04**: Her workflow calistirmasi icin temel loglama yapilir

### Kalite Kontrol (QA)

- [ ] **QA-01**: Yayinlamadan once minimum kelime sayisi kontrolu yapilir (pillar: 2500, cluster: 1200)
- [ ] **QA-02**: Yayinlamadan once minimum baslik sayisi kontrolu yapilir (en az 3 H2)
- [ ] **QA-03**: Yayinlamadan once gorsel varligi kontrolu yapilir
- [ ] **QA-04**: Yayinlamadan once internal link varligi kontrolu yapilir
- [ ] **QA-05**: Kalite kontrollerini gecemeyen icerik otomatik olarak draft statusunde kaydedilir

### Workflow Kontrol (WF)

- [ ] **WF-01**: Konu Planlayici workflow'u manuel veya webhook ile tetiklenebilir
- [ ] **WF-02**: Icerik Uretici workflow'u konu planlayicidan otomatik tetiklenir veya schedule ile calisir
- [ ] **WF-03**: API rate limiting ve exponential backoff ile error handling uygulanir
- [ ] **WF-04**: Basarisiz islemler icin retry mekanizmasi mevcuttur (max 3 deneme)

## v2 Requirements

### Gelismis Icerik

- **CONT-V2-01**: Cok asamali icerik iyilestirme (uret -> gozden gecir -> refine dongusu)
- **CONT-V2-02**: Okunabilirlik puanlamasi (Flesch-Kincaid benzeri Turkce metrik)
- **CONT-V2-03**: Intihal/benzerlik kontrolu

### Gelismis SEO

- **SEO-V2-01**: LSI keyword entegrasyonu
- **SEO-V2-02**: Schema markup (JSON-LD) uretimi
- **SEO-V2-03**: SERP feature optimizasyonu (featured snippet hedefleme)
- **SEO-V2-04**: Icerik tazeleme takibi (6 aylik refresh)

### Gelismis Konu Arastirma

- **TOPIC-V2-01**: Rakip icerik analizi
- **TOPIC-V2-02**: Trend konu tespiti
- **TOPIC-V2-03**: Konu bosluklari analizi

### Analitik

- **ANLY-V2-01**: GA4/Search Console performans takibi
- **ANLY-V2-02**: A/B test basliklari
- **ANLY-V2-03**: Icerik ROI raporlamasi

### Coklu Dil

- **LANG-V2-01**: Ingilizce icerik uretimi destegi
- **LANG-V2-02**: Dile ozgu SEO optimizasyonu

## Out of Scope

| Feature | Reason |
|---------|--------|
| Video script uretimi | Farkli medya, temel sistem oncelikli |
| Podcast script uretimi | Kapsam disi, blog icerigine odaklan |
| Sosyal medya paylasimi | Kapsam genislemesi, WordPress yayinlama yeterli |
| Ozel web dashboard | n8n UI ve Google Sheets yeterli |
| E-ticaret urun aciklamalari | Farkli kullanim alani |
| Otomatik site olusturma | WordPress sitelerinin mevcut oldugu varsayilir |
| Siyah sapka SEO (keyword stuffing, backlinking) | Google yonergelere aykiri |
| Gercek zamanli sohbet/isbirligi | Farkli domain |
| Ozel model egitimi | v1 icin asiri karmasik |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| TOPIC-01 | - | Pending |
| TOPIC-02 | - | Pending |
| TOPIC-03 | - | Pending |
| TOPIC-04 | - | Pending |
| TOPIC-05 | - | Pending |
| TOPIC-06 | - | Pending |
| TOPIC-07 | - | Pending |
| CONT-01 | - | Pending |
| CONT-02 | - | Pending |
| CONT-03 | - | Pending |
| CONT-04 | - | Pending |
| CONT-05 | - | Pending |
| CONT-06 | - | Pending |
| CONT-07 | - | Pending |
| CONT-08 | - | Pending |
| IMG-01 | - | Pending |
| IMG-02 | - | Pending |
| IMG-03 | - | Pending |
| LINK-01 | - | Pending |
| LINK-02 | - | Pending |
| LINK-03 | - | Pending |
| LINK-04 | - | Pending |
| PUB-01 | - | Pending |
| PUB-02 | - | Pending |
| PUB-03 | - | Pending |
| PUB-04 | - | Pending |
| PUB-05 | - | Pending |
| PUB-06 | - | Pending |
| DATA-01 | - | Pending |
| DATA-02 | - | Pending |
| DATA-03 | - | Pending |
| DATA-04 | - | Pending |
| QA-01 | - | Pending |
| QA-02 | - | Pending |
| QA-03 | - | Pending |
| QA-04 | - | Pending |
| QA-05 | - | Pending |
| WF-01 | - | Pending |
| WF-02 | - | Pending |
| WF-03 | - | Pending |
| WF-04 | - | Pending |

**Coverage:**
- v1 requirements: 41 total
- Mapped to phases: 0
- Unmapped: 41 (roadmap olusturulacak)

---
*Requirements defined: 2026-02-15*
*Last updated: 2026-02-15 after initial definition*
