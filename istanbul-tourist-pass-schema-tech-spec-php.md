# Istanbul Tourist Pass® — Schema Markup Tech Spec

> **Versiyon:** 3.0 (PHP Developer Edition)  
> **Tarih:** 30 Mart 2026  
> **Kapsam:** v26.istanbultouristpass.com  
> **Format:** JSON-LD via `<script type="application/ld+json">`  
> **Hedef Kitle:** PHP/Laravel Backend & Frontend Developer  
> **Syntax:** Laravel Blade + PHP — kendi framework'ünüze uyarlayın

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
12. [Diğer Sayfa Tipleri (About, Contact, How It Works, vs.)](#12-diğer-sayfa-tipleri)
13. [Implementasyon Checklist](#13-implementasyon-checklist)

---

## 1. Mimari Kararlar & Kurallar

### 1.1 Bileşen Mimarisi

Schema'lar **3 katmanda** üretilir. Her katman ayrı bir PHP sınıfı veya helper olmalıdır:

```
┌─────────────────────────────────────────────────┐
│  KATMAN 1: Global Schemas (her sayfada)         │
│  → SchemaService::globals($page)                │
│  → Organization, WebSite, BreadcrumbList        │
├─────────────────────────────────────────────────┤
│  KATMAN 2: Sayfa Tipi Schemas                   │
│  → SchemaService::forPage($pageType, $data)     │
│  → Product, FAQPage, BlogPosting, Event vs.     │
├─────────────────────────────────────────────────┤
│  KATMAN 3: Widget/Section Schemas (opsiyonel)   │
│  → SchemaService::reviews($reviews)             │
│  → SchemaService::video($video)                 │
│  → Sayfa içi bileşenlere bağlı                  │
└─────────────────────────────────────────────────┘
```

### 1.2 Dosya Yapısı (Laravel Önerisi)

```
app/
├── Services/
│   └── SchemaService.php           # Ana servis sınıfı
├── Helpers/
│   └── SchemaHelpers.php           # safe_json, iso_duration, vb.
├── Config/
│   └── schema.php                  # Sabit @id URI'ler, hreflang map
resources/
├── views/
│   ├── layouts/
│   │   └── app.blade.php           # <head> içinde schema render
│   └── components/
│       └── schemas/
│           ├── globals.blade.php   # Katman 1
│           ├── breadcrumb.blade.php
│           ├── attraction.blade.php
│           ├── blog-posting.blade.php
│           ├── faq-page.blade.php
│           ├── product.blade.php
│           └── review.blade.php
```

### 1.3 SchemaService Ana Sınıf

```php
<?php

namespace App\Services;

use App\Helpers\SchemaHelpers;

class SchemaService
{
    /**
     * Verilen schema array'ini <script type="application/ld+json"> olarak render eder.
     * Null/boş schema'ları sessizce atlar.
     */
    public static function render(?array $schema): string
    {
        if (empty($schema)) {
            return '';
        }

        $json = json_encode($schema, JSON_UNESCAPED_SLASHES | JSON_UNESCAPED_UNICODE | JSON_PRETTY_PRINT);

        if ($json === false) {
            // JSON encode hatası — log'la ama sayfayı bozma
            \Log::warning('Schema JSON encode failed', ['schema' => $schema]);
            return '';
        }

        return '<script type="application/ld+json">' . $json . '</script>';
    }

    /**
     * Birden fazla schema'yı tek seferde render eder.
     * Null değerler otomatik filtrelenir.
     */
    public static function renderAll(array $schemas): string
    {
        return collect($schemas)
            ->filter()  // null/empty olanları çıkar
            ->map(fn($s) => self::render($s))
            ->implode("\n");
    }
}
```

### 1.4 SchemaHelpers

```php
<?php

namespace App\Helpers;

class SchemaHelpers
{
    /**
     * JSON-LD için güvenli string üretir.
     * HTML tag'leri temizler, JSON-kıran karakterleri escape eder.
     */
    public static function safeJson(?string $value, int $maxLength = 5000): string
    {
        if (empty($value)) {
            return '';
        }

        $value = strip_tags($value);
        $value = html_entity_decode($value, ENT_QUOTES, 'UTF-8');
        $value = preg_replace('/\s+/', ' ', $value); // çoklu boşlukları tek boşluğa
        $value = trim($value);

        return mb_substr($value, 0, $maxLength);
    }

    /**
     * Saniyeyi ISO 8601 duration formatına çevirir.
     * 120 → "PT2M0S", 5400 → "PT1H30M0S"
     */
    public static function isoDuration(?int $seconds): ?string
    {
        if (!$seconds || $seconds <= 0) {
            return null;
        }

        $hours = intdiv($seconds, 3600);
        $minutes = intdiv($seconds % 3600, 60);
        $secs = $seconds % 60;

        if ($hours > 0) {
            return "PT{$hours}H{$minutes}M{$secs}S";
        }
        return "PT{$minutes}M{$secs}S";
    }

    /**
     * Okuma süresi hesaplar (dakika).
     * Ortalama 200 kelime/dk baz alınır.
     */
    public static function readingTime(?int $wordCount): ?int
    {
        if (!$wordCount || $wordCount <= 0) {
            return null;
        }
        return max(1, (int) round($wordCount / 200));
    }
}
```

### 1.5 Config Dosyası (`config/schema.php`)

```php
<?php

return [
    'base_url'          => env('APP_URL', 'https://www.istanbultouristpass.com'),
    'site_name'         => 'Istanbul Tourist Pass',
    'product_name'      => 'Istanbul Tourist Pass®',
    'currency'          => 'EUR',
    'contact_email'     => 'contact@istanbultouristpass.com',
    'contact_phone'     => '+908503048726',
    'whatsapp_url'      => 'https://wa.me/905303852026',
    'logo_url'          => 'https://www.istanbultouristpass.com/images/itp-logo.svg',
    'og_image_url'      => 'https://www.istanbultouristpass.com/images/itp-og.jpg',
    'founding_date'     => '2015',
    'default_author'    => 'ITP Editorial Team',
    'return_days'       => 30,

    'social_links' => [
        'https://www.facebook.com/istanbultouristpass',
        'https://www.instagram.com/istanbultouristpass',
        'https://www.youtube.com/@istanbultouristpass',
        'https://www.tiktok.com/@istanbultouristpass',
        'https://www.tripadvisor.com/AttractionProductReview-g293974-d12464056-Istanbul_Tourist_Pass',
    ],

    'supported_languages' => ['English','Turkish','German','Russian','French','Spanish','Italian','Arabic'],

    'ios_app_url'     => 'https://apps.apple.com/tr/app/istanbul-tourist-pass/id6535683117',
    'android_app_url' => 'https://play.google.com/store/apps/details?id=com.istanbultouristpass.app',

    'hreflang_map' => [
        'en' => ['hreflang' => 'en',      'prefix' => ''],
        'es' => ['hreflang' => 'es',      'prefix' => '/es'],
        'it' => ['hreflang' => 'it',      'prefix' => '/it'],
        'fr' => ['hreflang' => 'fr',      'prefix' => '/fr'],
        'de' => ['hreflang' => 'de',      'prefix' => '/de'],
        'ru' => ['hreflang' => 'ru',      'prefix' => '/ru'],
        'tr' => ['hreflang' => 'tr',      'prefix' => '/tr'],
        'ar' => ['hreflang' => 'ar',      'prefix' => '/ar'],
        'bg' => ['hreflang' => 'bg',      'prefix' => '/bg'],
        'cn' => ['hreflang' => 'zh-Hans', 'prefix' => '/cn'],
        'gr' => ['hreflang' => 'el',      'prefix' => '/gr'],
        'hr' => ['hreflang' => 'hr',      'prefix' => '/hr'],
        'hu' => ['hreflang' => 'hu',      'prefix' => '/hu'],
        'id' => ['hreflang' => 'id',      'prefix' => '/id'],
        'il' => ['hreflang' => 'he',      'prefix' => '/il'],
        'in' => ['hreflang' => 'hi',      'prefix' => '/in'],
        'ir' => ['hreflang' => 'fa',      'prefix' => '/ir'],
        'jp' => ['hreflang' => 'ja',      'prefix' => '/jp'],
        'kr' => ['hreflang' => 'ko',      'prefix' => '/kr'],
        'mk' => ['hreflang' => 'mk',      'prefix' => '/mk'],
        'pk' => ['hreflang' => 'ur',      'prefix' => '/pk'],
        'pl' => ['hreflang' => 'pl',      'prefix' => '/pl'],
        'pt' => ['hreflang' => 'pt',      'prefix' => '/pt'],
        'ro' => ['hreflang' => 'ro',      'prefix' => '/ro'],
        'tw' => ['hreflang' => 'zh-Hant', 'prefix' => '/tw'],
    ],
];
```

### 1.6 Fallback Kuralları (Genel)

| Durum | Fallback | Kod Örneği |
|---|---|---|
| `image` alanı boş | Site logosunu bas | `$attraction->featured_image_url ?? config('schema.og_image_url')` |
| `description` alanı boş | `name` kullan veya alanı **çıkar** | `if (!empty($desc)) { $schema['description'] = $desc; }` |
| `price` null veya 0 | Product schema'sını **hiç basma** | `if ($price > 0) { ... }` |
| `reviewCount` = 0 | `aggregateRating` bloğunu **çıkar** | `if ($reviewCount > 0) { ... }` |
| `author` bilgisi yok | Default author kullan | `$author->name ?? config('schema.default_author')` |
| `datePublished` yok | Alanı tamamen çıkar | `if ($date) { $schema['datePublished'] = $date; }` |
| `geo` koordinatları yok | `geo` bloğunu çıkar | `if ($lat && $lng) { ... }` |
| Sayfa draft/unpublished | Schema **hiç basılmamalı** | Controller'da kontrol et |

> **Altın Kural:** Eksik veriyle bozuk JSON basmaktansa, o schema bloğunu hiç basmamak daha iyidir. Google bozuk JSON-LD'yi penalize eder.

### 1.7 Layout'ta Kullanım (`app.blade.php`)

```html
<head>
    <meta charset="utf-8">
    <title>{{ $metaTitle }}</title>
    
    {{-- hreflang --}}
    @include('components.schemas.hreflang', ['slug' => $page->slug ?? ''])
    
    {{-- Schema markup --}}
    {!! \App\Services\SchemaService::renderAll($schemas ?? []) !!}
</head>
```

**Controller'da:**

```php
public function show(Attraction $attraction)
{
    $schemas = array_filter([
        // Katman 1: Global
        SchemaService::organization(),      // sadece anasayfada tam, diğerlerinde null döner
        SchemaService::website(),           // sadece anasayfada
        SchemaService::breadcrumb($this->getBreadcrumbs($attraction)),

        // Katman 2: Sayfa tipi
        SchemaService::touristAttraction($attraction),
        SchemaService::productOffer($attraction),
        $attraction->is_guided_tour ? SchemaService::event($attraction) : null,
        $attraction->faqs->count() > 0 ? SchemaService::faqPage($attraction->faqs) : null,

        // Katman 3: Widget
        $attraction->approved_reviews_count > 0 ? SchemaService::reviews($attraction) : null,
        $attraction->video_url ? SchemaService::video($attraction) : null,
    ]);

    return view('attractions.show', compact('attraction', 'schemas'));
}
```

---

## 2. Global Bileşenler (Her Sayfada)

### 2.1 Organization — Sadece Anasayfada Tam Tanım

**PHP Builder:**

```php
public static function organization(): ?array
{
    // Sadece anasayfada tam tanım yapılır
    // Diğer sayfalar @id referansı ile erişir (otomatik — Google birleştirir)
    if (!request()->is('/')) {
        return null;
    }

    $cfg = config('schema');

    return [
        '@context'     => 'https://schema.org',
        '@type'        => 'Organization',
        '@id'          => $cfg['base_url'] . '/#organization',
        'name'         => $cfg['site_name'],
        'alternateName'=> ['ITP', 'Istanbul Tourist Pass®'],
        'url'          => $cfg['base_url'],
        'logo'         => [
            '@type'  => 'ImageObject',
            'url'    => $cfg['logo_url'],
            'width'  => 300,
            'height' => 60,
        ],
        'image'        => $cfg['og_image_url'],
        'description'  => SchemaHelpers::safeJson(
            "Istanbul's One & Only Digital Sightseeing Pass. Save over 65% on 120+ attractions."
        ),
        'foundingDate' => $cfg['founding_date'],
        'email'        => $cfg['contact_email'],
        'telephone'    => $cfg['contact_phone'],
        'address'      => [
            '@type'           => 'PostalAddress',
            'addressLocality' => 'Istanbul',
            'addressRegion'   => 'Istanbul',
            'addressCountry'  => 'TR',
        ],
        'areaServed'   => [
            '@type'  => 'City',
            'name'   => 'Istanbul',
            'sameAs' => 'https://en.wikipedia.org/wiki/Istanbul',
        ],
        'sameAs'       => $cfg['social_links'],
        'contactPoint' => array_filter([
            [
                '@type'             => 'ContactPoint',
                'telephone'         => $cfg['contact_phone'],
                'contactType'       => 'customer service',
                'availableLanguage' => $cfg['supported_languages'],
                'areaServed'        => 'Worldwide',
                'hoursAvailable'    => [
                    '@type'     => 'OpeningHoursSpecification',
                    'dayOfWeek' => ['Monday','Tuesday','Wednesday','Thursday','Friday','Saturday','Sunday'],
                    'opens'     => '08:00',
                    'closes'    => '22:00',
                ],
            ],
            !empty($cfg['whatsapp_url']) ? [
                '@type'       => 'ContactPoint',
                'contactType' => 'customer service',
                'url'         => $cfg['whatsapp_url'],
                'name'        => 'WhatsApp Support',
            ] : null,
        ]),
        'slogan' => 'See More Pay Less',
    ];
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

**PHP Builder:**

```php
public static function website(): ?array
{
    if (!request()->is('/')) {
        return null;
    }

    $base = config('schema.base_url');

    return [
        '@context' => 'https://schema.org',
        '@type'    => 'WebSite',
        '@id'      => $base . '/#website',
        'name'     => config('schema.site_name'),
        'url'      => $base,
        'publisher'=> ['@id' => $base . '/#organization'],
        'inLanguage' => app()->getLocale(),
        'potentialAction' => [
            '@type'  => 'SearchAction',
            'target' => [
                '@type'       => 'EntryPoint',
                'urlTemplate' => $base . '/search?q={search_term_string}',
            ],
            'query-input' => 'required name=search_term_string',
        ],
    ];
}
```

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

---

### 2.3 BreadcrumbList — Her Sayfada

```
⚠️ KURAL: crumbs dizisi controller'dan gelmeli. Statik yazmayın.
⚠️ FALLBACK: crumbs boş veya 1 elemanlıysa (anasayfa) → schema basma.
⚠️ KURAL: Son elemanın item (URL) alanı olmaz — sadece name.
```

**PHP Builder:**

```php
/**
 * @param array $crumbs [['label' => 'Home', 'url' => '...'], ['label' => 'Attractions', 'url' => '...'], ...]
 */
public static function breadcrumb(array $crumbs): ?array
{
    // 0 veya 1 elemanlıysa breadcrumb gereksiz
    if (count($crumbs) < 2) {
        return null;
    }

    $items = [];
    foreach ($crumbs as $i => $crumb) {
        $item = [
            '@type'    => 'ListItem',
            'position' => $i + 1,
            'name'     => SchemaHelpers::safeJson($crumb['label']),
        ];

        // Son eleman hariç hepsinin URL'i olur
        $isLast = ($i === count($crumbs) - 1);
        if (!$isLast && !empty($crumb['url'])) {
            $item['item'] = $crumb['url'];
        }

        $items[] = $item;
    }

    return [
        '@context'        => 'https://schema.org',
        '@type'           => 'BreadcrumbList',
        'itemListElement' => $items,
    ];
}
```

**Controller'da kullanım:**

```php
// AttractionController@show
$breadcrumbs = [
    ['label' => 'Home',        'url' => config('schema.base_url')],
    ['label' => 'Attractions', 'url' => config('schema.base_url') . '/whats-included'],
    ['label' => $attraction->name, 'url' => null], // son eleman
];
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

---

### 2.4 hreflang — Her Sayfada (Blade Partial)

```html
{{-- resources/views/components/schemas/hreflang.blade.php --}}
{{-- 
  ⚠️ DİKKAT: English prefix boş string olduğundan, naif birleştirme
  "https://...com//slug" gibi çift slash üretir. rtrim + ltrim ile korunur.
--}}
@php
    $map  = config('schema.hreflang_map');
    $base = rtrim(config('schema.base_url'), '/');
    $path = $slug ? '/' . ltrim($slug, '/') : '';
@endphp
@foreach($map as $code => $lang)
<link rel="alternate" hreflang="{{ $lang['hreflang'] }}" href="{{ $base }}{{ $lang['prefix'] }}{{ $path }}" />
@endforeach
<link rel="alternate" hreflang="x-default" href="{{ $base }}{{ $path }}" />
```

**Doğru çıktı (anasayfa, `$slug = ''`):**
```html
<link rel="alternate" hreflang="en" href="https://www.istanbultouristpass.com" />
<link rel="alternate" hreflang="es" href="https://www.istanbultouristpass.com/es" />
<link rel="alternate" hreflang="tr" href="https://www.istanbultouristpass.com/tr" />
```

**Doğru çıktı (attraction detay, `$slug = 'hagia-sophia-guided-tour-...'`):**
```html
<link rel="alternate" hreflang="en" href="https://www.istanbultouristpass.com/hagia-sophia-guided-tour-..." />
<link rel="alternate" hreflang="tr" href="https://www.istanbultouristpass.com/tr/hagia-sophia-guided-tour-..." />
```

---

## 3. Anasayfa

**URL:** `/`  
**Controller:** `HomeController@index`

### 3.1 Product + AggregateOffer

```
📊 VERİ KAYNAĞI: passes tablosu (aktif olanlar) + reviews tablosu (aggregate)
🔄 GÜNCELLEME: Fiyat veya review değiştiğinde otomatik
⚠️ FALLBACK: reviewCount < 1 → aggregateRating bloğunu çıkar
⚠️ FALLBACK: Hiç aktif pass yoksa → bu schema'yı basma
```

**PHP Builder:**

```php
public static function mainProduct(): ?array
{
    $passes = Pass::where('is_active', true)->get();

    if ($passes->isEmpty()) {
        return null;
    }

    $cfg  = config('schema');
    $low  = $passes->min('starting_price');
    $high = $passes->max('max_price');

    // Fiyat verisi yoksa schema basma
    if (!$low || $low <= 0) {
        return null;
    }

    $reviewCount = Review::where('is_approved', true)->count();
    $avgRating   = Review::where('is_approved', true)->avg('rating');

    $schema = [
        '@context' => 'https://schema.org',
        '@type'    => 'Product',
        '@id'      => $cfg['base_url'] . '/#product',
        'name'     => $cfg['product_name'],
        'description' => SchemaHelpers::safeJson(
            "Istanbul's original digital sightseeing pass — access 120+ attractions, skip the lines, save over 65%."
        ),
        'brand' => ['@id' => $cfg['base_url'] . '/#organization'],
        'image' => $cfg['og_image_url'],
        'category' => 'Sightseeing Pass',
        'offers' => [
            '@type'         => 'AggregateOffer',
            'lowPrice'      => (string) $low,
            'highPrice'     => (string) $high,
            'priceCurrency' => $cfg['currency'],
            'offerCount'    => (string) $passes->count(),
            'availability'  => 'https://schema.org/InStock',
            'url'           => $cfg['base_url'] . '/istanbul-pass',
            'seller'        => ['@id' => $cfg['base_url'] . '/#organization'],
            'priceValidUntil' => date('Y') . '-12-31',
        ],
    ];

    // Sadece review varsa ekle
    if ($reviewCount > 0 && $avgRating > 0) {
        $schema['aggregateRating'] = [
            '@type'       => 'AggregateRating',
            'ratingValue' => (string) round($avgRating, 1),
            'reviewCount' => (string) $reviewCount,
            'bestRating'  => '5',
            'worstRating' => '1',
        ];
    }

    return $schema;
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
📊 VERİ KAYNAĞI: reviews tablosu (is_featured = true, is_approved = true)
🔄 DÖNGÜ: foreach ile max 5 review
⚠️ FALLBACK: 0 featured review → bu schema'yı tamamen çıkar
⚠️ LİMİT: max 5 review bas
```

**PHP Builder:**

```php
public static function featuredReviews(): ?array
{
    $reviews = Review::where('is_featured', true)
        ->where('is_approved', true)
        ->where('rating', '>=', 4)
        ->orderByDesc('created_at')
        ->limit(5)
        ->get();

    if ($reviews->isEmpty()) {
        return null;
    }

    $cfg = config('schema');

    return [
        '@context' => 'https://schema.org',
        '@type'    => 'Product',
        '@id'      => $cfg['base_url'] . '/#product',
        'name'     => $cfg['product_name'],
        'review'   => $reviews->map(fn($r) => [
            '@type'  => 'Review',
            'author' => [
                '@type' => 'Person',
                'name'  => SchemaHelpers::safeJson($r->author_name),
            ],
            'datePublished' => $r->created_at->format('Y-m-d'),
            'reviewRating'  => [
                '@type'      => 'Rating',
                'ratingValue'=> (string) $r->rating,
                'bestRating' => '5',
            ],
            'reviewBody' => SchemaHelpers::safeJson($r->body, 500),
        ])->toArray(),
    ];
}
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
            "author": {
                "@type": "Person",
                "name": "Justin Savoy"
            },
            "datePublished": "2025-03-15",
            "reviewRating": {
                "@type": "Rating",
                "ratingValue": "5",
                "bestRating": "5"
            },
            "reviewBody": "Istanbul Tourist Pass is absolutely incredible! We purchased the five day pass and absolutely got our moneys worth."
        },
        {
            "@type": "Review",
            "author": {
                "@type": "Person",
                "name": "Ondra Dvoulety"
            },
            "datePublished": "2025-05-10",
            "reviewRating": {
                "@type": "Rating",
                "ratingValue": "5",
                "bestRating": "5"
            },
            "reviewBody": "We took the Istanbul pass for three days and everything worked perfectly. Customer service always available via WhatsApp."
        }
    ]
}
```

---

### 3.3 HowTo

```
📊 VERİ KAYNAĞI: CMS veya semi-static (nadiren değişir)
💡 İPUCU: Adımlar nadiren değiştiği için CMS'den veya config'den gelebilir.
```

**PHP Builder:**

```php
public static function howTo(): array
{
    // Adımlar CMS'den geliyorsa: HowToStep::orderBy('position')->get()
    // Yoksa config/hardcoded:
    $steps = [
        ['title' => 'Get Your Pass',     'text' => 'Complete your purchase, and receive your Pass ID with usage details.'],
        ['title' => 'Download & Plan',   'text' => 'Download & activate your pass, book attractions, organize your days and track your credit points — all in one app.'],
        ['title' => 'Explore & Enjoy',   'text' => 'Just Show&Go — all digital, eco-friendly access to 120+ attractions with your e-ticket QR codes.'],
    ];

    return [
        '@context'    => 'https://schema.org',
        '@type'       => 'HowTo',
        'name'        => 'How to Use Istanbul Tourist Pass',
        'description' => 'Simple, Fast, and Hassle-Free — 3 steps to explore Istanbul.',
        'totalTime'   => 'PT5M',
        'step'        => collect($steps)->map(fn($s, $i) => [
            '@type'    => 'HowToStep',
            'position' => $i + 1,
            'name'     => $s['title'],
            'text'     => $s['text'],
        ])->toArray(),
    ];
}
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

## 4. Attraction Listing

**URL:** `/whats-included`  
**Controller:** `AttractionController@index`

### 4.1 ItemList

```
📊 VERİ KAYNAĞI: attractions tablosu (is_active = true, sıralı)
🔄 DÖNGÜ: foreach ile TÜM aktif attraction'lar
⚠️ KURAL: Pagination varsa sadece sayfa 1'e ItemList bas. Sayfa 2+ için basma.
```

**PHP Builder:**

```php
public static function attractionList(Collection $attractions): ?array
{
    if ($attractions->isEmpty()) {
        return null;
    }

    $cfg = config('schema');

    return [
        '@context'        => 'https://schema.org',
        '@type'           => 'ItemList',
        'name'            => 'Istanbul Tourist Pass Attractions',
        'numberOfItems'   => $attractions->count(),
        'itemListElement' => $attractions->values()->map(fn($a, $i) => array_filter([
            '@type'    => 'ListItem',
            'position' => $i + 1,
            'url'      => $cfg['base_url'] . '/' . $a->slug,
            'name'     => SchemaHelpers::safeJson($a->name),
            'image'    => $a->featured_image_url ?: null, // null ise array_filter temizler
        ]))->toArray(),
    ];
}
```

**Gerçek Çıktı (ilk 3 attraction gösterilmiştir, tüm 120+ basılmalı):**

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

## 5. Attraction Detay

**URL:** `/:attraction_slug`  
**Controller:** `AttractionController@show`  
**En fazla schema içeren sayfa tipi: 7 adet**

### 5.1 TouristAttraction

```
📊 VERİ KAYNAĞI: attractions + attraction_locations + attraction_tags
⚠️ FALLBACK: geo yoksa → geo bloğunu çıkar. address yoksa → address'i çıkar.
```

**PHP Builder:**

```php
public static function touristAttraction(Attraction $a): array
{
    $cfg    = config('schema');
    $images = $a->images->pluck('url')->toArray();

    $schema = [
        '@context'        => 'https://schema.org',
        '@type'           => 'TouristAttraction',
        '@id'             => $cfg['base_url'] . '/' . $a->slug . '/#attraction',
        'name'            => SchemaHelpers::safeJson($a->name),
        'description'     => SchemaHelpers::safeJson($a->long_description ?: $a->short_description),
        'image'           => !empty($images) ? $images : [$a->featured_image_url ?? $cfg['og_image_url']],
        'url'             => $cfg['base_url'] . '/' . $a->slug,
        'isAccessibleForFree' => $a->is_free ?? false,
        'publicAccess'    => true,
        'containedInPlace'=> [
            '@type'  => 'City',
            'name'   => 'Istanbul',
            'sameAs' => 'https://en.wikipedia.org/wiki/Istanbul',
        ],
    ];

    // Opsiyonel: Tourist types
    if ($a->tourist_types && count($a->tourist_types) > 0) {
        $schema['touristType'] = $a->tourist_types;
    }

    // Opsiyonel: Diller
    if ($a->available_languages && count($a->available_languages) > 0) {
        $schema['availableLanguage'] = $a->available_languages;
    }

    // Opsiyonel: Koordinatlar
    if ($a->latitude && $a->longitude) {
        $schema['geo'] = [
            '@type'     => 'GeoCoordinates',
            'latitude'  => (float) $a->latitude,
            'longitude' => (float) $a->longitude,
        ];
    }

    // Opsiyonel: Adres
    if ($a->street_address) {
        $address = [
            '@type'           => 'PostalAddress',
            'streetAddress'   => SchemaHelpers::safeJson($a->street_address),
            'addressLocality' => $a->district ?? 'Istanbul',
            'addressRegion'   => 'Istanbul',
            'addressCountry'  => 'TR',
        ];
        if ($a->postal_code) {
            $address['postalCode'] = $a->postal_code;
        }
        $schema['address'] = $address;
    }

    return $schema;
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
    "image": [
        "https://www.istanbultouristpass.com/_astro/large_daf27debdbd2_Z1AeMYM.webp"
    ],
    "url": "https://www.istanbultouristpass.com/hagia-sophia-guided-tour-with-skip-the-ticket-line-entry",
    "isAccessibleForFree": false,
    "publicAccess": true,
    "containedInPlace": {
        "@type": "City",
        "name": "Istanbul",
        "sameAs": "https://en.wikipedia.org/wiki/Istanbul"
    },
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
    }
}
```

---

### 5.2 Product + Offer

```
📊 VERİ KAYNAĞI: attractions tablosu (current_price, is_active)
⚠️ KRİTİK: price, sayfada render edilen fiyatla BİREBİR eşleşmeli.
   Farklı olursa Google "price mismatch" hatası verir.
⚠️ FALLBACK: price null/0 → BU SCHEMA'YI HİÇ BASMA
```

**PHP Builder:**

```php
public static function productOffer(Attraction $a): ?array
{
    // Fiyat yoksa veya 0 ise schema basma
    $price = $a->current_price;
    if (!$price || $price <= 0) {
        return null;
    }

    $cfg = config('schema');

    $schema = [
        '@context' => 'https://schema.org',
        '@type'    => 'Product',
        'name'     => SchemaHelpers::safeJson($a->name),
        'description' => SchemaHelpers::safeJson($a->short_description),
        'image'    => $a->featured_image_url ?? $cfg['og_image_url'],
        'sku'      => 'ITP-' . ($a->sku ?? $a->id),
        'brand'    => ['@id' => $cfg['base_url'] . '/#organization'],
        'category' => SchemaHelpers::safeJson($a->category->name ?? 'Attraction'),
        'offers'   => [
            '@type'             => 'Offer',
            'price'             => (string) $price,
            'priceCurrency'     => $cfg['currency'],
            'availability'      => $a->is_active
                                    ? 'https://schema.org/InStock'
                                    : 'https://schema.org/OutOfStock',
            'validFrom'         => ($a->valid_from ?? now()->startOfYear())->format('Y-m-d'),
            'priceValidUntil'   => date('Y') . '-12-31',
            'url'               => $cfg['base_url'] . '/' . $a->slug,
            'seller'            => ['@id' => $cfg['base_url'] . '/#organization'],
            'itemCondition'     => 'https://schema.org/NewCondition',
            'hasMerchantReturnPolicy' => [
                '@type'                => 'MerchantReturnPolicy',
                'applicableCountry'    => 'TR',
                'returnPolicyCategory' => 'https://schema.org/MerchantReturnFiniteReturnWindow',
                'merchantReturnDays'   => $cfg['return_days'],
                'returnFees'           => 'https://schema.org/FreeReturn',
            ],
        ],
    ];

    // Review varsa ekle
    $reviewCount = $a->approved_reviews_count ?? $a->approvedReviews()->count();
    if ($reviewCount > 0) {
        $avgRating = $a->approved_reviews_avg_rating ?? $a->approvedReviews()->avg('rating');
        $schema['aggregateRating'] = [
            '@type'       => 'AggregateRating',
            'ratingValue' => (string) round($avgRating, 1),
            'reviewCount' => (string) $reviewCount,
            'bestRating'  => '5',
        ];
    }

    return $schema;
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
    "brand": {
        "@id": "https://www.istanbultouristpass.com/#organization"
    },
    "category": "Museums",
    "offers": {
        "@type": "Offer",
        "price": "35",
        "priceCurrency": "EUR",
        "availability": "https://schema.org/InStock",
        "validFrom": "2025-01-01",
        "priceValidUntil": "2026-12-31",
        "url": "https://www.istanbultouristpass.com/hagia-sophia-guided-tour-with-skip-the-ticket-line-entry",
        "seller": {
            "@id": "https://www.istanbultouristpass.com/#organization"
        },
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
⚠️ KOŞUL: SADECE is_guided_tour=true veya is_event=true olanlarda bas.
   Müze giriş bileti gibi attraction'lar için Event schema BASMA.
```

**PHP Builder:**

```php
public static function event(Attraction $a): ?array
{
    if (!$a->is_guided_tour && !$a->is_event) {
        return null;
    }

    $cfg = config('schema');

    $schema = [
        '@context'             => 'https://schema.org',
        '@type'                => 'Event',
        'name'                 => SchemaHelpers::safeJson($a->event_name ?? $a->name),
        'description'          => SchemaHelpers::safeJson($a->short_description),
        'image'                => $a->featured_image_url ?? $cfg['og_image_url'],
        'eventAttendanceMode'  => 'https://schema.org/OfflineEventAttendanceMode',
        'eventStatus'          => $a->is_active
                                   ? 'https://schema.org/EventScheduled'
                                   : 'https://schema.org/EventCancelled',
        'location' => [
            '@type'   => 'Place',
            'name'    => SchemaHelpers::safeJson($a->venue_name ?? $a->name),
            'address' => [
                '@type'           => 'PostalAddress',
                'addressLocality' => 'Istanbul',
                'addressCountry'  => 'TR',
            ],
        ],
        'organizer' => ['@id' => $cfg['base_url'] . '/#organization'],
        'offers'    => [
            '@type'        => 'Offer',
            'price'        => (string) $a->current_price,
            'priceCurrency'=> $cfg['currency'],
            'availability' => 'https://schema.org/InStock',
            'url'          => $cfg['base_url'] . '/' . $a->slug,
        ],
        'inLanguage' => $a->primary_language ?? 'en',
    ];

    // Opsiyonel: Schedule
    if ($a->schedule_days && $a->schedule_start_time) {
        $schema['eventSchedule'] = [
            '@type'            => 'Schedule',
            'repeatFrequency'  => 'P1D',
            'byDay'            => $a->schedule_days, // ['Monday','Tuesday',...]
            'startTime'        => $a->schedule_start_time,
            'endTime'          => $a->schedule_end_time ?? '17:00:00',
            'scheduleTimezone' => 'Europe/Istanbul',
        ];
    }

    // Opsiyonel: Koordinatlar
    if ($a->latitude && $a->longitude) {
        $schema['location']['geo'] = [
            '@type'     => 'GeoCoordinates',
            'latitude'  => (float) $a->latitude,
            'longitude' => (float) $a->longitude,
        ];
    }

    // Opsiyonel: Süre
    if ($a->duration_minutes) {
        $schema['duration'] = 'PT' . $a->duration_minutes . 'M';
    }

    return $schema;
}
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
    "organizer": {
        "@id": "https://www.istanbultouristpass.com/#organization"
    },
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
📊 VERİ KAYNAĞI: attraction_faqs tablosu (attraction_id FK)
🔄 DÖNGÜ: O attraction'a ait tüm aktif FAQ'lar
⚠️ FALLBACK: 0 FAQ → schema basma
⚠️ DİKKAT: Cevaplarda HTML olabilir — strip_tags kullan
```

**PHP Builder:**

```php
public static function faqPage($faqs): ?array
{
    // Collection veya array olabilir
    $faqs = collect($faqs)->filter(fn($f) => !empty($f->question) && !empty($f->answer));

    if ($faqs->isEmpty()) {
        return null;
    }

    return [
        '@context'   => 'https://schema.org',
        '@type'      => 'FAQPage',
        'mainEntity' => $faqs->map(fn($f) => [
            '@type' => 'Question',
            'name'  => SchemaHelpers::safeJson($f->question),
            'acceptedAnswer' => [
                '@type' => 'Answer',
                'text'  => SchemaHelpers::safeJson($f->answer),
            ],
        ])->toArray(),
    ];
}
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

## 6–7. Blog Listing & Blog Detay

### 6.1 Blog Listing (CollectionPage + ItemList)

```
📊 VERİ KAYNAĞI: articles tablosu (is_published = true, sıralı)
🔄 DÖNGÜ: foreach ile tüm yayınlanmış yazılar
⚠️ LİMİT: Pagination varsa sadece sayfa 1'e ItemList bas
💡 İPUCU: CollectionPage builder'ı Bölüm 12.2'deki collectionPage() fonksiyonunu kullanır.
   ItemList builder'ı Bölüm 4.1'deki attractionList() ile aynı pattern'dir.
```

**Controller:**

```php
// BlogController@index
public function index()
{
    $articles = Article::where('is_published', true)
        ->with('category')
        ->orderByDesc('published_at')
        ->get();

    $schemas = array_filter([
        SchemaService::breadcrumb([
            ['label' => 'Home', 'url' => config('schema.base_url')],
            ['label' => 'Blog', 'url' => null],
        ]),
        SchemaService::collectionPage(  // Bölüm 12.2'deki fonksiyon
            'blog',
            'Istanbul Travel Guide — Tips, Guides & Stories',
            'Plan your trip like a local with our Istanbul travel guides and insider tips.'
        ),
        SchemaService::articleList($articles),
    ]);

    return view('blog.index', compact('articles', 'schemas'));
}
```

**PHP Builder (ArticleList):**

```php
public static function articleList(Collection $articles): ?array
{
    if ($articles->isEmpty()) {
        return null;
    }

    $cfg = config('schema');

    return [
        '@context'        => 'https://schema.org',
        '@type'           => 'ItemList',
        'name'            => 'Istanbul Travel Guide Articles',
        'itemListElement' => $articles->values()->map(fn($a, $i) => [
            '@type'    => 'ListItem',
            'position' => $i + 1,
            'url'      => $cfg['base_url'] . '/blog/' . $a->category->slug . '/' . $a->slug,
            'name'     => SchemaHelpers::safeJson($a->title),
        ])->toArray(),
    ];
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

### 7.1 BlogPosting

```
📊 VERİ KAYNAĞI: articles + authors + categories
⚠️ KRİTİK: dateModified'ı her render'da güncellemeyin — sadece gerçek edit'te.
⚠️ FALLBACK: author yoksa → config('schema.default_author')
⚠️ FALLBACK: image yoksa → og_image_url
⚠️ FALLBACK: wordCount yoksa → alanı çıkar
```

**PHP Builder:**

```php
public static function blogPosting(Article $article): array
{
    $cfg    = config('schema');
    $author = $article->author;

    $schema = [
        '@context'  => 'https://schema.org',
        '@type'     => 'BlogPosting',
        '@id'       => $cfg['base_url'] . '/blog/' . $article->category->slug . '/' . $article->slug . '/#article',
        'headline'  => SchemaHelpers::safeJson($article->title, 110),
        'description' => SchemaHelpers::safeJson($article->meta_description ?: $article->excerpt),
        'datePublished' => $article->published_at->format('Y-m-d'),
        'inLanguage'    => app()->getLocale(),
        'url'           => $cfg['base_url'] . '/blog/' . $article->category->slug . '/' . $article->slug,
        'image' => [
            '@type'  => 'ImageObject',
            'url'    => $article->featured_image_url ?? $cfg['og_image_url'],
            'width'  => $article->featured_image_width ?? 1200,
            'height' => $article->featured_image_height ?? 630,
        ],
        'author' => [
            '@type'    => 'Person',
            '@id'      => $cfg['base_url'] . '/authors/' . ($author->slug ?? 'editorial-team') . '/#person',
            'name'     => SchemaHelpers::safeJson($author->name ?? $cfg['default_author']),
            'url'      => $author->url ?? $cfg['base_url'] . '/about-us',
            'worksFor' => ['@id' => $cfg['base_url'] . '/#organization'],
        ],
        'publisher'        => ['@id' => $cfg['base_url'] . '/#organization'],
        'mainEntityOfPage' => ['@type' => 'WebPage', '@id' => $cfg['base_url'] . '/blog/' . $article->category->slug . '/' . $article->slug . '/#webpage'],
        'articleSection'   => SchemaHelpers::safeJson($article->category->name),
        'isPartOf'         => ['@id' => $cfg['base_url'] . '/#website'],
        'about'            => ['@type' => 'City', 'name' => 'Istanbul', 'sameAs' => 'https://en.wikipedia.org/wiki/Istanbul'],
    ];

    // dateModified — sadece gerçekten güncellendiyse
    if ($article->updated_at && $article->updated_at->gt($article->published_at)) {
        $schema['dateModified'] = $article->updated_at->format('Y-m-d');
    }

    // wordCount & readTime — varsa ekle
    if ($article->word_count && $article->word_count > 0) {
        $schema['wordCount']    = $article->word_count;
        $schema['timeRequired'] = 'PT' . SchemaHelpers::readingTime($article->word_count) . 'M';
    }

    // Keywords — varsa ekle
    $tags = $article->tags->pluck('name')->toArray();
    if (!empty($tags)) {
        $schema['keywords'] = $tags;
    }

    // Author expertise — varsa ekle
    if ($author && $author->expertise && count($author->expertise) > 0) {
        $schema['author']['knowsAbout'] = $author->expertise;
    }

    // Author jobTitle — varsa ekle
    if ($author && $author->job_title) {
        $schema['author']['jobTitle'] = SchemaHelpers::safeJson($author->job_title);
    }

    return $schema;
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
        "worksFor": {
            "@id": "https://www.istanbultouristpass.com/#organization"
        },
        "knowsAbout": ["Istanbul Tourism", "Turkish Culture", "Travel Planning"]
    },
    "publisher": {
        "@id": "https://www.istanbultouristpass.com/#organization"
    },
    "mainEntityOfPage": {
        "@type": "WebPage",
        "@id": "https://www.istanbultouristpass.com/blog/tips-guides/istanbul-travel-mistakes-to-avoid/#webpage"
    },
    "articleSection": "Tips & Guides",
    "keywords": ["istanbul travel tips", "istanbul mistakes", "tourist guide istanbul"],
    "isPartOf": {
        "@id": "https://www.istanbultouristpass.com/#website"
    },
    "about": {
        "@type": "City",
        "name": "Istanbul",
        "sameAs": "https://en.wikipedia.org/wiki/Istanbul"
    }
}
```

---

## 8–9. Blog Kategori & FAQ

Blog Kategori, Blog Listing (ItemList) ile aynı pattern'i kullanır — sadece filtrelenmiş articles ve farklı breadcrumb.

FAQ sayfası, Bölüm 5.4'teki `faqPage()` builder'ını kullanır — veri kaynağı attraction değil global FAQ tablosudur:

```php
// FaqController@index
$faqs    = Faq::where('is_published', true)->orderBy('sort_order')->get();
$schemas = [
    SchemaService::breadcrumb($breadcrumbs),
    SchemaService::faqPage($faqs),  // aynı fonksiyon, farklı veri
];
```

---

## 10. Pass & Fiyatlar

**URL:** `/istanbul-pass`

### 10.1 Product (Her Pass İçin — Döngü)

```
📊 VERİ KAYNAĞI: passes tablosu + pass_variants tablosu
🔄 DÖNGÜ: Her aktif pass → ayrı bir Product schema
⚠️ KRİTİK: Variant sayısına göre Offer vs AggregateOffer seçimi otomatik
```

**PHP Builder:**

```php
public static function passProducts(): array
{
    $passes = Pass::where('is_active', true)->with('variants')->get();
    $cfg    = config('schema');

    return $passes->map(function (Pass $pass) use ($cfg) {
        $variants = $pass->variants->where('is_active', true);

        if ($variants->isEmpty()) {
            return null; // Aktif variant yoksa schema basma
        }

        $schema = [
            '@context'    => 'https://schema.org',
            '@type'       => 'Product',
            'name'        => SchemaHelpers::safeJson($pass->name),
            'description' => SchemaHelpers::safeJson($pass->description),
            'sku'         => 'ITP-' . strtoupper($pass->slug),
            'brand'       => ['@id' => $cfg['base_url'] . '/#organization'],
            'image'       => $pass->image_url ?? $cfg['og_image_url'],
            'category'    => 'Sightseeing Pass',
        ];

        // Tek variant → Offer, birden fazla → AggregateOffer
        if ($variants->count() === 1) {
            $v = $variants->first();
            $schema['offers'] = [
                '@type'           => 'Offer',
                'price'           => (string) $v->price,
                'priceCurrency'   => $cfg['currency'],
                'availability'    => 'https://schema.org/InStock',
                'url'             => $cfg['base_url'] . '/istanbul-pass?pass=' . $pass->slug,
                'priceValidUntil' => date('Y') . '-12-31',
                'seller'          => ['@id' => $cfg['base_url'] . '/#organization'],
            ];
        } else {
            $schema['offers'] = [
                '@type'           => 'AggregateOffer',
                'lowPrice'        => (string) $variants->min('price'),
                'highPrice'       => (string) $variants->max('price'),
                'priceCurrency'   => $cfg['currency'],
                'offerCount'      => (string) $variants->count(),
                'availability'    => 'https://schema.org/InStock',
                'url'             => $cfg['base_url'] . '/istanbul-pass?pass=' . $pass->slug,
                'priceValidUntil' => date('Y') . '-12-31',
                'seller'          => ['@id' => $cfg['base_url'] . '/#organization'],
            ];
        }

        // Review varsa ekle
        if ($pass->review_count > 0) {
            $schema['aggregateRating'] = [
                '@type'       => 'AggregateRating',
                'ratingValue' => (string) round($pass->average_rating, 1),
                'reviewCount' => (string) $pass->review_count,
                'bestRating'  => '5',
            ];
        }

        return $schema;
    })->filter()->values()->toArray();
}
```

**Gerçek Çıktı (PRIME Pass örneği — 3 pass'ın biri):**

```json
{
    "@context": "https://schema.org",
    "@type": "Product",
    "name": "Istanbul PRIME Pass®",
    "description": "Ultimate Istanbul Experience — 7 landmarks, 120+ top attractions, premium activities & experiences. Priority hosted access + dedicated support line.",
    "sku": "ITP-PRIME",
    "brand": {
        "@id": "https://www.istanbultouristpass.com/#organization"
    },
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
        "seller": {
            "@id": "https://www.istanbultouristpass.com/#organization"
        }
    },
    "aggregateRating": {
        "@type": "AggregateRating",
        "ratingValue": "4.8",
        "reviewCount": "5500",
        "bestRating": "5"
    }
}
```

> **⚠️ Controller'da kullanım notu:** `passProducts()` birden fazla schema döndürür (her pass = ayrı Product). `renderAll()` ile kullanırken `array_merge` ile spread edin:

```php
// PassController@index
$passSchemas = SchemaService::passProducts(); // [Product1, Product2, Product3]

$schemas = array_merge(
    [
        SchemaService::breadcrumb($breadcrumbs),
    ],
    $passSchemas  // spread — her biri ayrı <script> bloğu olur
);
```

---

## 11. Reviews Sayfası

**URL:** `/reviews`  
**Controller:** `ReviewController@index`

```
📊 VERİ KAYNAĞI: reviews tablosu (tüm approved, limit 10)
🔄 DÖNGÜ: foreach — anasayfadan farkı: limit 10, video testimonials eklenir
⚠️ FALLBACK: 0 review → schema basma
```

Anasayfadaki `featuredReviews()` builder'ını (Bölüm 3.2) tekrar kullanın, tek fark `limit(10)`:

```php
// ReviewController@index
$reviews = Review::where('is_approved', true)
    ->orderByDesc('created_at')
    ->limit(10)
    ->get();

$schemas = array_filter([
    SchemaService::breadcrumb($breadcrumbs),
    $reviews->isNotEmpty() ? SchemaService::reviewCollection($reviews) : null,
    // Video testimonials varsa:
    ...SchemaService::videoTestimonials(
        VideoTestimonial::where('is_active', true)->limit(5)->get()
    ),
]);
```

### 11.1 VideoObject (Testimonial Video)

```
📊 VERİ KAYNAĞI: video_testimonials tablosu
🔄 DÖNGÜ: Her video ayrı bir schema
⚠️ FALLBACK: Video yoksa basma. duration yoksa alanı çıkar.
```

**PHP Builder:**

```php
/**
 * Birden fazla VideoObject schema üretir (her video = ayrı schema).
 * @return array[] Spread edilerek schemas dizisine eklenebilir
 */
public static function videoTestimonials($videos): array
{
    return collect($videos)->filter()->map(function ($video) {
        $schema = [
            '@context'     => 'https://schema.org',
            '@type'        => 'VideoObject',
            'name'         => SchemaHelpers::safeJson($video->title),
            'description'  => SchemaHelpers::safeJson($video->description ?: $video->title),
            'thumbnailUrl' => $video->thumbnail_url,
            'uploadDate'   => $video->published_at?->format('Y-m-d')
                              ?? $video->created_at->format('Y-m-d'),
            'contentUrl'   => $video->youtube_url ?? $video->video_url,
            'publisher'    => ['@id' => config('schema.base_url') . '/#organization'],
        ];

        // Opsiyonel: YouTube embed
        if ($video->youtube_id) {
            $schema['embedUrl'] = 'https://www.youtube.com/embed/' . $video->youtube_id;
        }

        // Opsiyonel: Süre
        if ($video->duration_seconds && $video->duration_seconds > 0) {
            $schema['duration'] = SchemaHelpers::isoDuration($video->duration_seconds);
        }

        return $schema;
    })->values()->toArray();
}
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
    "publisher": {
        "@id": "https://www.istanbultouristpass.com/#organization"
    },
    "embedUrl": "https://www.youtube.com/embed/7JQ-kfExIf8",
    "duration": "PT2M15S"
}
```

---

## 12. Diğer Sayfa Tipleri

Bu sayfaların hepsi generic bir `WebPage` schema + `BreadcrumbList` kullanır. Tek bir reusable fonksiyon yeterlidir:

**PHP Builder:**

```php
/**
 * Generic WebPage schema — About, Contact, How It Works, Compare, Legal vs. için.
 *
 * @param string $slug         Sayfa slug'ı ("about-us", "contact", "privacy-policy")
 * @param string $title        Meta title
 * @param string $description  Meta description
 * @param string $schemaType   Schema @type override (default: WebPage)
 * @param array|null $mainEntity  Opsiyonel mainEntity (Service, DigitalDocument vs.)
 */
public static function genericPage(
    string $slug,
    string $title,
    string $description,
    string $schemaType = 'WebPage',
    ?array $mainEntity = null
): array {
    $cfg = config('schema');

    $schema = [
        '@context'    => 'https://schema.org',
        '@type'       => $schemaType,
        '@id'         => $cfg['base_url'] . '/' . $slug . '/#webpage',
        'name'        => SchemaHelpers::safeJson($title),
        'description' => SchemaHelpers::safeJson($description),
        'url'         => $cfg['base_url'] . '/' . $slug,
        'isPartOf'    => ['@id' => $cfg['base_url'] . '/#website'],
        'inLanguage'  => app()->getLocale(),
    ];

    if ($mainEntity) {
        $schema['mainEntity'] = $mainEntity;
    }

    return $schema;
}
```

**Sayfa bazlı kullanım örnekleri:**

| Sayfa | `$schemaType` | `$mainEntity` |
|---|---|---|
| `/about-us` | `AboutPage` | `['@id' => '.../#organization']` |
| `/contact` | `ContactPage` | Organization ContactPoint referansı |
| `/how-it-works` | `WebPage` | — (HowTo ayrı basılır) |
| `/compare-passes` | `WebPage` | — |
| `/download-app` | `WebPage` | — (SoftwareApplication ayrı basılır) |
| `/plan-and-save` | `WebPage` | — |
| `/what-you-get` | `WebPage` | `['@id' => '.../#product']` |
| `/free-digital-istanbul-guidebook` | `WebPage` | `['@type' => 'DigitalDocument', ...]` |
| `/group-request` | `WebPage` | `['@type' => 'Service', ...]` |
| `/become-affiliate-partner` | `WebPage` | — |
| `/privacy-policy` | `WebPage` | — (+ `lastReviewed` alanı ekle) |

**Gerçek Çıktı (About sayfası):**

```json
{
    "@context": "https://schema.org",
    "@type": "AboutPage",
    "@id": "https://www.istanbultouristpass.com/about-us/#webpage",
    "name": "About Istanbul Tourist Pass — Istanbul Experts Since 1995",
    "description": "Learn about Istanbul Tourist Pass — Istanbul's original sightseeing pass trusted by 500K+ travelers from 150+ countries since 2015.",
    "url": "https://www.istanbultouristpass.com/about-us",
    "isPartOf": {
        "@id": "https://www.istanbultouristpass.com/#website"
    },
    "inLanguage": "en",
    "mainEntity": {
        "@id": "https://www.istanbultouristpass.com/#organization"
    }
}
```

---

### 12.2 CollectionPage (Blog Listing & Attraction Listing için)

```
📊 VERİ KAYNAĞI: page meta verileri
💡 İPUCU: Bu builder hem /blog hem /whats-included hem blog kategori sayfalarında kullanılır.
```

**PHP Builder:**

```php
public static function collectionPage(string $slug, string $title, string $description): array
{
    $cfg = config('schema');

    return [
        '@context'    => 'https://schema.org',
        '@type'       => 'CollectionPage',
        '@id'         => $cfg['base_url'] . '/' . $slug . '/#webpage',
        'name'        => SchemaHelpers::safeJson($title),
        'description' => SchemaHelpers::safeJson($description),
        'url'         => $cfg['base_url'] . '/' . $slug,
        'isPartOf'    => ['@id' => $cfg['base_url'] . '/#website'],
        'inLanguage'  => app()->getLocale(),
    ];
}
```

**Gerçek Çıktı (`/blog`):**

```json
{
    "@context": "https://schema.org",
    "@type": "CollectionPage",
    "@id": "https://www.istanbultouristpass.com/blog/#webpage",
    "name": "Istanbul Travel Guide — Tips, Guides & Stories",
    "description": "Plan your trip like a local with our Istanbul travel guides, insider tips, and expert recommendations.",
    "url": "https://www.istanbultouristpass.com/blog",
    "isPartOf": {
        "@id": "https://www.istanbultouristpass.com/#website"
    },
    "inLanguage": "en"
}
```

---

### 12.3 ContactPage + ContactPoint

```
📊 VERİ KAYNAĞI: config/schema.php (iletişim bilgileri)
```

**PHP Builder:**

```php
public static function contactPage(): array
{
    $cfg = config('schema');

    return [
        '@context'    => 'https://schema.org',
        '@type'       => 'ContactPage',
        '@id'         => $cfg['base_url'] . '/contact/#webpage',
        'name'        => 'Contact Istanbul Tourist Pass — Get Help',
        'description' => 'Get in touch with our Istanbul expert team via phone, WhatsApp, email, or live chat.',
        'url'         => $cfg['base_url'] . '/contact',
        'isPartOf'    => ['@id' => $cfg['base_url'] . '/#website'],
        'inLanguage'  => app()->getLocale(),
        'mainEntity'  => [
            '@id' => $cfg['base_url'] . '/#organization',
        ],
    ];
}
```

**Gerçek Çıktı:**

```json
{
    "@context": "https://schema.org",
    "@type": "ContactPage",
    "@id": "https://www.istanbultouristpass.com/contact/#webpage",
    "name": "Contact Istanbul Tourist Pass — Get Help",
    "description": "Get in touch with our Istanbul expert team via phone, WhatsApp, email, or live chat.",
    "url": "https://www.istanbultouristpass.com/contact",
    "isPartOf": {
        "@id": "https://www.istanbultouristpass.com/#website"
    },
    "inLanguage": "en",
    "mainEntity": {
        "@id": "https://www.istanbultouristpass.com/#organization"
    }
}
```

> **Not:** ContactPoint detayları zaten Organization schema'sında (Bölüm 2.1) tanımlıdır. Google, `@id` referansı ile otomatik birleştirir. Ayrı bir ContactPoint schema'sı basmaya gerek yoktur.

---

### 12.4 SoftwareApplication (Download App sayfası)

```
📊 VERİ KAYNAĞI: config/schema.php (app URL'leri, uygulama bilgisi)
⚠️ FALLBACK: App store URL'leri yoksa bu schema'yı basma
```

**PHP Builder:**

```php
public static function softwareApplication(): ?array
{
    $cfg = config('schema');

    // App store URL'leri yoksa schema basma
    if (empty($cfg['ios_app_url']) && empty($cfg['android_app_url'])) {
        return null;
    }

    $schema = [
        '@context'            => 'https://schema.org',
        '@type'               => 'SoftwareApplication',
        'name'                => $cfg['site_name'],
        'operatingSystem'     => 'iOS 15.0+, Android 8.0+',
        'applicationCategory' => 'TravelApplication',
        'description'         => SchemaHelpers::safeJson(
            'Download the Istanbul Tourist Pass app to access your digital pass, manage reservations, and explore Istanbul with audio guides in 25 languages.'
        ),
        'offers' => [
            '@type'         => 'Offer',
            'price'         => '0',
            'priceCurrency' => $cfg['currency'],
        ],
        'author' => ['@id' => $cfg['base_url'] . '/#organization'],
    ];

    if (!empty($cfg['ios_app_url'])) {
        $schema['installUrl'] = $cfg['ios_app_url'];
    }
    if (!empty($cfg['android_app_url'])) {
        $schema['downloadUrl'] = $cfg['android_app_url'];
    }

    return $schema;
}
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
    "offers": {
        "@type": "Offer",
        "price": "0",
        "priceCurrency": "EUR"
    },
    "author": {
        "@id": "https://www.istanbultouristpass.com/#organization"
    },
    "installUrl": "https://apps.apple.com/tr/app/istanbul-tourist-pass/id6535683117",
    "downloadUrl": "https://play.google.com/store/apps/details?id=com.istanbultouristpass.app"
}
```

---

### 12.5 SiteNavigationElement (Anasayfada, opsiyonel)

```
📊 VERİ KAYNAĞI: navigation/menu tablosu veya config
🔄 DÖNGÜ: Ana menü item'ları üzerinden
💡 ÖNCELİK: 🟢 BONUS — Sprint 3
```

**PHP Builder:**

```php
public static function siteNavigation(): ?array
{
    // Menü item'ları CMS'den veya config'den
    $menuItems = MenuItem::where('menu_id', 'main')
        ->where('is_visible', true)
        ->orderBy('sort_order')
        ->get();

    if ($menuItems->isEmpty()) {
        return null;
    }

    $cfg = config('schema');

    return [
        '@context' => 'https://schema.org',
        '@type'    => 'SiteNavigationElement',
        'name'     => 'Main Navigation',
        'hasPart'  => $menuItems->map(fn($item) => [
            '@type' => 'WebPage',
            'name'  => SchemaHelpers::safeJson($item->label),
            'url'   => $item->full_url ?? $cfg['base_url'] . '/' . $item->slug,
        ])->toArray(),
    ];
}
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

### 12.6 OfferCatalog (Attraction Listing — kategori bazlı)

```
📊 VERİ KAYNAĞI: attraction_categories tablosu + count query
🔄 DÖNGÜ: Kategoriler ve her birinin aktif attraction sayısı
⚠️ FALLBACK: Kategoride 0 attraction varsa o kategoriyi atla
💡 ÖNCELİK: 🟢 BONUS — Sprint 3
```

**PHP Builder:**

```php
public static function offerCatalog(): ?array
{
    $categories = AttractionCategory::withCount(['attractions' => function ($q) {
        $q->where('is_active', true);
    }])->having('attractions_count', '>', 0)->get();

    if ($categories->isEmpty()) {
        return null;
    }

    return [
        '@context'        => 'https://schema.org',
        '@type'           => 'OfferCatalog',
        'name'            => 'Istanbul Tourist Pass Attraction Categories',
        'itemListElement' => $categories->map(fn($c) => [
            '@type'         => 'OfferCatalog',
            'name'          => SchemaHelpers::safeJson($c->name),
            'numberOfItems' => $c->attractions_count,
        ])->toArray(),
    ];
}
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

## 13. Implementasyon Checklist

### 13.1 Test Matrisi

| # | Test | Nasıl | Geçme Kriteri |
|---|---|---|---|
| 1 | JSON syntax | `json_decode()` + `json_last_error()` | Hata yok |
| 2 | Google Rich Results | https://search.google.com/test/rich-results | Tüm schema'lar valid |
| 3 | Fiyat tutarlılığı | Schema price vs. sayfadaki price | Birebir eşleşme |
| 4 | Boş veri senaryosu | DB'den image/review/faq sil, sayfayı test et | JSON bozulmamış, blok basılmamış |
| 5 | Multi-language | `/tr/`, `/de/` sayfaları | `inLanguage` doğru, içerik o dilde |
| 6 | Draft sayfa | Unpublished attraction/blog | Schema hiç basılmamış |
| 7 | Yeni kayıt | Yeni attraction ekle → detay sayfası | Schema'lar otomatik oluşmuş |
| 8 | XSS | `name` alanına `<script>alert(1)</script>` yaz | Tag temizlenmiş |
| 9 | Null price | Attraction fiyatını null yap | Product schema basılmamış |
| 10 | 0 review | Attraction'ın tüm review'larını sil | aggregateRating bloğu yok |

### 13.2 Yaygın Hatalar

| Hata | Neden | Çözüm |
|---|---|---|
| `price mismatch` | Schema fiyatı ≠ sayfadaki | Aynı `$attraction->current_price` kaynağını kullan |
| `Missing field "image"` | null gelmiş | `?? config('schema.og_image_url')` fallback |
| `Invalid date format` | Boş string basılmış | `if ($date) {}` koşulu |
| `Duplicate schema` | Aynı Product 2 kez | `@id` kontrolü |
| `JSON parse error` | Tırnak/newline var | `SchemaHelpers::safeJson()` kullan |
| `priceValidUntil expired` | Geçen yıl kalmış | `date('Y') . '-12-31'` dinamik hesapla |

### 13.3 Sprint Planı

| Sprint | Süre | Kapsam | Schema |
|---|---|---|---|
| **1** | 1-2 hafta | SchemaService + helpers + global bileşenler + Anasayfa + Attraction Detay + Blog Detay + FAQ + Pass | ~35 |
| **2** | 1-2 hafta | Listing'ler + ItemList + About/Contact/HowItWorks + App + Kategori | ~18 |
| **3** | 1 hafta | Service sayfaları + Partner + Legal + Guidebook + Video'lar | ~9 |
| **TOPLAM** | | **22 sayfa tipi** | **~62** |

### 13.4 Doğrulama Araçları

| Araç | URL | Ne İçin |
|---|---|---|
| Google Rich Results Test | https://search.google.com/test/rich-results | Doğrulama + preview |
| Schema.org Validator | https://validator.schema.org | Syntax |
| Google Search Console | https://search.google.com/search-console | Canlı performans |
| JSON-LD Playground | https://json-ld.org/playground | Linked data |

---

> **Toplam:** 22 sayfa tipi, ~62 schema, 3-katmanlı PHP bileşen mimarisi  
> **Her schema için:** PHP builder kodu + gerçek çıktı örneği + fallback kuralları + veri kaynağı notu  
> **Tüm builder'lar:** Organization, WebSite, BreadcrumbList, hreflang, Product, AggregateOffer, Review, HowTo, ItemList, CollectionPage, TouristAttraction, Offer, Event, FAQPage, BlogPosting, VideoObject, SoftwareApplication, ContactPage, SiteNavigationElement, OfferCatalog, genericPage  
> **Değişkenler:** Laravel/Eloquent varsayılmıştır — kendi ORM/framework'ünüze uyarlayın
