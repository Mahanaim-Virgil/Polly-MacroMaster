# Polly MacroMaster

Klavye ve fare hareketlerini kaydedip tekrar oynatan Windows masaüstü uygulaması.
C# / .NET 8 ve WinForms ile yazıldı; sistem genelinde girdi yakalama ve simülasyon için Win32 API kullanır.

<!-- Ekran görüntüsü: docs/screenshot.png eklenince aşağıdaki satırın yorumunu kaldır -->
<!-- ![Polly ana ekranı](docs/screenshot.png) -->

## Özellikler

- **Kayıt ve oynatma:** Klavye ve fare olayları zamanlamasıyla birlikte kaydedilir ve tekrar oynatılır.
- **Oynatma ayarları:** Hız çarpanı, tekrar sayısı, sonsuz döngü, başlangıç gecikmesi, göreli koordinatlar,
  hata durumunda durma ve gerçek girdi göndermeyen simülasyon modu.
- **Makro sadeleştirme:** Gereksiz fare hareketlerini ayıklar, bekleme sürelerini korur ve değişiklikleri
  uygulamadan önce önizleme olarak gösterir.
- **Makro kütüphanesi:** Kayıtlar JSON veya XML olarak saklanır, filtrelenip yeniden adlandırılabilir.
- **Olay düzenleme:** Kaydedilen olaylar listede filtrelenip tek tek düzenlenebilir; fare yolu harita üzerinde önizlenir.
- **Oturum raporu:** Bir kayıt için metin veya HTML analiz raporu üretir.
- **Global kısayollar** (değiştirilebilir):

  | Tuş | İşlev |
  | --- | --- |
  | F8 | Kaydı başlat / durdur |
  | F9 | Oynatmayı başlat / durdur |
  | F10 | Durdur |
  | F12 | Kısayol ayarları |

## Mimari

```
MacroMaster.Domain          Modeller ve enum'lar (MacroSession, MacroEvent, PlaybackSettings)
MacroMaster.Application     Arayüzler ve iş mantığı (kayıt, oynatma, sadeleştirme, uygulama durumu)
MacroMaster.Infrastructure  Win32 hook'ları, P/Invoke ile girdi gönderme, dosya saklama, loglama
MacroMaster.WinForms        Arayüz, tema ve bağımlılıkların bağlandığı composition root
MacroMaster.Tests           xUnit testleri
```

Neden böyle: Windows'a özgü kod (klavye/fare hook'ları, `SendInput`) Infrastructure katmanında ve
Application'daki arayüzlerin arkasında duruyor. Bu sayede kayıt, oynatma ve sadeleştirme mantığı gerçek
klavye ve fare olmadan, sahte (fake) girdi kaynaklarıyla test edilebiliyor.

Diğer kararlar:
- **Atomik dosya yazma:** Makro ve ayar dosyaları önce geçici dosyaya yazılıp sonra yerine taşınır;
  yazma yarıda kesilirse eski dosya bozulmaz.
- **Uyarılar hata sayılır:** `Directory.Build.props` içinde .NET analizörleri açık ve `TreatWarningsAsErrors` etkin.

## Çalıştırma

Gereksinim: Windows ve .NET 8 SDK.

```bash
dotnet run --project MacroMaster.WinForms
```

Testler:

```bash
dotnet test MacroMaster.Tests
```

Ayarlar, makrolar ve loglar `%LOCALAPPDATA%` altındaki uygulama klasöründe tutulur.

## Kurulum dosyası

`installer/PollyInstaller.iss`, [Inno Setup](https://jrsoftware.org/isinfo.php) ile `Polly_Setup_v1.0.0.exe`
kurulum dosyasını üretir. Önce uygulamayı yayınlayın (`dotnet publish`), sonra betiği Inno Setup ile derleyin.
