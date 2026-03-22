# Mobil Uygulama Entegrasyon Kılavuzu

## Platform Desteği

- **Android**: Flutter (öncelikli)
- **iOS**: Swift Native (sonrası)

## Mimari

```
┌─────────────────────────────────────────────────────────────┐
│                    ODAK MOBİL APP                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │  Ana Sayfa  │  │ Kategoriler │  │  Haber Detay        │ │
│  │  (Latest)   │  │  (Tabs)     │  │  + WebView Kaynak   │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
│                                                             │
│  ┌─────────────────────────────────────────────────────────┤
│  │                 DATA LAYER                              │
│  │  • GitHub Raw API (odak-news-data repo)                 │
│  │  • Local Cache (SharedPreferences / UserDefaults)       │
│  │  • Firebase Cloud Messaging (Push Notifications)        │
│  └─────────────────────────────────────────────────────────┤
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Flutter Implementasyonu

### 1. Article Model

```dart
// lib/models/article.dart
class Article {
  final String id;
  final String slug;
  final String title;
  final String summary;
  final String content;
  final String imageUrl;
  final String source;
  final String sourceUrl;
  final String sourceLogo;
  final String originalUrl;
  final String category;
  final DateTime publishedAt;

  Article({
    required this.id,
    required this.slug,
    required this.title,
    required this.summary,
    required this.content,
    required this.imageUrl,
    required this.source,
    required this.sourceUrl,
    required this.sourceLogo,
    required this.originalUrl,
    required this.category,
    required this.publishedAt,
  });

  factory Article.fromJson(Map<String, dynamic> json) {
    return Article(
      id: json['id'] ?? '',
      slug: json['slug'] ?? '',
      title: json['title'] ?? '',
      summary: json['summary'] ?? '',
      content: json['content'] ?? '',
      imageUrl: json['imageUrl'] ?? '',
      source: json['source'] ?? '',
      sourceUrl: json['sourceUrl'] ?? '',
      sourceLogo: json['sourceLogo'] ?? '',
      originalUrl: json['originalUrl'] ?? '',
      category: json['category'] ?? 'Genel',
      publishedAt: DateTime.parse(json['publishedAt'] ?? DateTime.now().toIso8601String()),
    );
  }
}
```

### 2. News Service

```dart
// lib/services/news_service.dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class NewsService {
  static const String baseUrl = 'https://raw.githubusercontent.com/furkancingoz/odak-news-data/main';

  // Son 10 haber (Ana Sayfa)
  Future<List<Article>> getLatestNews() async {
    final response = await http.get(Uri.parse('$baseUrl/latest.json'));
    if (response.statusCode == 200) {
      final data = jsonDecode(response.body);
      return (data['articles'] as List)
          .map((json) => Article.fromJson(json))
          .toList();
    }
    throw Exception('Failed to load latest news');
  }

  // Kategori haberleri
  Future<List<Article>> getCategoryNews(String category) async {
    final response = await http.get(Uri.parse('$baseUrl/categories/$category.json'));
    if (response.statusCode == 200) {
      final data = jsonDecode(response.body);
      return (data['articles'] as List)
          .map((json) => Article.fromJson(json))
          .toList();
    }
    throw Exception('Failed to load $category news');
  }

  // Manifest
  Future<Map<String, dynamic>> getManifest() async {
    final response = await http.get(Uri.parse('$baseUrl/manifest.json'));
    if (response.statusCode == 200) {
      return jsonDecode(response.body);
    }
    throw Exception('Failed to load manifest');
  }
}
```

### 3. Category Tabs

```dart
// lib/widgets/category_tabs.dart
class CategoryTabs extends StatelessWidget {
  static const categories = [
    {'name': 'Gündem', 'slug': 'gundem'},
    {'name': 'Spor', 'slug': 'spor'},
    {'name': 'Ekonomi', 'slug': 'ekonomi'},
    {'name': 'Teknoloji', 'slug': 'teknoloji'},
    {'name': 'Sağlık', 'slug': 'saglik'},
    {'name': 'Bilim', 'slug': 'bilim'},
    {'name': 'Siyaset', 'slug': 'siyaset'},
    {'name': 'Genel', 'slug': 'genel'},
  ];

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: categories.length,
      child: Column(
        children: [
          TabBar(
            isScrollable: true,
            tabs: categories.map((c) => Tab(text: c['name'])).toList(),
          ),
          Expanded(
            child: TabBarView(
              children: categories.map((c) =>
                CategoryNewsList(category: c['slug']!)
              ).toList(),
            ),
          ),
        ],
      ),
    );
  }
}
```

## Push Notifications (FCM)

### Konfigürasyon

```dart
// lib/services/notification_service.dart
import 'package:firebase_messaging/firebase_messaging.dart';

class NotificationService {
  final FirebaseMessaging _fcm = FirebaseMessaging.instance;

  // Kategori aboneliği
  Future<void> subscribeToCategory(String category) async {
    await _fcm.subscribeToTopic('category_$category');
  }

  Future<void> unsubscribeFromCategory(String category) async {
    await _fcm.unsubscribeFromTopic('category_$category');
  }

  // Sessiz saatler (local)
  bool isQuietHours() {
    final now = DateTime.now();
    final hour = now.hour;
    // 21:00 - 09:00 arası sessiz
    return hour >= 21 || hour < 9;
  }
}
```

### Topic Yapısı

| Topic | Açıklama |
|-------|----------|
| `category_gundem` | Gündem haberleri |
| `category_spor` | Spor haberleri |
| `category_ekonomi` | Ekonomi haberleri |
| `breaking_news` | Son dakika (herkese) |
| `daily_digest` | Günlük özet |

## WebView Entegrasyonu

```dart
// lib/screens/source_webview.dart
import 'package:webview_flutter/webview_flutter.dart';

class SourceWebView extends StatelessWidget {
  final String url;
  final String sourceName;

  const SourceWebView({required this.url, required this.sourceName});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(sourceName),
        actions: [
          IconButton(
            icon: Icon(Icons.open_in_browser),
            onPressed: () => launchUrl(Uri.parse(url)),
          ),
        ],
      ),
      body: WebViewWidget(
        controller: WebViewController()
          ..loadRequest(Uri.parse(url))
          ..setJavaScriptMode(JavaScriptMode.unrestricted),
      ),
    );
  }
}
```

## Reklam Entegrasyonu (AdMob)

```dart
// Reklam yerleri
// 1. Ana sayfa - Liste arası (her 5 haberde 1)
// 2. Haber detay - Alt kısım
// 3. Kategori geçişlerinde - Interstitial (nadiren)

// Premium kullanıcılar için reklam kontrolü
bool showAds = !userIsPremium;
```

## Kullanıcı Ayarları

```dart
class UserSettings {
  // Bildirim tercihleri
  List<String> subscribedCategories;
  bool quietHoursEnabled;
  int quietHoursStart; // 21 (default)
  int quietHoursEnd;   // 9 (default)
  int maxDailyNotifications; // 5 (default)

  // Premium
  bool isPremium;
  DateTime? premiumExpiry;
}
```

## Güncelleme Stratejisi

1. **App Launch**: Manifest kontrol et → değişiklik varsa kategori verilerini güncelle
2. **Pull to Refresh**: İlgili kategoriyi yeniden çek
3. **Background Fetch**: Her 30 dk'da bir manifest kontrol (iOS/Android)
4. **Push Notification**: Bildirime tıklayınca ilgili haberi göster
