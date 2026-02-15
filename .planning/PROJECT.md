# Otomatik Blog Post Uretim Sistemi

## What This Is

n8n workflow'lari kullanarak WordPress bloglari icin otomatik, SEO-optimize iccerik ureten bir sistem. Pillar-cluster post stratejisiyle calisiyor. Tum AI islemleri OpenRouter API uzerinden, gorseller Google Imagen API ile uretiliyor. Birden fazla blog sitesi icin konfigure edilebilir sekilde tasarlandi.

## Core Value

Bir niche ve hedef kitle verildiginde, SEO uyumlu pillar-cluster yapisinda blog iceriklerini arastirmadan yayinlamaya kadar tamamen otomatik uretmek ve WordPress'e yayinlamak.

## Requirements

### Validated

(None yet -- ship to validate)

### Active

- [ ] Webhook/manuel tetikleme ile konu planlama workflow'u baslatilabilir
- [ ] Verilen niche icin pillar post konulari ve iliskili cluster post'lar otomatik belirlenir
- [ ] Her konu icin hedef keyword, arama amaci ve LSI keyword'ler uretilir
- [ ] Icerik takvimi olusturulur (yayin sirasi ve tarih atamasi)
- [ ] Tum konu verileri Google Sheets'e kaydedilir
- [ ] Her konu icin arastirma, outline ve tam icerik otomatik uretilir (OpenRouter)
- [ ] Icerik uzman bilgisi tasir ama basit, anlasilir Turkce ile yazilir
- [ ] Pillar post'lar 2500-3500, cluster post'lar 1200-1800 kelime olur
- [ ] Her post icin Google Imagen API ile featured image uretilir
- [ ] Markdown'dan HTML'e donusum ve post-processing yapilir
- [ ] Pillar-cluster arasi iki yonlu internal linking otomatik eklenir
- [ ] Icerik WordPress'e yayinlanir (site bazinda draft/publish konfigure edilebilir)
- [ ] Yayinlanan post'un ID ve URL'si Google Sheets'te guncellenir
- [ ] Sistem birden fazla WordPress sitesi icin konfigure edilebilir
- [ ] Kalite kontrolleri (kelime sayisi, baslik sayisi, gorsel, link) otomatik yapilir
- [ ] API rate limiting ve error handling mevcuttur

### Out of Scope

- Multi-language destegi -- v1 sadece Turkce, coklu dil sonraki surumlerde
- A/B test basliklari -- analytics entegrasyonu gerektirir, v2'de degerlendirilecek
- Video script uretimi -- temel sistem oncelikli, v2+ ozellik
- Icerik guncelleme (6 aylik refresh) -- v1'de yok, sistem oturdugundan sonra eklenecek
- Mermaid.js diyagramlar -- v1'de gorsel icin sadece Imagen kullanilacak
- SVG illustrasyonlar -- v1 kapsami disinda
- AI infografikler -- v1'de featured image yeterli

## Context

- n8n instance ai.mokadijital.com adresinde self-hosted olarak calisiyor
- Test sitesi: mokadijital.com/blog (WordPress)
- OpenRouter API hesabi hazir ve aktif
- Google Imagen API kredisi mevcut
- Proje bir lead generation girisiminin parcasi
- Prompt.md dosyasinda detayli teknik tasarim mevcut (workflow adimlari, node dizilimleri, API yapilandirmalari)

## Constraints

- **AI Provider**: Tum LLM cagrilari OpenRouter API uzerinden (model secimi: Claude 3.5 Sonnet, GPT-4 Turbo, Perplexity)
- **Gorsel**: Google Imagen API (DALL-E 3 yerine)
- **Platform**: n8n workflow otomasyon platformu (ai.mokadijital.com)
- **Hedef**: WordPress (REST API / MCP entegrasyonu)
- **Veri Saklama**: Google Sheets (konu takibi ve durum yonetimi)
- **Dil**: Sadece Turkce icerik
- **Maliyet**: API kullanimi token bazli, post basina ~$0.20-0.50 hedefi

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| OpenRouter API (tek AI provider) | Birden fazla modele tek API uzerinden erisim | -- Pending |
| Google Imagen (gorsel uretimi) | Mevcut kredi, DALL-E yerine maliyet avantaji | -- Pending |
| Google Sheets (veri saklama) | Kolay gorunturleme, manuel mudahale kolayligi | -- Pending |
| Site bazinda draft/publish | Farkli siteler icin farkli yayin politikasi | -- Pending |
| Gelismis ozellikler v2'ye ertelendi | Oncelik temel sistemin calisir olmasi | -- Pending |

---
*Last updated: 2026-02-15 after initialization*
