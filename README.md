# Odak News Data

Odak haber platformu merkezi veri deposu. Tüm haberler burada arşivlenir ve indekslenir.

## Veri Yapısı

```
odak-news-data/
├── latest.json              # Son 50 haber (Web için)
├── manifest.json            # Metadata ve son güncelleme
├── daily/                   # Günlük haberler (TÜM ARŞİV)
│   ├── 2026/
│   │   └── 03/
│   │       ├── 23.json      # 2026-03-23 haberleri
│   │       ├── 22.json
│   │       └── ...
│   └── index.json           # Tüm günlerin listesi
├── categories/              # Kategori bazlı (son 100)
│   ├── gundem.json
│   ├── ekonomi.json
│   ├── spor.json
│   ├── teknoloji.json
│   ├── saglik.json
│   ├── bilim.json
│   ├── siyaset.json
│   └── genel.json
├── social/                  # Sosyal medya kuyruğu
│   ├── queue.json           # Paylaşım bekleyenler
│   └── posted.json          # Paylaşılanlar
└── stats/                   # İstatistikler
    └── daily.json           # Günlük istatistikler
```

## Haber Yapısı

```json
{
  "id": "abc123",
  "slug": "haber-basligi-abc123",
  "title": "Haber Başlığı",
  "summary": "Haber özeti...",
  "content": "Tam içerik...",
  "imageUrl": "https://...",
  "source": "Anadolu Ajansı",
  "category": "Gündem",
  "publishedAt": "2026-03-23T14:30:00Z",
  "score": 85,
  "r2ImageUrl": "https://r2.dev/posts/slug.jpg"
}
```

## API Kullanımı

### Son Haberler
```javascript
const res = await fetch('https://raw.githubusercontent.com/furkancingoz/odak-news-data/main/latest.json');
const { articles } = await res.json();
```

### Belirli Bir Gün
```javascript
const res = await fetch('https://raw.githubusercontent.com/furkancingoz/odak-news-data/main/daily/2026/03/23.json');
const { articles } = await res.json();
```

### Kategori
```javascript
const res = await fetch('https://raw.githubusercontent.com/furkancingoz/odak-news-data/main/categories/ekonomi.json');
const { articles } = await res.json();
```

### Sosyal Medya Kuyruğu
```javascript
const res = await fetch('https://raw.githubusercontent.com/furkancingoz/odak-news-data/main/social/queue.json');
const { articles } = await res.json();
// Skor >= 75 olan haberler
```

## Güncelleme

- **Sıklık**: Her 15-20 dakikada
- **Kaynak**: Pushholder API + Akıllı Filtreleme
- **Arşiv**: Kalıcı (silinmez)
- **R2 Görseller**: Skor >= 75 olanlar için

## Skorlama

| Skor | Anlam | Aksiyon |
|------|-------|---------|
| 85-100 | Çok Önemli | Sosyal medya + Site |
| 75-84 | Önemli | Sosyal medya önerisi + Site |
| 40-74 | Normal | Sadece site |
| 0-39 | Düşük | Atla |

## İndeksleme Avantajları

Tüm haberler arşivlenerek:
- Google indekslemesi için içerik zenginliği
- Geçmiş haberlere kolay erişim
- Trend analizi imkanı
- SEO için geniş içerik tabanı

---

**Odak** - Sadece bilmen gerekenler.
