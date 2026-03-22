# Veri Yapısı Dokümantasyonu

## Article (Haber) Şeması

Her haber aşağıdaki yapıya sahiptir:

```typescript
interface Article {
  // Temel Bilgiler
  id: string;                    // Benzersiz ID (Pushholder'dan)
  slug: string;                  // URL-friendly slug
  title: string;                 // Haber başlığı
  summary: string;               // AI tarafından oluşturulan özet
  content: string;               // Tam içerik (HTML olabilir)

  // Görsel
  imageUrl: string;              // Kapak görseli URL

  // Kaynak Bilgileri (Orijinal)
  source: string;                // Kaynak adı (örn: "CNN Türk")
  sourceUrl: string;             // Kaynak web sitesi
  sourceLogo: string;            // Kaynak logosu URL
  originalUrl: string;           // Haberin orijinal linki

  // Kategori
  category: string;              // Ana kategori
  categories: Category[];        // Tüm kategoriler
  tags: string[];                // Etiketler

  // Zaman
  publishedAt: string;           // Yayınlanma zamanı (ISO 8601)
  fetchedAt: string;             // Çekilme zamanı (ISO 8601)

  // İstatistikler (opsiyonel)
  viewCount?: number;
  likeCount?: number;

  // Öne çıkan mı?
  isFeatured?: boolean;
}

interface Category {
  id: string;
  name: string;
  slug: string;
}
```

## Dosya Yapıları

### latest.json (Web için)

```json
{
  "version": "1.0",
  "updatedAt": "2026-03-22T10:00:00.000Z",
  "count": 10,
  "articles": [
    { /* Article */ },
    { /* Article */ }
  ]
}
```

### daily/{date}.json (Günlük Arşiv)

```json
{
  "date": "2026-03-22",
  "updatedAt": "2026-03-22T10:00:00.000Z",
  "count": 150,
  "articles": [
    { /* Article */ }
  ]
}
```

### categories/{category}.json (Kategori Bazlı)

```json
{
  "category": "Spor",
  "slug": "spor",
  "updatedAt": "2026-03-22T10:00:00.000Z",
  "count": 100,
  "articles": [
    { /* Article */ }
  ]
}
```

### manifest.json (Metadata)

```json
{
  "version": "1.0",
  "updatedAt": "2026-03-22T10:00:00.000Z",
  "retentionDays": 7,
  "updateIntervalMinutes": 10,
  "categories": [
    { "name": "Gündem", "slug": "gundem", "count": 50 },
    { "name": "Spor", "slug": "spor", "count": 30 }
  ],
  "dailyFiles": [
    "2026-03-22",
    "2026-03-21"
  ],
  "stats": {
    "totalArticles": 500,
    "todayArticles": 80,
    "sources": ["CNN Türk", "NTV", "Halk TV"]
  }
}
```

## Kategori Eşleştirmesi

| Pushholder Kategorisi | Slug | Dosya Adı |
|----------------------|------|-----------|
| Gündem | gundem | gundem.json |
| Spor | spor | spor.json |
| Ekonomi | ekonomi | ekonomi.json |
| Teknoloji | teknoloji | teknoloji.json |
| Sağlık | saglik | saglik.json |
| Bilim | bilim | bilim.json |
| Siyaset | siyaset | siyaset.json |
| Genel | genel | genel.json |

## Blacklist (Filtrelenen Kaynaklar)

Aşağıdaki kaynaklar filtrelenir ve gösterilmez:
- Maçkolik

## Veri Retention

- Günlük dosyalar: **7 gün** tutulur
- Kategori dosyaları: Son **100 haber** tutulur
- latest.json: Son **10 haber** tutulur
