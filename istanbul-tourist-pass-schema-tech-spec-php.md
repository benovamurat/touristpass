# Istanbul Tourist Pass® — Schema Markup Tech Spec

> **Versiyon:** 3.1 (Language-Agnostic Edition)  
> **Tarih:** 30 Mart 2026  
> **Kapsam:** v26.istanbultouristpass.com  
> **Format:** JSON-LD via `<script type="application/ld+json">`  
> **Hedef Kitle:** Backend & Frontend Developer (herhangi bir stack)  
> **Template Syntax:** `{{ variable }}` — kendi framework'ünüze çevirin

---

## İçindekiler

1. [Mimari Kararlar & Kurallar](#1-mimari-kararlar--kurallar)
2. [Global Bileşenler (Her Sayfada)](#2-global-bileşenler-her-sayfada)
3. [Anasayfa](#3-anasayfa)
4. [Attraction Listing](#4-attraction-listing)
5. [Attraction Detay](#5-attraction-detay)
6. [Blog Listing](#6-blog-listing)
7. [Blog Detay](#7-blog-detay)
8. [Blog Kategori](#8-blog-kategori)
9. [FAQ Sayfası](#9-faq-sayfası)
10. [Pass & Fiyatlar](#10-pass--fiyatlar)
11. [Reviews Sayfası](#11-reviews-sayfası)
12. [About Sayfası](#12-about-sayfası)
13. [Contact Sayfası](#13-contact-sayfası)
14. [How It Works](#14-how-it-works-sayfası)
15. [Compare Passes](#15-compare-passes-sayfası)
16. [Download App](#16-download-app-sayfası)
17. [Plan & Save](#17-plan--save-sayfası)
18. [What You Get](#18-what-you-get-sayfası)
19. [Free Digital Guidebook](#19-free-digital-guidebook-sayfası)
20. [Group Request](#20-group-request-sayfası)
21. [Special Offer & Service Sayfaları](#21-special-offer--service-sayfaları)
22. [Partner & Affiliate](#22-partner--affiliate-sayfaları)
23. [Legal Sayfaları](#23-legal-sayfaları)
24. [Implementasyon Checklist](#24-implementasyon-checklist)

---

## 1. Mimari Kararlar & Kurallar

### 1.1 Bileşen Mimarisi

Schema'lar **3 katmanda** üretilir. Her katman ayrı bir fonksiyon veya bileşen olmalıdır:

```
┌─────────────────────────────────────────────────┐
│  KATMAN 1: Global Schemas (her sayfada)         │
│  → generateGlobalSchemas(page)                  │
│  → Organization, WebSite, BreadcrumbList        │
├─────────────────────────────────────────────────┤
│  KATMAN 2: Sayfa Tipi Schemas                   │
│  → generatePageSchemas(pageType, pageData)      │
│  → Product, FAQPage, BlogPosting, Event vs.     │
├─────────────────────────────────────────────────┤
│  KATMAN 3: Widget/Section Schemas (opsiyonel)   │
│  → generateReviewSchema(reviews)                │
│  → generateVideoSchema(video)                   │
│  → Sayfa içi bileşenlere bağlı                  │
└─────────────────────────────────────────────────┘
```

### 1.2 @id Referans Sistemi

Her entity'nin benzersiz bir `@id` URI'si olmalıdır. Tam tanım **bir kez** yapılır, diğer sayfalarda sadece referans verilir.

| Entity | @id URI |
|--------|---------|
| Organization | `https://www.istanbultouristpass.com/#organization` |
| WebSite | `https://www.istanbultouristpass.com/#website` |
| Ana Ürün | `https://www.istanbultouristpass.com/#product` |
| Her sayfa | `https://www.istanbultouristpass.com/{slug}/#webpage` |
| Her attraction | `https://www.istanbultouristpass.com/{slug}/#attraction` |
| Her blog yazısı | `https://www.istanbultouristpass.com/blog/{cat}/{slug}/#article` |
| Her yazar | `https://www.istanbultouristpass.com/authors/{slug}/#person` |

### 1.3 Dinamik Değişken Notasyonu

Bu dökümandaki template şablonlarda `{{ variable }}` kullanılmıştır. Gerçek çıktı örneklerinde ise render edilmiş nihai JSON gösterilmiştir. Kendi stack'inize göre uyarlayın.

### 1.4 JSON Güvenliği

Tüm kullanıcı kaynaklı string'ler (`name`, `description`, `reviewBody`) JSON-LD'ye basılmadan önce:

- HTML tag'leri temizlenmeli (`strip_tags`)
- JSON-breaking karakterler escape edilmeli (`"`, `\n`, `\r`)
- Çoklu boşluklar tek boşluğa indirilmeli
- Maksimum 5.000 karakter ile sınırlandırılmalı (Google önerisi)

Bu işlemi yapan bir `safeJson(value, maxLength)` helper fonksiyonu yazmanız önerilir.

### 1.5 Sabit Değerler Referansı

Aşağıdaki değerler merkezi bir config/ayar dosyasından çekilmeli, hardcode edilmemelidir:

| Anahtar | Değer |
|---|---|
| `base_url` | `https://www.istanbultouristpass.com` |
| `site_name` | `Istanbul Tourist Pass` |
| `product_name` | `Istanbul Tourist Pass®` |
| `currency` | `EUR` |
| `contact_email` | `contact@istanbultouristpass.com` |
| `contact_phone` | `+908503048726` |
| `whatsapp_url` | `https://wa.me/905303852026` |
| `logo_url` | `https://www.istanbultouristpass.com/images/itp-logo.svg` |
| `og_image_url` | `https://www.istanbultouristpass.com/images/itp-og.jpg` |
| `founding_date` | `2015` |
| `default_author` | `ITP Editorial Team` |
| `return_days` | `30` |
| `ios_app_url` | `https://apps.apple.com/tr/app/istanbul-tourist-pass/id6535683117` |
| `android_app_url` | `https://play.google.com/store/apps/details?id=com.istanbultouristpass.app` |
| `social_links` | Facebook, Instagram, YouTube, TikTok, TripAdvisor URL'leri |
| `supported_languages` | English, Turkish, German, Russian, French, Spanish, Italian, Arabic |

### 1.6 hreflang Dil Kodu Eşleştirmesi

| Site kodu | hreflang kodu | URL prefix |
|---|---|---|
| en | `en` | *(yok — default)* |
| es | `es` | `/es` |
| it | `it` | `/it` |
| fr | `fr` | `/fr` |
| de | `de` | `/de` |
| ru | `ru` | `/ru` |
| tr | `tr` | `/tr` |
| ar | `ar` | `/ar` |
| bg | `bg` | `/bg` |
| cn | `zh-Hans` | `/cn` |
| gr | `el` | `/gr` |
| hr | `hr` | `/hr` |
| hu | `hu` | `/hu` |
| id | `id` | `/id` |
| il | `he` | `/il` |
| in | `hi` | `/in` |
| ir | `fa` | `/ir` |
| jp | `ja` | `/jp` |
| kr | `ko` | `/kr` |
| mk | `mk` | `/mk` |
| pk | `ur` | `/pk` |
| pl | `pl` | `/pl` |
| pt | `pt` | `/pt` |
| ro | `ro` | `/ro` |
| tw | `zh-Hant` | `/tw` |

> **⚠️ DİKKAT:** English prefix boş string olduğundan, naif URL birleştirme `https://...com//slug` gibi çift slash üretebilir. URL birleştirmede trim koruması ekleyin.

### 1.7 Fallback Kuralları (Genel)

| Durum | Fallback Davranışı |
|---|---|
| `image` alanı boş | Site logosunu / OG image'ı bas |
| `description` alanı boş | `name` alanını kullan veya alanı tamamen **çıkar** |
| `price` null veya 0 | Product schema'sının **tamamını basma** |
| `reviewCount` = 0 | `aggregateRating` bloğunun **tamamını çıkar** |
| `author` bilgisi yok | Default author kullan (config'den) |
| `datePublished` yok | Alanı tamamen çıkar (yanlış tarih basmaktan iyidir) |
| `geo` koordinatları yok | `geo` bloğunu tamamen çıkar |
| Sayfa draft/unpublished | Schema **hiç basılmamalı** |

> **Altın Kural:** Eksik veriyle bozuk JSON basmaktansa, o schema bloğunu hiç basmamak daha iyidir. Google bozuk JSON-LD'yi penalize eder.

---

## 2. Global Bileşenler (Her Sayfada)

> Bu bileşenler layout/head template'inde **bir kez** tanımlanır ve tüm sayfalara otomatik eklenir.

### 2.1 Organization — Sadece Anasayfada Tam Tanım

```
📊 VERİ KAYNAĞI: Merkezi config / ayarlar
🔄 GÜNCELLEME: Nadiren değişir — deploy-time cache'lenebilir
⚠️ KURAL: Tam tanım YALNIZCA anasayfada. Diğer sayfalar @id referansı ile erişir.
```

**Template:**

```json
{
    "@context": "https://schema.org",
    "@type": "Organization",
    "@id": "{{ config.base_url }}/#organization",
    "name": "{{ config.site_name }}",
    "alternateName": ["ITP", "Istanbul Tourist Pass®"],
    "url": "{{ config.base_url }}",
    "logo": {
        "@type": "ImageObject",
        "url": "{{ config.logo_url }}",
        "width": 300,
        "height": 60
    },
    "image": "{{ config.og_image_url }}",
    "description": "{{ config.site_description | safeJson }}",
    "foundingDate": "{{ config.founding_date }}",
    "email": "{{ config.contact_email }}",
    "telephone": "{{ config.contact_phone }}",
    "sameAs": {{ config.social_links | toJsonArray }},
    "contactPoint": [
        ... {{ config'den dinamik }}
    ]
}
```

**Gerçek Çıktı:**

```json
{
    "@context": "https://schema.org",
    "@type": "Organization",
    "@id": "https://www.istanbultouristpass.com/#organization",
    "name": "Istanbul Tourist Pass",
    "alternateName": ["ITP", "Istanbul Tourist Pass®"],
    "url": "https://www.istanbultouristpass.com",
    "logo": {
        "@type": "ImageObject",
        "url": "https://www.istanbultouristpass.com/images/itp-logo.svg",
        "width": 300,
        "height": 60
    },
    "image": "https://www.istanbultouristpass.com/images/itp-og.jpg",
    "description": "Istanbul's One & Only Digital Sightseeing Pass. Save over 65% on 120+ attractions.",
    "foundingDate": "2015",
    "email": "contact@istanbultouristpass.com",
    "telephone": "+908503048726",
    "address": {
        "@type": "PostalAddress",
        "addressLocality": "Istanbul",
        "addressRegion": "Istanbul",
        "addressCountry": "TR"
    },
    "areaServed": {
        "@type": "City",
        "name": "Istanbul",
        "sameAs": "https://en.wikipedia.org/wiki/Istanbul"
    },
    "sameAs": [
        "https://www.facebook.com/istanbultouristpass",
        "https://www.instagram.com/istanbultouristpass",
        "https://www.youtube.com/@istanbultouristpass",
        "https://www.tiktok.com/@istanbultouristpass",
        "https://www.tripadvisor.com/AttractionProductReview-g293974-d12464056-Istanbul_Tourist_Pass"
    ],
    "contactPoint": [
        {
            "@type": "ContactPoint",
            "telephone": "+908503048726",
            "contactType": "customer service",
            "availableLanguage": ["English","Turkish","German","Russian","French","Spanish","Italian","Arabic"],
            "areaServed": "Worldwide",
            "hoursAvailable": {
                "@type": "OpeningHoursSpecification",
                "dayOfWeek": ["Monday","Tuesday","Wednesday","Thursday","Friday","Saturday","Sunday"],
                "opens": "08:00",
                "closes": "22:00"
            }
        },
        {
            "@type": "ContactPoint",
            "contactType": "customer service",
            "url": "https://wa.me/905303852026",
            "name": "WhatsApp Support"
        }
    ],
    "slogan": "See More Pay Less"
}
```

---

### 2.2 WebSite + SearchAction — Sadece Anasayfada

**Gerçek Çıktı:**

```json
{
    "@context": "https://schema.org",
    "@type": "WebSite",
    "@id": "https://www.istanbultouristpass.com/#website",
    "name": "Istanbul Tourist Pass",
    "url": "https://www.istanbultouristpass.com",
    "publisher": {
        "@id": "https://www.istanbultouristpass.com/#organization"
    },
    "inLanguage": "en",
    "potentialAction": {
        "@type": "SearchAction",
        "target": {
            "@type": "EntryPoint",
            "urlTemplate": "https://www.istanbultouristpass.com/search?q={search_term_string}"
        },
        "query-input": "required name=search_term_string"
    }
}
```

> **Not:** Sitede arama özelliği yoksa `potentialAction` bloğunu çıkarın.

---

### 2.3 BreadcrumbList — Her Sayfada

```
⚠️ KURAL: Breadcrumb dizisi sayfa controller/route'undan dinamik gelmeli. Statik yazmayın.
⚠️ FALLBACK: Dizi boş veya 1 elemanlıysa (anasayfa) → schema basma.
⚠️ KURAL: Son elemanın item (URL) alanı olmaz — sadece name.
🔄 DÖNGÜ: Breadcrumb dizisi üzerinde iterasyon ile üretilir.
```

**Template:**

```json
{
    "@context": "https://schema.org",
    "@type": "BreadcrumbList",
    "itemListElement": [
        // Her {{ crumb }} için:
        {
            "@type": "ListItem",
            "position": {{ loop.index }},
            "name": "{{ crumb.label | safeJson }}",
            "item": "{{ crumb.url }}"   // ← son elemanda bu alan OLMAZ
        }
    ]
}
```

**Gerçek Çıktı (Hagia Sophia sayfası):**

```json
{
    "@context": "https://schema.org",
    "@type": "BreadcrumbList",
    "itemListElement": [
        {
            "@type": "ListItem",
            "position": 1,
            "name": "Home",
            "item": "https://www.istanbultouristpass.com"
        },
        {
            "@type": "ListItem",
            "position": 2,
            "name": "Attractions",
            "item": "https://www.istanbultouristpass.com/whats-included"
        },
        {
            "@type": "ListItem",
            "position": 3,
            "name": "Hagia Sophia Guided Tour with Skip-the-Ticket-Line Entry"
        }
    ]
}
```

**Breadcrumb hiyerarşi tablosu:**

| Sayfa | Yapı |
|---|---|
| `/whats-included` | Home → Attractions |
| `/hagia-sophia-guided-tour-...` | Home → Attractions → Hagia Sophia Guided Tour |
| `/blog` | Home → Blog |
| `/blog/tips-guides` | Home → Blog → Tips & Guides |
| `/blog/tips-guides/istanbul-travel-mistakes` | Home → Blog → Tips & Guides → Istanbul Travel Mistakes |
| `/istanbul-pass` | Home → Passes & Prices |
| `/frequently-asked-questions` | Home → FAQ |
| `/about-us` | Home → About Us |
| `/contact` | Home → Contact |
| `/reviews` | Home → Reviews |
| `/how-it-works` | Home → How It Works |

---

### 2.4 hreflang — Her Sayfada

```
⚠️ DİKKAT: English prefix boş olduğundan çift slash riski var — URL birleştirmede trim kullanın.
🔄 DÖNGÜ: Bölüm 1.6'daki dil tablosu üzerinde iterasyon.
```

**Gerçek Çıktı (anasayfa):**

```html
<link rel="alternate" hreflang="en" href="https://www.istanbultouristpass.com" />
<link rel="alternate" hreflang="es" href="https://www.istanbultouristpass.com/es" />
<link rel="alternate" hreflang="tr" href="https://www.istanbultouristpass.com/tr" />
<!-- ... 25 dil için devam eder -->
<link rel="alternate" hreflang="x-default" href="https://www.istanbultouristpass.com" />
```

**Gerçek Çıktı (attraction detay):**

```html
<link rel="alternate" hreflang="en" href="https://www.istanbultouristpass.com/hagia-sophia-guided-tour-..." />
<link rel="alternate" hreflang="tr" href="https://www.istanbultouristpass.com/tr/hagia-sophia-guided-tour-..." />
<!-- ... 25 dil için devam eder -->
<link rel="alternate" hreflang="x-default" href="https://www.istanbultouristpass.com/hagia-sophia-guided-tour-..." />
```

---

## 3. Anasayfa

**URL:** `/`

### 3.1 Product + AggregateOffer

```
📊 VERİ KAYNAĞI: passes tablosu (aktif olanlar) + reviews tablosu (aggregate)
🔄 GÜNCELLEME: Fiyat veya review değiştiğinde otomatik
⚠️ FALLBACK: reviewCount < 1 → aggregateRating bloğunu çıkar
⚠️ FALLBACK: Hiç aktif pass yoksa → bu schema'yı basma
⚠️ FALLBACK: Fiyat verisi yoksa → schema basma
```

**Template:**

```json
{
    "@context": "https://schema.org",
    "@type": "Product",
    "@id": "{{ config.base_url }}/#product",
    "name": "{{ config.product_name }}",
    "description": "{{ config.product_description | safeJson }}",
    "brand": { "@id": "{{ config.base_url }}/#organization" },
    "image": "{{ config.og_image_url }}",
    "category": "Sightseeing Pass",
    "offers": {
        "@type": "AggregateOffer",
        "lowPrice": "{{ passes | minPrice }}",
        "highPrice": "{{ passes | maxPrice }}",
        "priceCurrency": "{{ config.currency }}",
        "offerCount": "{{ passes | activeCount }}",
        "availability": "https://schema.org/InStock",
        "url": "{{ config.base_url }}/istanbul-pass",
        "seller": { "@id": "{{ config.base_url }}/#organization" },
        "priceValidUntil": "{{ currentYear }}-12-31"
    },
    // ⚠️ SADECE reviewCount > 0 ise ekle:
    "aggregateRating": {
        "@type": "AggregateRating",
        "ratingValue": "{{ reviews.averageRating | round(1) }}",
        "reviewCount": "{{ reviews.totalCount }}",
        "bestRating": "5",
        "worstRating": "1"
    }
}
```

**Gerçek Çıktı:**

```json
{
    "@context": "https://schema.org",
    "@type": "Product",
    "@id": "https://www.istanbultouristpass.com/#product",
    "name": "Istanbul Tourist Pass®",
    "description": "Istanbul's original digital sightseeing pass — access 120+ attractions, skip the lines, save over 65%.",
    "brand": {
        "@id": "https://www.istanbultouristpass.com/#organization"
    },
    "image": "https://www.istanbultouristpass.com/images/itp-og.jpg",
    "category": "Sightseeing Pass",
    "offers": {
        "@type": "AggregateOffer",
        "lowPrice": "49",
        "highPrice": "289",
        "priceCurrency": "EUR",
        "offerCount": "3",
        "availability": "https://schema.org/InStock",
        "url": "https://www.istanbultouristpass.com/istanbul-pass",
        "seller": {
            "@id": "https://www.istanbultouristpass.com/#organization"
        },
        "priceValidUntil": "2026-12-31"
    },
    "aggregateRating": {
        "@type": "AggregateRating",
        "ratingValue": "4.8",
        "reviewCount": "10000",
        "bestRating": "5",
        "worstRating": "1"
    }
}
```

---

### 3.2 Review Örnekleri

```
📊 VERİ KAYNAĞI: reviews tablosu (is_featured = true, is_approved = true, rating >= 4)
🔄 DÖNGÜ: Döngü ile max 5 review. Statik yazmayın — DB'den çekin.
⚠️ FALLBACK: 0 featured review → bu schema'yı tamamen çıkar
```

**Gerçek Çıktı:**

```json
{
    "@context": "https://schema.org",
    "@type": "Product",
    "@id": "https://www.istanbultouristpass.com/#product",
    "name": "Istanbul Tourist Pass®",
    "review": [
        {
            "@type": "Review",
            "author": { "@type": "Person", "name": "Justin Savoy" },
            "datePublished": "2025-03-15",
            "reviewRating": { "@type": "Rating", "ratingValue": "5", "bestRating": "5" },
            "reviewBody": "Istanbul Tourist Pass is absolutely incredible! We purchased the five day pass and absolutely got our moneys worth."
        },
        {
            "@type": "Review",
            "author": { "@type": "Person", "name": "Ondra Dvoulety" },
            "datePublished": "2025-05-10",
            "reviewRating": { "@type": "Rating", "ratingValue": "5", "bestRating": "5" },
            "reviewBody": "We took the Istanbul pass for three days and everything worked perfectly. Customer service always available via WhatsApp."
        }
    ]
}
```

---

### 3.3 HowTo (Nasıl Çalışır)

```
📊 VERİ KAYNAĞI: CMS veya config (nadiren değişir — semi-static OK)
🔄 DÖNGÜ: Adım dizisi üzerinde iterasyon
```

**Gerçek Çıktı:**

```json
{
    "@context": "https://schema.org",
    "@type": "HowTo",
    "name": "How to Use Istanbul Tourist Pass",
    "description": "Simple, Fast, and Hassle-Free — 3 steps to explore Istanbul.",
    "totalTime": "PT5M",
    "step": [
        {
            "@type": "HowToStep",
            "position": 1,
            "name": "Get Your Pass",
            "text": "Complete your purchase, and receive your Pass ID with usage details."
        },
        {
            "@type": "HowToStep",
            "position": 2,
            "name": "Download & Plan",
            "text": "Download & activate your pass, book attractions, organize your days and track your credit points — all in one app."
        },
        {
            "@type": "HowToStep",
            "position": 3,
            "name": "Explore & Enjoy",
            "text": "Just Show&Go — all digital, eco-friendly access to 120+ attractions with your e-ticket QR codes."
        }
    ]
}
```

---

### 3.4 SoftwareApplication (Mobil Uygulama)

```
📊 VERİ KAYNAĞI: config (app store URL'leri)
⚠️ FALLBACK: App store URL'leri yoksa bu schema'yı basma
```

**Gerçek Çıktı:**

```json
{
    "@context": "https://schema.org",
    "@type": "SoftwareApplication",
    "name": "Istanbul Tourist Pass",
    "operatingSystem": "iOS 15.0+, Android 8.0+",
    "applicationCategory": "TravelApplication",
    "description": "Download the Istanbul Tourist Pass app to access your digital pass, manage reservations, and explore Istanbul with audio guides in 25 languages.",
    "offers": { "@type": "Offer", "price": "0", "priceCurrency": "EUR" },
    "author": { "@id": "https://www.istanbultouristpass.com/#organization" },
    "installUrl": "https://apps.apple.com/tr/app/istanbul-tourist-pass/id6535683117",
    "downloadUrl": "https://play.google.com/store/apps/details?id=com.istanbultouristpass.app"
}
```

---

### 3.5 SiteNavigationElement

```
📊 VERİ KAYNAĞI: Navigasyon/menü tablosu veya config
🔄 DÖNGÜ: Ana menü item'ları üzerinden
💡 ÖNCELİK: 🟢 BONUS — Sprint 3
```

**Gerçek Çıktı:**

```json
{
    "@context": "https://schema.org",
    "@type": "SiteNavigationElement",
    "name": "Main Navigation",
    "hasPart": [
        { "@type": "WebPage", "name": "Attractions", "url": "https://www.istanbultouristpass.com/whats-included" },
        { "@type": "WebPage", "name": "Passes & Prices", "url": "https://www.istanbultouristpass.com/istanbul-pass" },
        { "@type": "WebPage", "name": "Compare Passes", "url": "https://www.istanbultouristpass.com/compare-passes" },
        { "@type": "WebPage", "name": "Blog", "url": "https://www.istanbultouristpass.com/blog" },
        { "@type": "WebPage", "name": "FAQs", "url": "https://www.istanbultouristpass.com/frequently-asked-questions" },
        { "@type": "WebPage", "name": "Contact Us", "url": "https://www.istanbultouristpass.com/contact" }
    ]
}
```

---

### 3.6 VideoObject (Testimonial Videoları)

```
📊 VERİ KAYNAĞI: video_testimonials tablosu
🔄 DÖNGÜ: Her video için ayrı bir schema
⚠️ FALLBACK: Video yoksa basma. duration yoksa alanı çıkar.
```

**Gerçek Çıktı (tek bir video):**

```json
{
    "@context": "https://schema.org",
    "@type": "VideoObject",
    "name": "Alina from Ukraine — Istanbul Tourist Pass Review",
    "description": "Real traveler Alina shares her Istanbul Tourist Pass experience.",
    "thumbnailUrl": "https://img.youtube.com/vi/7JQ-kfExIf8/hqdefault.jpg",
    "uploadDate": "2025-06-01",
    "contentUrl": "https://www.youtube.com/watch?v=7JQ-kfExIf8",
    "publisher": { "@id": "https://www.istanbultouristpass.com/#organization" },
    "embedUrl": "https://www.youtube.com/embed/7JQ-kfExIf8",
    "duration": "PT2M15S"
}
```

---

## 4. Attraction Listing

**URL:** `/whats-included`

### 4.1 ItemList (Attraction Carousel)

```
📊 VERİ KAYNAĞI: attractions tablosu (is_active = true, sıralı)
🔄 DÖNGÜ: Döngü ile TÜM aktif attraction'lar
⚠️ KURAL: Pagination varsa sadece sayfa 1'e ItemList bas. Sayfa 2+ için basma.
```

**Template:**

```json
{
    "@context": "https://schema.org",
    "@type": "ItemList",
    "name": "Istanbul Tourist Pass Attractions",
    "numberOfItems": {{ attractions.count }},
    "itemListElement": [
        // Her {{ attraction }} için:
        {
            "@type": "ListItem",
            "position": {{ loop.index }},
            "url": "{{ config.base_url }}/{{ attraction.slug }}",
            "name": "{{ attraction.name | safeJson }}",
            "image": "{{ attraction.featured_image_url }}"  // ← yoksa alanı çıkar
        }
    ]
}
```

**Gerçek Çıktı (ilk 3 gösterilmiştir — tüm 120+ basılmalı):**

```json
{
    "@context": "https://schema.org",
    "@type": "ItemList",
    "name": "Istanbul Tourist Pass Attractions",
    "numberOfItems": 120,
    "itemListElement": [
        {
            "@type": "ListItem",
            "position": 1,
            "url": "https://www.istanbultouristpass.com/hagia-sophia-guided-tour-with-skip-the-ticket-line-entry",
            "name": "Hagia Sophia Guided Tour with Skip-the-Ticket-Line Entry",
            "image": "https://www.istanbultouristpass.com/_astro/large_daf27debdbd2_Z1AeMYM.webp"
        },
        {
            "@type": "ListItem",
            "position": 2,
            "url": "https://www.istanbultouristpass.com/galata-tower-hosted-entry-with-audio-guide",
            "name": "Galata Tower Hosted Entry with Audio Guide",
            "image": "https://www.istanbultouristpass.com/_astro/large_4f498124bc47_Z9vnmS.webp"
        },
        {
            "@type": "ListItem",
            "position": 3,
            "url": "https://www.istanbultouristpass.com/topkapi-palace-guided-tour",
            "name": "Topkapi Palace Museum Guided Tour with Harem Including Entry Tickets",
            "image": "https://www.istanbultouristpass.com/_astro/large_681668205c12_ZcCxcf.webp"
        }
    ]
}
```

---

### 4.2 OfferCatalog (Kategori Bazlı)

```
📊 VERİ KAYNAĞI: attraction_categories tablosu + her birinin aktif attraction sayısı
🔄 DÖNGÜ: Kategoriler üzerinde iterasyon
⚠️ FALLBACK: Kategoride 0 attraction varsa o kategoriyi atla
💡 ÖNCELİK: 🟢 BONUS
```

**Gerçek Çıktı:**

```json
{
    "@context": "https://schema.org",
    "@type": "OfferCatalog",
    "name": "Istanbul Tourist Pass Attraction Categories",
    "itemListElement": [
        { "@type": "OfferCatalog", "name": "Museums", "numberOfItems": 35 },
        { "@type": "OfferCatalog", "name": "Mosques & Places of Worship", "numberOfItems": 9 },
        { "@type": "OfferCatalog", "name": "Historical Landmarks", "numberOfItems": 30 },
        { "@type": "OfferCatalog", "name": "Sightseeing & Bosphorus Cruise", "numberOfItems": 12 },
        { "@type": "OfferCatalog", "name": "Shows & Entertainment", "numberOfItems": 7 },
        { "@type": "OfferCatalog", "name": "Family Friendly", "numberOfItems": 20 }
    ]
}
```

---

## 5. Attraction Detay

**URL:** `/:attraction_slug`  
**En fazla schema içeren sayfa tipi: 7 adet**

### 5.1 TouristAttraction

```
📊 VERİ KAYNAĞI: attractions tablosu + locations + tags
⚠️ FALLBACK: geo yoksa → geo bloğunu çıkar. address yoksa → address bloğunu çıkar.
```

**Template:**

```json
{
    "@context": "https://schema.org",
    "@type": "TouristAttraction",
    "@id": "{{ config.base_url }}/{{ attraction.slug }}/#attraction",
    "name": "{{ attraction.name | safeJson }}",
    "description": "{{ attraction.long_description | safeJson }}",
    "image": {{ attraction.images | toJsonArray }},
    "url": "{{ config.base_url }}/{{ attraction.slug }}",
    "touristType": {{ attraction.tourist_types | toJsonArray }},  // ← yoksa çıkar
    "availableLanguage": {{ attraction.languages | toJsonArray }}, // ← yoksa çıkar
    "geo": { ... },          // ← lat/lng yoksa TÜMÜNÜ çıkar
    "address": { ... },      // ← street_address yoksa TÜMÜNÜ çıkar
    "containedInPlace": { "@type": "City", "name": "Istanbul", "sameAs": "https://en.wikipedia.org/wiki/Istanbul" }
}
```

**Gerçek Çıktı (Hagia Sophia):**

```json
{
    "@context": "https://schema.org",
    "@type": "TouristAttraction",
    "@id": "https://www.istanbultouristpass.com/hagia-sophia-guided-tour-with-skip-the-ticket-line-entry/#attraction",
    "name": "Hagia Sophia Guided Tour with Skip-the-Ticket-Line Entry",
    "description": "Explore the iconic Hagia Sophia mosque — a UNESCO World Heritage Site — with an expert English-speaking guide. Skip the long ticket lines and discover 1,500 years of Byzantine and Ottoman history.",
    "image": ["https://www.istanbultouristpass.com/_astro/large_daf27debdbd2_Z1AeMYM.webp"],
    "url": "https://www.istanbultouristpass.com/hagia-sophia-guided-tour-with-skip-the-ticket-line-entry",
    "isAccessibleForFree": false,
    "publicAccess": true,
    "touristType": ["Cultural Tourism", "Historical Tourism", "Religious Tourism"],
    "availableLanguage": ["English", "Turkish", "German", "Spanish"],
    "geo": {
        "@type": "GeoCoordinates",
        "latitude": 41.0086,
        "longitude": 28.9802
    },
    "address": {
        "@type": "PostalAddress",
        "streetAddress": "Sultan Ahmet, Ayasofya Meydanı No:1",
        "addressLocality": "Fatih",
        "addressRegion": "Istanbul",
        "addressCountry": "TR",
        "postalCode": "34122"
    },
    "containedInPlace": {
        "@type": "City",
        "name": "Istanbul",
        "sameAs": "https://en.wikipedia.org/wiki/Istanbul"
    }
}
```

---

### 5.2 Product + Offer

```
📊 VERİ KAYNAĞI: attractions tablosu (current_price, is_active)
⚠️ KRİTİK: price, sayfada render edilen fiyatla BİREBİR eşleşmeli. Farklı kaynak kullanımı Google'da "price mismatch" hatasına yol açar.
⚠️ FALLBACK: price null/0 → BU SCHEMA'YI HİÇ BASMA
⚠️ FALLBACK: reviewCount = 0 → aggregateRating bloğunu çıkar
```

**Template:**

```json
// ⚠️ KOŞUL: SADECE attraction.current_price > 0 ise bas
{
    "@context": "https://schema.org",
    "@type": "Product",
    "name": "{{ attraction.name | safeJson }}",
    "description": "{{ attraction.short_description | safeJson }}",
    "image": "{{ attraction.featured_image_url ?? config.og_image_url }}",
    "sku": "ITP-{{ attraction.sku ?? attraction.id }}",
    "brand": { "@id": "{{ config.base_url }}/#organization" },
    "category": "{{ attraction.category.name | safeJson }}",
    "offers": {
        "@type": "Offer",
        "price": "{{ attraction.current_price }}",
        "priceCurrency": "{{ config.currency }}",
        "availability": "{{ attraction.is_active ? 'https://schema.org/InStock' : 'https://schema.org/OutOfStock' }}",
        "priceValidUntil": "{{ currentYear }}-12-31",
        "url": "{{ config.base_url }}/{{ attraction.slug }}",
        "seller": { "@id": "{{ config.base_url }}/#organization" }
    },
    // ⚠️ SADECE reviewCount > 0 ise ekle:
    "aggregateRating": {
        "@type": "AggregateRating",
        "ratingValue": "{{ attraction.average_rating | round(1) }}",
        "reviewCount": "{{ attraction.review_count }}",
        "bestRating": "5"
    }
}
```

**Gerçek Çıktı (Hagia Sophia):**

```json
{
    "@context": "https://schema.org",
    "@type": "Product",
    "name": "Hagia Sophia Guided Tour with Skip-the-Ticket-Line Entry",
    "description": "Skip the long lines and explore Hagia Sophia with a licensed guide. Includes entry ticket and 60-90 minute guided tour.",
    "image": "https://www.istanbultouristpass.com/_astro/large_daf27debdbd2_Z1AeMYM.webp",
    "sku": "ITP-HSG-001",
    "brand": { "@id": "https://www.istanbultouristpass.com/#organization" },
    "category": "Museums",
    "offers": {
        "@type": "Offer",
        "price": "35",
        "priceCurrency": "EUR",
        "availability": "https://schema.org/InStock",
        "validFrom": "2025-01-01",
        "priceValidUntil": "2026-12-31",
        "url": "https://www.istanbultouristpass.com/hagia-sophia-guided-tour-with-skip-the-ticket-line-entry",
        "seller": { "@id": "https://www.istanbultouristpass.com/#organization" },
        "itemCondition": "https://schema.org/NewCondition",
        "hasMerchantReturnPolicy": {
            "@type": "MerchantReturnPolicy",
            "applicableCountry": "TR",
            "returnPolicyCategory": "https://schema.org/MerchantReturnFiniteReturnWindow",
            "merchantReturnDays": 30,
            "returnFees": "https://schema.org/FreeReturn"
        }
    },
    "aggregateRating": {
        "@type": "AggregateRating",
        "ratingValue": "4.8",
        "reviewCount": "350",
        "bestRating": "5"
    }
}
```

---

### 5.3 Event (Rehberli Turlar & Gösteriler)

```
⚠️ KOŞUL: SADECE is_guided_tour veya is_event flag'i true olanlarda bas.
   Müze giriş bileti gibi attraction'lar için Event schema BASMA.
⚠️ FALLBACK: schedule yoksa → eventSchedule bloğunu çıkar
⚠️ FALLBACK: koordinat yoksa → geo bloğunu çıkar
⚠️ FALLBACK: duration yoksa → duration alanını çıkar
```

**Gerçek Çıktı (Hagia Sophia Guided Tour):**

```json
{
    "@context": "https://schema.org",
    "@type": "Event",
    "name": "Hagia Sophia Guided Tour",
    "description": "Expert-led guided tour of Hagia Sophia with skip-the-line entry",
    "image": "https://www.istanbultouristpass.com/_astro/large_daf27debdbd2_Z1AeMYM.webp",
    "eventAttendanceMode": "https://schema.org/OfflineEventAttendanceMode",
    "eventStatus": "https://schema.org/EventScheduled",
    "eventSchedule": {
        "@type": "Schedule",
        "repeatFrequency": "P1D",
        "byDay": ["Monday","Tuesday","Wednesday","Thursday","Friday","Saturday","Sunday"],
        "startTime": "09:00:00",
        "endTime": "17:00:00",
        "scheduleTimezone": "Europe/Istanbul"
    },
    "location": {
        "@type": "Place",
        "name": "Hagia Sophia",
        "address": {
            "@type": "PostalAddress",
            "addressLocality": "Istanbul",
            "addressCountry": "TR"
        },
        "geo": {
            "@type": "GeoCoordinates",
            "latitude": 41.0086,
            "longitude": 28.9802
        }
    },
    "organizer": { "@id": "https://www.istanbultouristpass.com/#organization" },
    "offers": {
        "@type": "Offer",
        "price": "35",
        "priceCurrency": "EUR",
        "availability": "https://schema.org/InStock",
        "url": "https://www.istanbultouristpass.com/hagia-sophia-guided-tour-with-skip-the-ticket-line-entry"
    },
    "duration": "PT90M",
    "inLanguage": "en"
}
```

---

### 5.4 FAQPage (Per Attraction)

```
📊 VERİ KAYNAĞI: attraction_faqs tablosu (attraction_id ile ilişkili)
🔄 DÖNGÜ: O attraction'a ait tüm FAQ'lar üzerinde iterasyon
⚠️ FALLBACK: 0 FAQ → schema basma
⚠️ DİKKAT: Cevaplardaki HTML tag'leri temizlenmeli (strip_tags)
```

**Gerçek Çıktı (Hagia Sophia FAQ'ları):**

```json
{
    "@context": "https://schema.org",
    "@type": "FAQPage",
    "mainEntity": [
        {
            "@type": "Question",
            "name": "How long is the Hagia Sophia guided tour?",
            "acceptedAnswer": {
                "@type": "Answer",
                "text": "The guided tour lasts approximately 60-90 minutes, depending on the group size and guide's narrative."
            }
        },
        {
            "@type": "Question",
            "name": "Is Hagia Sophia included in Istanbul Tourist Pass?",
            "acceptedAnswer": {
                "@type": "Answer",
                "text": "Yes, the Hagia Sophia Guided Tour is included in the FAST Pass, DISCOVER Pass, and PRIME Pass."
            }
        },
        {
            "@type": "Question",
            "name": "What is the dress code for Hagia Sophia?",
            "acceptedAnswer": {
                "@type": "Answer",
                "text": "As an active mosque, visitors must dress modestly. Women should cover their hair, shoulders, and knees. Free scarves and coverings are available at the entrance."
            }
        }
    ]
}
```

---

### 5.5 Review + AggregateRating (Per Attraction)

```
📊 VERİ KAYNAĞI: reviews tablosu (attraction_id ile filtrelenen, is_approved = true)
🔄 DÖNGÜ: Max 5 approved review
⚠️ FALLBACK: 0 review → tüm bloğu çıkar
```

**Gerçek Çıktı:**

```json
{
    "@context": "https://schema.org",
    "@type": "Product",
    "name": "Hagia Sophia Guided Tour with Skip-the-Ticket-Line Entry",
    "review": [
        {
            "@type": "Review",
            "author": { "@type": "Person", "name": "Nurin" },
            "datePublished": "2025-11-10",
            "reviewRating": { "@type": "Rating", "ratingValue": "5", "bestRating": "5" },
            "reviewBody": "We had a tour at Hagia Sophia Mosque with Ilke Nur! She was very accommodating and friendly."
        },
        {
            "@type": "Review",
            "author": { "@type": "Person", "name": "Ruomei" },
            "datePublished": "2025-10-05",
            "reviewRating": { "@type": "Rating", "ratingValue": "5", "bestRating": "5" },
            "reviewBody": "Our tour guide was very knowledgeable about the Hagia Sofia. We really enjoyed his explanation."
        }
    ],
    "aggregateRating": {
        "@type": "AggregateRating",
        "ratingValue": "4.8",
        "reviewCount": "350",
        "bestRating": "5"
    }
}
```

---

## 6. Blog Listing

**URL:** `/blog`

### 6.1 CollectionPage + ItemList

```
📊 VERİ KAYNAĞI: articles tablosu (is_published = true, tarih sıralı)
🔄 DÖNGÜ: Yayınlanmış blog yazıları üzerinde iterasyon
⚠️ KURAL: Pagination varsa sadece sayfa 1'e ItemList bas
```

**Gerçek Çıktı (CollectionPage):**

```json
{
    "@context": "https://schema.org",
    "@type": "CollectionPage",
    "@id": "https://www.istanbultouristpass.com/blog/#webpage",
    "name": "Istanbul Travel Guide — Tips, Guides & Stories",
    "description": "Plan your trip like a local with our Istanbul travel guides and insider tips.",
    "url": "https://www.istanbultouristpass.com/blog",
    "isPartOf": { "@id": "https://www.istanbultouristpass.com/#website" },
    "inLanguage": "en"
}
```

**Gerçek Çıktı (ItemList):**

```json
{
    "@context": "https://schema.org",
    "@type": "ItemList",
    "name": "Istanbul Travel Guide Articles",
    "itemListElement": [
        {
            "@type": "ListItem",
            "position": 1,
            "url": "https://www.istanbultouristpass.com/blog/shopping-in-istanbul/best-shopping-malls-in-istanbul-for-tourists",
            "name": "Best Shopping Malls in Istanbul for Tourists"
        },
        {
            "@type": "ListItem",
            "position": 2,
            "url": "https://www.istanbultouristpass.com/blog/food-drink/turkish-wine-and-istanbul-wine-houses",
            "name": "From Vine to Bosphorus: Discovering Turkish Wine..."
        },
        {
            "@type": "ListItem",
            "position": 3,
            "url": "https://www.istanbultouristpass.com/blog/tips-guides/istanbul-travel-mistakes-to-avoid",
            "name": "Istanbul Travel Mistakes to Avoid"
        }
    ]
}
```

---

## 7. Blog Detay

**URL:** `/blog/:category_slug/:article_slug`

### 7.1 BlogPosting

```
📊 VERİ KAYNAĞI: articles + authors + categories tabloları
⚠️ KRİTİK: dateModified'ı her sayfa render'ında güncellemeyin — sadece gerçek içerik editinde.
⚠️ FALLBACK: author yoksa → config'deki default_author kullan
⚠️ FALLBACK: image yoksa → OG image kullan
⚠️ FALLBACK: wordCount yoksa → wordCount ve timeRequired alanlarını çıkar
⚠️ FALLBACK: tags yoksa → keywords alanını çıkar
```

**Template:**

```json
{
    "@context": "https://schema.org",
    "@type": "BlogPosting",
    "@id": "{{ config.base_url }}/blog/{{ article.category.slug }}/{{ article.slug }}/#article",
    "headline": "{{ article.title | safeJson | truncate(110) }}",
    "description": "{{ article.meta_description | safeJson }}",
    "datePublished": "{{ article.published_at | dateFormat('Y-m-d') }}",
    "dateModified": "...",   // ← sadece updated_at > published_at ise ekle, yoksa çıkar
    "wordCount": ...,        // ← yoksa çıkar
    "timeRequired": "PT{{ wordCount / 200 | round }}M",  // ← wordCount yoksa çıkar
    "inLanguage": "{{ currentLocale }}",
    "url": "{{ config.base_url }}/blog/{{ article.category.slug }}/{{ article.slug }}",
    "image": { "@type": "ImageObject", "url": "{{ article.featured_image_url ?? config.og_image_url }}", "width": 1200, "height": 630 },
    "author": {
        "@type": "Person",
        "@id": "{{ config.base_url }}/authors/{{ author.slug ?? 'editorial-team' }}/#person",
        "name": "{{ author.name ?? config.default_author | safeJson }}",
        "url": "{{ author.url ?? config.base_url + '/about-us' }}",
        "jobTitle": "...",       // ← yoksa çıkar
        "worksFor": { "@id": "{{ config.base_url }}/#organization" },
        "knowsAbout": [...]      // ← yoksa çıkar
    },
    "publisher": { "@id": "{{ config.base_url }}/#organization" },
    "articleSection": "{{ article.category.name | safeJson }}",
    "keywords": {{ article.tags | pluck('name') | toJsonArray }},  // ← yoksa çıkar
    "about": { "@type": "City", "name": "Istanbul", "sameAs": "https://en.wikipedia.org/wiki/Istanbul" }
}
```

**Gerçek Çıktı:**

```json
{
    "@context": "https://schema.org",
    "@type": "BlogPosting",
    "@id": "https://www.istanbultouristpass.com/blog/tips-guides/istanbul-travel-mistakes-to-avoid/#article",
    "headline": "Istanbul Travel Mistakes to Avoid",
    "description": "Common mistakes tourists make when visiting Istanbul and how to avoid them for a smoother, more enjoyable trip.",
    "datePublished": "2025-11-15",
    "dateModified": "2025-12-01",
    "wordCount": 2200,
    "timeRequired": "PT11M",
    "inLanguage": "en",
    "url": "https://www.istanbultouristpass.com/blog/tips-guides/istanbul-travel-mistakes-to-avoid",
    "image": {
        "@type": "ImageObject",
        "url": "https://cdn.istanbultouristpass.com/banner/doc/istanbul-layover-what-to-do.jpg",
        "width": 1200,
        "height": 630
    },
    "author": {
        "@type": "Person",
        "@id": "https://www.istanbultouristpass.com/authors/editorial-team/#person",
        "name": "ITP Editorial Team",
        "url": "https://www.istanbultouristpass.com/about-us",
        "jobTitle": "Senior Travel Content Creator",
        "worksFor": { "@id": "https://www.istanbultouristpass.com/#organization" },
        "knowsAbout": ["Istanbul Tourism", "Turkish Culture", "Travel Planning"]
    },
    "publisher": { "@id": "https://www.istanbultouristpass.com/#organization" },
    "mainEntityOfPage": {
        "@type": "WebPage",
        "@id": "https://www.istanbultouristpass.com/blog/tips-guides/istanbul-travel-mistakes-to-avoid/#webpage"
    },
    "articleSection": "Tips & Guides",
    "keywords": ["istanbul travel tips", "istanbul mistakes", "tourist guide istanbul"],
    "isPartOf": { "@id": "https://www.istanbultouristpass.com/#website" },
    "about": {
        "@type": "City",
        "name": "Istanbul",
        "sameAs": "https://en.wikipedia.org/wiki/Istanbul"
    }
}
```

---

## 8. Blog Kategori

**URL:** `/blog/:category_slug`

Blog Listing (Bölüm 6) ile aynı yapı — `CollectionPage` + `ItemList`. Tek fark: kategoriye göre filtrelenmiş articles ve farklı breadcrumb (Home → Blog → Tips & Guides).

---

## 9. FAQ Sayfası

**URL:** `/frequently-asked-questions`

```
📊 VERİ KAYNAĞI: faqs tablosu (is_published = true, sort_order sıralı)
🔄 DÖNGÜ: Tüm aktif FAQ'lar üzerinde iterasyon
⚠️ FALLBACK: 0 FAQ → schema basma
⚠️ DİKKAT: Cevaplardaki HTML temizlenmeli
```

Bölüm 5.4'teki `FAQPage` yapısının aynısı — veri kaynağı attraction değil global FAQ tablosudur.

---

## 10. Pass & Fiyatlar

**URL:** `/istanbul-pass`

### 10.1 Product (Her Pass İçin — Döngü)

```
📊 VERİ KAYNAĞI: passes tablosu + pass_variants tablosu
🔄 DÖNGÜ: Her aktif pass → ayrı bir Product schema (ayrı <script> bloğu)
⚠️ KRİTİK: Variant sayısına göre Offer vs AggregateOffer seçimi:
   - 1 variant → Offer (tek fiyat)
   - 2+ variant → AggregateOffer (lowPrice / highPrice)
⚠️ FALLBACK: Aktif variant yoksa → o pass'ın schema'sını basma
```

**Gerçek Çıktı (PRIME Pass — 3 pass'tan biri):**

```json
{
    "@context": "https://schema.org",
    "@type": "Product",
    "name": "Istanbul PRIME Pass®",
    "description": "Ultimate Istanbul Experience — 7 landmarks, 120+ top attractions, premium activities & experiences. Priority hosted access + dedicated support line.",
    "sku": "ITP-PRIME",
    "brand": { "@id": "https://www.istanbultouristpass.com/#organization" },
    "image": "https://www.istanbultouristpass.com/images/prime-pass.webp",
    "category": "Sightseeing Pass",
    "offers": {
        "@type": "AggregateOffer",
        "lowPrice": "139",
        "highPrice": "289",
        "priceCurrency": "EUR",
        "offerCount": "7",
        "availability": "https://schema.org/InStock",
        "url": "https://www.istanbultouristpass.com/istanbul-pass?pass=prime",
        "priceValidUntil": "2026-12-31",
        "seller": { "@id": "https://www.istanbultouristpass.com/#organization" }
    },
    "aggregateRating": {
        "@type": "AggregateRating",
        "ratingValue": "4.8",
        "reviewCount": "5500",
        "bestRating": "5"
    }
}
```

> **Not:** FAST Pass ve DISCOVER Pass için de aynı yapıda ayrı Product schema'ları basılmalıdır.

---

## 11. Reviews Sayfası

**URL:** `/reviews`

```
📊 VERİ KAYNAĞI: reviews tablosu (is_approved = true, max 10) + video_testimonials tablosu
🔄 DÖNGÜ: Review'lar ve video'lar üzerinde ayrı ayrı iterasyon
⚠️ FALLBACK: 0 review → schema basma
```

Anasayfadaki Review schema (Bölüm 3.2) ile aynı yapı. Farklar: review limiti 10'a çıkar, video testimonials (Bölüm 3.6) için ek VideoObject blokları eklenir.

---

## 12. About Sayfası

**URL:** `/about-us`

```
📊 VERİ KAYNAĞI: Sayfa meta verileri + Organization @id referansı
```

**Template:**

```json
{
    "@context": "https://schema.org",
    "@type": "AboutPage",
    "@id": "{{ config.base_url }}/about-us/#webpage",
    "name": "{{ page.meta_title | safeJson }}",
    "description": "{{ page.meta_description | safeJson }}",
    "url": "{{ config.base_url }}/about-us",
    "isPartOf": { "@id": "{{ config.base_url }}/#website" },
    "inLanguage": "{{ currentLocale }}",
    "mainEntity": { "@id": "{{ config.base_url }}/#organization" }
}
```

**Gerçek Çıktı:**

```json
{
    "@context": "https://schema.org",
    "@type": "AboutPage",
    "@id": "https://www.istanbultouristpass.com/about-us/#webpage",
    "name": "About Istanbul Tourist Pass — Istanbul Experts Since 1995",
    "description": "Learn about Istanbul Tourist Pass — Istanbul's original sightseeing pass trusted by 500K+ travelers from 150+ countries since 2015.",
    "url": "https://www.istanbultouristpass.com/about-us",
    "isPartOf": { "@id": "https://www.istanbultouristpass.com/#website" },
    "inLanguage": "en",
    "mainEntity": { "@id": "https://www.istanbultouristpass.com/#organization" }
}
```

---

## 13. Contact Sayfası

**URL:** `/contact`

```
📊 VERİ KAYNAĞI: Sayfa meta verileri + Organization @id referansı
💡 NOT: ContactPoint detayları zaten Organization schema'sında (Bölüm 2.1) tanımlıdır. Google @id referansı ile otomatik birleştirir.
```

**Gerçek Çıktı:**

```json
{
    "@context": "https://schema.org",
    "@type": "ContactPage",
    "@id": "https://www.istanbultouristpass.com/contact/#webpage",
    "name": "Contact Istanbul Tourist Pass — Get Help",
    "description": "Get in touch with our Istanbul expert team via phone, WhatsApp, email, or live chat. Available 7 days a week.",
    "url": "https://www.istanbultouristpass.com/contact",
    "isPartOf": { "@id": "https://www.istanbultouristpass.com/#website" },
    "inLanguage": "en",
    "mainEntity": { "@id": "https://www.istanbultouristpass.com/#organization" }
}
```

---

## 14. How It Works Sayfası

**URL:** `/how-it-works`

```
📊 VERİ KAYNAĞI: Sayfa meta verileri + HowTo (Bölüm 3.3) ayrıca basılır
💡 NOT: Bu sayfada 2 schema basılır — WebPage + HowTo
```

**Gerçek Çıktı (WebPage):**

```json
{
    "@context": "https://schema.org",
    "@type": "WebPage",
    "@id": "https://www.istanbultouristpass.com/how-it-works/#webpage",
    "name": "How Istanbul Tourist Pass Works — Simple 3-Step Process",
    "description": "Get your pass, download the app, and start exploring Istanbul's 120+ attractions with skip-the-line access.",
    "url": "https://www.istanbultouristpass.com/how-it-works",
    "isPartOf": { "@id": "https://www.istanbultouristpass.com/#website" },
    "inLanguage": "en"
}
```

> HowTo schema'sı (adımlar) → Bölüm 3.3'teki çıktının aynısı bu sayfada da basılır.

---

## 15. Compare Passes Sayfası

**URL:** `/compare-passes`

```
📊 VERİ KAYNAĞI: Sayfa meta verileri + pass Product @id referansları
```

**Gerçek Çıktı:**

```json
{
    "@context": "https://schema.org",
    "@type": "WebPage",
    "@id": "https://www.istanbultouristpass.com/compare-passes/#webpage",
    "name": "Compare Istanbul Tourist Passes — FAST vs DISCOVER vs PRIME",
    "description": "Compare all Istanbul Tourist Pass options side by side. Find the best pass for your trip based on duration, attractions, and budget.",
    "url": "https://www.istanbultouristpass.com/compare-passes",
    "isPartOf": { "@id": "https://www.istanbultouristpass.com/#website" },
    "inLanguage": "en"
}
```

---

## 16. Download App Sayfası

**URL:** `/download-app`

```
📊 VERİ KAYNAĞI: Sayfa meta verileri + SoftwareApplication (Bölüm 3.4)
💡 NOT: Bu sayfada 2 schema basılır — WebPage + SoftwareApplication
⚠️ FALLBACK: App store URL'leri yoksa SoftwareApplication basma
```

**Gerçek Çıktı (WebPage):**

```json
{
    "@context": "https://schema.org",
    "@type": "WebPage",
    "@id": "https://www.istanbultouristpass.com/download-app/#webpage",
    "name": "Download the Istanbul Tourist Pass® App",
    "description": "Download the Istanbul Tourist Pass app for iOS and Android to manage your pass, plan your itinerary, and explore Istanbul with audio guides in 25 languages.",
    "url": "https://www.istanbultouristpass.com/download-app",
    "isPartOf": { "@id": "https://www.istanbultouristpass.com/#website" },
    "inLanguage": "en"
}
```

> SoftwareApplication schema'sı → Bölüm 3.4'teki çıktının aynısı bu sayfada da basılır.

---

## 17. Plan & Save Sayfası

**URL:** `/plan-and-save`

```
📊 VERİ KAYNAĞI: Sayfa meta verileri
💡 NOT: Hesaplama aracı varsa opsiyonel WebApplication schema eklenebilir
```

**Gerçek Çıktı:**

```json
{
    "@context": "https://schema.org",
    "@type": "WebPage",
    "@id": "https://www.istanbultouristpass.com/plan-and-save/#webpage",
    "name": "Plan & Save Your Istanbul Trip | Istanbul Tourist Pass®",
    "description": "Plan your Istanbul itinerary and see how much you save with Istanbul Tourist Pass compared to buying individual tickets.",
    "url": "https://www.istanbultouristpass.com/plan-and-save",
    "isPartOf": { "@id": "https://www.istanbultouristpass.com/#website" },
    "inLanguage": "en"
}
```

---

## 18. What You Get Sayfası

**URL:** `/what-you-get`

```
📊 VERİ KAYNAĞI: Sayfa meta verileri + ana ürün Product @id referansı
```

**Gerçek Çıktı:**

```json
{
    "@context": "https://schema.org",
    "@type": "WebPage",
    "@id": "https://www.istanbultouristpass.com/what-you-get/#webpage",
    "name": "What You Get with Istanbul Tourist Pass® — All Included Benefits",
    "description": "See everything included in your Istanbul Tourist Pass: skip-the-line access, guided tours, Bosphorus cruises, audio guides in 25 languages, mobile app, and expert support.",
    "url": "https://www.istanbultouristpass.com/what-you-get",
    "isPartOf": { "@id": "https://www.istanbultouristpass.com/#website" },
    "inLanguage": "en",
    "mainEntity": { "@id": "https://www.istanbultouristpass.com/#product" }
}
```

---

## 19. Free Digital Guidebook Sayfası

**URL:** `/free-digital-istanbul-guidebook`

```
📊 VERİ KAYNAĞI: Sayfa meta verileri + DigitalDocument bilgisi
💡 NOT: Lead generation sayfası — PDF indirme formu
```

**Gerçek Çıktı:**

```json
{
    "@context": "https://schema.org",
    "@type": "WebPage",
    "@id": "https://www.istanbultouristpass.com/free-digital-istanbul-guidebook/#webpage",
    "name": "Free Digital Istanbul Guidebook | Istanbul Tourist Pass®",
    "description": "Download our free digital Istanbul guidebook with insider tips, must-see attractions, and practical travel advice.",
    "url": "https://www.istanbultouristpass.com/free-digital-istanbul-guidebook",
    "isPartOf": { "@id": "https://www.istanbultouristpass.com/#website" },
    "inLanguage": "en",
    "mainEntity": {
        "@type": "DigitalDocument",
        "name": "Free Istanbul Digital Guidebook",
        "description": "Comprehensive digital guidebook covering Istanbul's top attractions, neighborhoods, food scene, and practical travel tips.",
        "author": { "@id": "https://www.istanbultouristpass.com/#organization" },
        "encodingFormat": "application/pdf",
        "isAccessibleForFree": true,
        "offers": {
            "@type": "Offer",
            "price": "0",
            "priceCurrency": "EUR",
            "availability": "https://schema.org/InStock"
        }
    }
}
```

---

## 20. Group Request Sayfası

**URL:** `/group-request`

```
📊 VERİ KAYNAĞI: Sayfa meta verileri + Service tanımı
```

**Gerçek Çıktı:**

```json
{
    "@context": "https://schema.org",
    "@type": "WebPage",
    "@id": "https://www.istanbultouristpass.com/group-request/#webpage",
    "name": "Group Requests | Istanbul Tourist Pass®",
    "description": "Plan a group visit to Istanbul with special group pricing and dedicated support. Perfect for school groups, corporate teams, and tour operators.",
    "url": "https://www.istanbultouristpass.com/group-request",
    "isPartOf": { "@id": "https://www.istanbultouristpass.com/#website" },
    "inLanguage": "en",
    "mainEntity": {
        "@type": "Service",
        "name": "Istanbul Tourist Pass — Group Booking Service",
        "description": "Special group pricing and dedicated coordination for groups of 10+ travelers visiting Istanbul.",
        "provider": { "@id": "https://www.istanbultouristpass.com/#organization" },
        "serviceType": "Group Travel Planning",
        "areaServed": { "@type": "City", "name": "Istanbul" }
    }
}
```

---

## 21. Special Offer & Service Sayfaları

**URL'ler:** `/tourist-sim-card-...`, `/meet-and-greet-...`, `/special-discounts`, `/health-benefits`, `/exclusive-travel-services`

```
📊 VERİ KAYNAĞI: Sayfa meta verileri + Service/Offer bilgileri
🔄 HER SERVİS SAYFASI İÇİN: Ayrı ayrı WebPage + Service schema
⚠️ FALLBACK: price yoksa → offers bloğunu çıkar
```

**Template:**

```json
{
    "@context": "https://schema.org",
    "@type": "WebPage",
    "@id": "{{ config.base_url }}/{{ page.slug }}/#webpage",
    "name": "{{ page.meta_title | safeJson }}",
    "description": "{{ page.meta_description | safeJson }}",
    "url": "{{ config.base_url }}/{{ page.slug }}",
    "isPartOf": { "@id": "{{ config.base_url }}/#website" },
    "inLanguage": "{{ currentLocale }}",
    "mainEntity": {
        "@type": "Service",
        "name": "{{ service.name | safeJson }}",
        "description": "{{ service.description | safeJson }}",
        "provider": { "@id": "{{ config.base_url }}/#organization" },
        "serviceType": "{{ service.type }}",
        "areaServed": { "@type": "{{ service.area_type }}", "name": "{{ service.area_name }}" },
        // ⚠️ SADECE price varsa ekle:
        "offers": {
            "@type": "Offer",
            "priceCurrency": "{{ config.currency }}",
            "price": "{{ service.price }}",
            "availability": "https://schema.org/InStock"
        }
    }
}
```

**Gerçek Çıktı (Tourist eSIM sayfası):**

```json
{
    "@context": "https://schema.org",
    "@type": "WebPage",
    "@id": "https://www.istanbultouristpass.com/tourist-sim-card-internet-access-on-your-device/#webpage",
    "name": "Tourist eSIM in Turkey | Istanbul Tourist Pass®",
    "description": "Stay connected during your Istanbul trip with a tourist eSIM. Easy setup, instant activation, and unlimited data coverage across Turkey.",
    "url": "https://www.istanbultouristpass.com/tourist-sim-card-internet-access-on-your-device",
    "isPartOf": { "@id": "https://www.istanbultouristpass.com/#website" },
    "inLanguage": "en",
    "mainEntity": {
        "@type": "Service",
        "name": "Tourist eSIM in Turkey",
        "description": "Stay connected during your Istanbul trip with a tourist eSIM. Easy setup, instant activation, and unlimited data coverage across Turkey.",
        "provider": { "@id": "https://www.istanbultouristpass.com/#organization" },
        "serviceType": "Telecommunications Service",
        "areaServed": { "@type": "Country", "name": "Turkey" },
        "offers": {
            "@type": "Offer",
            "priceCurrency": "EUR",
            "availability": "https://schema.org/InStock"
        }
    }
}
```

**Gerçek Çıktı (Meet & Greet sayfası):**

```json
{
    "@context": "https://schema.org",
    "@type": "WebPage",
    "@id": "https://www.istanbultouristpass.com/meet-and-greet-service-at-istanbul-airport/#webpage",
    "name": "Meet and Greet Service at Istanbul Airport | Istanbul Tourist Pass®",
    "description": "VIP airport welcome service with a personal greeter at Istanbul Airport. Assistance with customs, baggage, and transfer coordination.",
    "url": "https://www.istanbultouristpass.com/meet-and-greet-service-at-istanbul-airport",
    "isPartOf": { "@id": "https://www.istanbultouristpass.com/#website" },
    "inLanguage": "en",
    "mainEntity": {
        "@type": "Service",
        "name": "Meet and Greet Service at Istanbul Airport",
        "description": "VIP airport welcome service with a personal greeter at Istanbul Airport.",
        "provider": { "@id": "https://www.istanbultouristpass.com/#organization" },
        "serviceType": "Airport Concierge Service",
        "areaServed": { "@type": "Airport", "name": "Istanbul Airport", "iataCode": "IST" }
    }
}
```

**Gerçek Çıktı (Special Discounts sayfası):**

```json
{
    "@context": "https://schema.org",
    "@type": "WebPage",
    "@id": "https://www.istanbultouristpass.com/special-discounts/#webpage",
    "name": "Special Discounts & Deals | Istanbul Tourist Pass®",
    "description": "Exclusive discounts and special offers for Istanbul Tourist Pass holders at partner restaurants, shops, and services.",
    "url": "https://www.istanbultouristpass.com/special-discounts",
    "isPartOf": { "@id": "https://www.istanbultouristpass.com/#website" },
    "inLanguage": "en",
    "mainEntity": {
        "@type": "OfferCatalog",
        "name": "Istanbul Tourist Pass — Special Discounts & Deals",
        "description": "Exclusive discounts for Istanbul Tourist Pass holders.",
        "itemListElement": [
            {
                "@type": "Offer",
                "name": "30% Off Health & Wellness Services",
                "description": "Exclusive health benefit discounts at top Istanbul hospitals and wellness centers.",
                "priceCurrency": "EUR"
            }
        ]
    }
}
```

---

## 22. Partner & Affiliate Sayfaları

**URL'ler:** `/become-affiliate-partner`, `/reseller`, `/creators-influencers`

```
📊 VERİ KAYNAĞI: Sayfa meta verileri
💡 NOT: Basit WebPage yeterli — ek mainEntity gerekmez
```

**Gerçek Çıktı (Affiliate Partner sayfası):**

```json
{
    "@context": "https://schema.org",
    "@type": "WebPage",
    "@id": "https://www.istanbultouristpass.com/become-affiliate-partner/#webpage",
    "name": "Become an Affiliate Partner — Istanbul Tourist Pass",
    "description": "Join Istanbul Tourist Pass affiliate program. Earn commissions by promoting Istanbul's #1 sightseeing pass to your audience.",
    "url": "https://www.istanbultouristpass.com/become-affiliate-partner",
    "isPartOf": { "@id": "https://www.istanbultouristpass.com/#website" },
    "inLanguage": "en"
}
```

---

## 23. Legal Sayfaları

**URL'ler:** `/privacy-policy`, `/terms-of-use`, `/cookie-policy`

```
📊 VERİ KAYNAĞI: Sayfa meta verileri
💡 NOT: lastReviewed alanı eklenmeli — son güncelleme tarihi
```

**Gerçek Çıktı (Privacy Policy):**

```json
{
    "@context": "https://schema.org",
    "@type": "WebPage",
    "@id": "https://www.istanbultouristpass.com/privacy-policy/#webpage",
    "name": "Privacy Policy | Istanbul Tourist Pass®",
    "description": "Istanbul Tourist Pass privacy policy. Learn how we collect, use, and protect your personal data.",
    "url": "https://www.istanbultouristpass.com/privacy-policy",
    "isPartOf": { "@id": "https://www.istanbultouristpass.com/#website" },
    "inLanguage": "en",
    "lastReviewed": "2025-12-01"
}
```

---

## 24. Implementasyon Checklist

### 24.1 Test Matrisi

| # | Test | Nasıl | Geçme Kriteri |
|---|---|---|---|
| 1 | JSON syntax | JSON validator/parser | Hata yok |
| 2 | Google Rich Results | https://search.google.com/test/rich-results | Tüm schema'lar valid |
| 3 | Fiyat tutarlılığı | Schema price vs. sayfadaki price | Birebir eşleşme |
| 4 | Boş veri senaryosu | DB'den image/review/faq sil, sayfayı test et | JSON bozulmamış, blok basılmamış |
| 5 | Multi-language | `/tr/`, `/de/` sayfaları | `inLanguage` doğru, içerik o dilde |
| 6 | Draft sayfa | Unpublished attraction/blog | Schema hiç basılmamış |
| 7 | Yeni kayıt | Yeni attraction ekle → detay sayfası | Schema'lar otomatik oluşmuş |
| 8 | XSS | `name` alanına `<script>alert(1)</script>` yaz | Tag temizlenmiş |
| 9 | Null price | Attraction fiyatını null yap | Product schema basılmamış |
| 10 | 0 review | Attraction'ın tüm review'larını sil | aggregateRating bloğu yok |

### 24.2 Yaygın Hatalar

| Hata | Neden | Çözüm |
|---|---|---|
| `price mismatch` | Schema fiyatı ≠ sayfadaki fiyat | Aynı veri kaynağından çek |
| `Missing field "image"` | null/empty gelmiş | Fallback: OG image URL |
| `Invalid date format` | Boş string basılmış | Tarih yoksa alanı çıkar |
| `Duplicate schema` | Aynı Product 2 kez basılmış | @id kontrolü + tek kaynak |
| `JSON parse error` | Description'da tırnak/newline var | safeJson helper kullan |
| `priceValidUntil expired` | Geçen yılın tarihi kalmış | currentYear dinamik hesapla |
| `reviewCount: 0 + ratingValue` | 0 review'la rating basmak | count > 0 koşulu ekle |

### 24.3 Sprint Planı

| Sprint | Süre | Kapsam | Schema |
|---|---|---|---|
| **1** | 1-2 hafta | Global bileşenler + Anasayfa + Attraction Detay + Blog Detay + FAQ + Pass & Fiyatlar + Reviews | ~35 |
| **2** | 1-2 hafta | Listing'ler + ItemList'ler + About/Contact/How-It-Works + Download App + Blog Kategori | ~18 |
| **3** | 1 hafta | Service sayfaları + Partner + Legal + Guidebook + OfferCatalog + VideoObject'ler | ~9 |
| **TOPLAM** | | **22 sayfa tipi** | **~62** |

### 24.4 Doğrulama Araçları

| Araç | URL | Ne İçin |
|---|---|---|
| Google Rich Results Test | https://search.google.com/test/rich-results | Doğrulama + preview |
| Schema.org Validator | https://validator.schema.org | Syntax |
| Google Search Console | https://search.google.com/search-console | Canlı performans |
| JSON-LD Playground | https://json-ld.org/playground | Linked data |

---

> **Toplam:** 22 sayfa tipi, ~62 schema, 3-katmanlı bileşen mimarisi  
> **Her schema için:** Dinamik template + gerçek çıktı örneği + fallback kuralları + veri kaynağı notu  
> **Tüm builder'lar:** Organization, WebSite, BreadcrumbList, hreflang, Product, AggregateOffer, Review, HowTo, ItemList, CollectionPage, TouristAttraction, Offer, Event, FAQPage, BlogPosting, VideoObject, SoftwareApplication, ContactPage, SiteNavigationElement, OfferCatalog, genericPage
