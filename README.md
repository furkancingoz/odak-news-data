# Odak News Data

Odak haber platformu için merkezi veri deposu. Web ve mobil uygulamalar bu repo'dan veri çeker.

## Veri Yapısı

```
odak-news-data/
├── latest.json              # Son 10 haber (Web landing için)
├── manifest.json            # Metadata ve son güncelleme bilgisi
├── daily/                   # Günlük haberler (7 gün tutulur)
│   ├── 2026-03-22.json
│   ├── 2026-03-21.json
│   └── ...
├── categories/              # Kategori bazlı haberler
│   ├── gundem.json
│   ├── spor.json
│   ├── ekonomi.json
│   ├── teknoloji.json
│   ├── saglik.json
│   ├── bilim.json
│   ├── siyaset.json
│   └── genel.json
└── docs/                    # Dokümantasyon
    ├── DATA_STRUCTURE.md
    ├── API_USAGE.md
    └── MOBILE_INTEGRATION.md
```

## Hızlı Başlangıç

### Web için (Son 10 Haber)
```javascript
const response = await fetch('https://raw.githubusercontent.com/furkancingoz/odak-news-data/main/latest.json');
const data = await response.json();
console.log(data.articles); // Son 10 haber
```

### Mobil için (Kategori Bazlı)
```javascript
const response = await fetch('https://raw.githubusercontent.com/furkancingoz/odak-news-data/main/categories/spor.json');
const data = await response.json();
console.log(data.articles); // Spor haberleri (son 100)
```

## Güncelleme Sıklığı

- **Otomatik güncelleme**: Her 10 dakikada bir
- **Veri kaynağı**: Pushholder API
- **Retention**: 7 gün

## Kategoriler

| Kategori | Dosya | Açıklama |
|----------|-------|----------|
| Gündem | `gundem.json` | Genel gündem haberleri |
| Spor | `spor.json` | Spor haberleri |
| Ekonomi | `ekonomi.json` | Ekonomi ve finans |
| Teknoloji | `teknoloji.json` | Teknoloji haberleri |
| Sağlık | `saglik.json` | Sağlık haberleri |
| Bilim | `bilim.json` | Bilim haberleri |
| Siyaset | `siyaset.json` | Siyaset haberleri |
| Genel | `genel.json` | Genel haberler |

## Lisans

Bu veri deposu Odak uygulaması için kullanılmak üzere tasarlanmıştır.
Haberler orijinal kaynaklarına aittir.

---

**Odak** - Sadece bilmen gerekenler.
