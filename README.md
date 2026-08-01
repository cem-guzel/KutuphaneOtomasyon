# Kütüphane Otomasyon Sistemi

ASP.NET Core MVC ile geliştirilmiş, uçtan uca bir kütüphane yönetim sistemi. Kitap, üye ve ödünç/iade süreçlerinin yönetiminin yanı sıra, dış API entegrasyonları ve yapay zeka destekli öneri/risk analizi ile zenginleştirilmiş bir otomasyon platformu.

## Özellikler

- **Kitap / Üye / Ödünç Yönetimi (CRUD)** — Book, Member ve Borrow varlıkları üzerinden tam kapsamlı ekleme, güncelleme, silme ve listeleme akışları
- **Dinamik Filtreleme Modülü** — herhangi bir DataTable'a kopyala-yapıştır ile eklenebilen, sütun tipini (tarih/sayı/metin/select) otomatik algılayan, önbellekleme ile 5.000+ satırda performanslı çalışan bir filtreleme modali
- **Google Books API Entegrasyonu** — başlık veya ISBN ile arama yapılarak kitap bilgilerinin (yazar, kapak, açıklama, ISBN vb.) tek tıkla forma aktarılması
- **Barkod / ISBN Tarama** — ZXing-js/Quagga2 tabanlı, kamera ve dosya yükleme destekli barkod okuma; mobil cihazlarla ngrok üzerinden HTTPS tünelleme ve QR kod ile hızlı erişim
- **ISBN Çoklu Kaynak Arama** — Google Books API, yerel ISBN kaynağı ve sistem veritabanını sırayla sorgulayarak en zengin metadata'yı döndüren katmanlı arama mantığı
- **Yapay Zeka Destekli Öneri ve Risk Motoru** — hata toleranslı hibrit mimari: `IAiAssistant` arayüzü üzerinden çalışan, AI erişilemediğinde kurala dayalı `NullAiAssistant` katmanına düşen bir yapı
  - Kitap önerisi: kategori/yazar benzerliği, popülerlik ve stok durumuna göre yerel re-rank ile desteklenmiş LLM önerileri
  - Üye risk analizi: gecikme geçmişi, aktif ödünçler ve zamanında iade oranına dayalı sayısal risk skoru + LLM tarafından üretilen açıklayıcı özet
  - Sağlayıcı esnekliği: OpenAI (GPT-5 / GPT-4o) ile başlanıp, maliyet ve sürdürülebilirlik nedeniyle self-hosted **Ollama**'ya geçiş yapıldı
- **AI Log Modülü** — tüm AI çağrılarının (kaynak, model, süre, durum, hata) izlendiği, başarı oranı ve ortalama yanıt süresi gibi özet metriklerin gösterildiği bir izleme paneli
- **Kimlik Doğrulama** — Cookie Authentication tabanlı giriş/çıkış akışı, CSRF koruması ve rol bazlı yetkilendirme (`[Authorize]`)
- **Dashboard** — toplam kitap/üye/aktif ödünç/gecikme sayıları ve Chart.js ile görselleştirilmiş ödünç trendleri

## Teknoloji Yığını

- **Framework:** ASP.NET Core MVC, Entity Framework Core
- **Veritabanı:** SQL Server
- **AI:** OpenAI API (GPT-5 / GPT-4o), Ollama (self-hosted, yerel model)
- **Gerçek Zamanlı İletişim:** SignalR (mobil barkod tarama senkronizasyonu için)
- **Barkod Tarama:** ZXing-js / Quagga2
- **Frontend:** DataTables, Select2, Chart.js, Bootstrap
- **Dış Entegrasyon:** Google Books API

## Kurulum

1. Depoyu klonla:
```bash
   git clone https://github.com/cem-guzel/KutuphaneOtomasyon.git
   cd KutuphaneOtomasyon
```

2. `appsettings.json` dosyasında veritabanı bağlantısını kendi SQL Server ortamına göre düzenle:
```json
   "ConnectionStrings": {
     "DefaultConnection": "Server=localhost\\SQLEXPRESS;Database=KutuphaneOtomasyon;Trusted_Connection=True;MultipleActiveResultSets=true;TrustServerCertificate=True"
   }
```

3. Google Books ve OpenAI API anahtarlarını **.NET User Secrets** ile tanımla (bu değerler `appsettings.json` içinde tutulmaz):
```bash
   dotnet user-secrets set "GoogleBooksApiKey" "buraya-kendi-key'in"
   dotnet user-secrets set "OpenAI:ApiKey" "buraya-kendi-key'in"
```

4. Yapay zeka için varsayılan sağlayıcı **Ollama**'dır (self-hosted, ücretsiz). `appsettings.json` içindeki `AI` bölümü zaten yapılandırılmıştır:
```json
   "AI": {
     "Provider": "Ollama",
     "Ollama": {
       "BaseUrl": "http://localhost:11434",
       "Model": "llama3.1:8b-instruct-q5_K_M"
     }
   }
```
   Çalıştırmadan önce [Ollama](https://ollama.com/) kurulu olmalı ve ilgili model indirilmiş olmalıdır:
```bash
   ollama pull llama3.1:8b-instruct-q5_K_M
```
   OpenAI kullanmak istersen, `AI:Provider` değerini `"OpenAI"` olarak değiştirebilirsin.

5. Veritabanını oluştur:
```bash
   dotnet ef database update
```

6. Projeyi çalıştır:
```bash
   dotnet run
```

7. Tarayıcıda `/swagger` üzerinden API uç noktalarını incele, veya uygulamanın ana arayüzüne git.

> **Not:** Ollama kurulu değilse veya erişilemezse, sistem otomatik olarak kurala dayalı (`NullAiAssistant`) moda düşer — yapay zeka önerileri olmadan da temel işlevsellik (öneri/risk hesaplama) çalışmaya devam eder.

## Not

Bu proje şu an için otomatik test altyapısı içermemektedir; hata takibi AI Log modülü üzerinden manuel izlemeyle yapılmaktadır.