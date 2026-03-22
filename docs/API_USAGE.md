# API Kullanım Kılavuzu

## Base URL

```
https://raw.githubusercontent.com/furkancingoz/odak-news-data/main/
```

## Endpoints

### 1. Son Haberler (Web Landing)

**URL**: `/latest.json`

**Kullanım**:
```javascript
const LATEST_URL = 'https://raw.githubusercontent.com/furkancingoz/odak-news-data/main/latest.json';

async function getLatestNews() {
  const response = await fetch(LATEST_URL);
  const data = await response.json();
  return data.articles; // 10 haber
}
```

**Response**:
```json
{
  "version": "1.0",
  "updatedAt": "2026-03-22T10:00:00.000Z",
  "count": 10,
  "articles": [...]
}
```

### 2. Kategori Haberleri (Mobil App)

**URL**: `/categories/{kategori}.json`

**Kategoriler**: `gundem`, `spor`, `ekonomi`, `teknoloji`, `saglik`, `bilim`, `siyaset`, `genel`

**Kullanım**:
```javascript
async function getCategoryNews(category) {
  const url = `https://raw.githubusercontent.com/furkancingoz/odak-news-data/main/categories/${category}.json`;
  const response = await fetch(url);
  const data = await response.json();
  return data.articles; // 100 haber
}

// Örnek
const sporNews = await getCategoryNews('spor');
```

### 3. Günlük Arşiv

**URL**: `/daily/{YYYY-MM-DD}.json`

**Kullanım**:
```javascript
async function getDailyNews(date) {
  const url = `https://raw.githubusercontent.com/furkancingoz/odak-news-data/main/daily/${date}.json`;
  const response = await fetch(url);
  const data = await response.json();
  return data.articles;
}

// Örnek
const today = new Date().toISOString().split('T')[0];
const todayNews = await getDailyNews(today);
```

### 4. Manifest (Metadata)

**URL**: `/manifest.json`

**Kullanım**:
```javascript
async function getManifest() {
  const url = 'https://raw.githubusercontent.com/furkancingoz/odak-news-data/main/manifest.json';
  const response = await fetch(url);
  return await response.json();
}

// Güncel kategorileri al
const manifest = await getManifest();
console.log(manifest.categories);
console.log(manifest.updatedAt);
```

## Cache Stratejisi

### Web
```javascript
// 10 dakika cache
const CACHE_DURATION = 10 * 60 * 1000;

let cache = { data: null, timestamp: 0 };

async function getLatestWithCache() {
  if (Date.now() - cache.timestamp < CACHE_DURATION && cache.data) {
    return cache.data;
  }

  const data = await getLatestNews();
  cache = { data, timestamp: Date.now() };
  return data;
}
```

### Mobil (Flutter)
```dart
// shared_preferences ile cache
Future<List<Article>> getCachedNews(String category) async {
  final prefs = await SharedPreferences.getInstance();
  final cacheKey = 'news_$category';
  final cacheTimeKey = 'news_${category}_time';

  final cachedTime = prefs.getInt(cacheTimeKey) ?? 0;
  final now = DateTime.now().millisecondsSinceEpoch;

  // 10 dakika cache
  if (now - cachedTime < 600000) {
    final cached = prefs.getString(cacheKey);
    if (cached != null) {
      return parseArticles(jsonDecode(cached));
    }
  }

  // Fetch fresh data
  final data = await fetchCategoryNews(category);
  prefs.setString(cacheKey, jsonEncode(data));
  prefs.setInt(cacheTimeKey, now);
  return data;
}
```

## Error Handling

```javascript
async function fetchWithFallback(url) {
  try {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    console.error('Fetch error:', error);
    // Fallback: cached data veya boş array
    return { articles: [] };
  }
}
```

## Rate Limiting

GitHub Raw CDN'de rate limit yok, ancak:
- İstemci tarafında cache kullanın (10 dk önerilir)
- Gereksiz istek atmayın
- Manifest'i kontrol ederek sadece değişiklik varsa çekin

## CORS

GitHub Raw URL'leri CORS destekler. Herhangi bir domain'den istek atabilirsiniz.
