# Dişçi Bul - Mobil Uygulama (Flutter)

Bu depo, Dişçi Bul mobil uygulamasının Flutter tabanlı istemci kodunu içermektedir. Proje, modern mobil geliştirme prensiplerini ve güçlü bir backend entegrasyonunu bir araya getirerek kullanıcı dostu bir deneyim sunmayı hedeflemektedir.

## Mimari Genel Bakış

- **Dil:** Dart
- **Framework:** Flutter
- **State Yönetimi:** Riverpod (StateNotifier)
- **Backend Entegrasyonu:** Supabase (REST API ve Realtime)
- **Yerel Depolama:** `shared_preferences`
- **Telemetri:** `AppEventService` ile Supabase'e event loglama

## Özellikler

- 🏥 **Hastane Dizini:** Tüm hastaneleri görüntüleme, filtreleme ve arama
- 👨‍⚕️ **Doktor Dizini:** Doktorları görüntüleme, filtreleme ve arama
- 📅 **Randevu Yönetimi:** Randevu oluşturma, görüntüleme, iptal etme
- ⭐ **Değerlendirme:** Hastane ve doktorlara yorum ve puan verme
- 👤 **Kullanıcı Profili:** Profil görüntüleme ve düzenleme
- 🔔 **Bildirimler:** Randevu hatırlatmaları ve sistem bildirimleri
- 🌍 **Çoklu Dil Desteği:** Türkçe ve İngilizce dil seçenekleri
- 📍 **Konum Tabanlı Arama:** Yakındaki hastaneleri bulma

## Kurulum ve Çalıştırma

### 1. Ön Gereksinimler

- Flutter SDK (stable channel)
- Dart SDK
- Git
- Bir Supabase projesi (URL ve API anahtarları gereklidir)

### 2. Depoyu Klonlama

```bash
git clone https://github.com/CemBlt/dent-mobile.git
cd dent-mobile
```

### 3. Bağımlılıkları Yükleme

```bash
flutter pub get
```

### 4. Ortam Değişkenlerini Yapılandırma

`assets/env.client` dosyasını oluşturun ve içine Supabase `SUPABASE_URL` ve `SUPABASE_ANON_KEY` değerlerini ekleyin:

```
SUPABASE_URL=https://your-supabase-url.supabase.co
SUPABASE_ANON_KEY=your-supabase-anon-key
```

**Supabase URL ve Key Nasıl Bulunur?**

1. [Supabase Dashboard](https://app.supabase.com/)'a giriş yapın
2. Projenizi seçin
3. Sol menüden **Settings** > **API** seçeneğine tıklayın
4. Şu bilgileri kopyalayın:
   - **Project URL** → `SUPABASE_URL` olarak kullanın
   - **anon public** key → `SUPABASE_ANON_KEY` olarak kullanın

### 5. Uygulamayı Çalıştırma

```bash
flutter run
```

## Kullanıcı Kayıt & E-posta Doğrulaması

Uygulama, kullanıcı kayıt işlemlerinde e-posta doğrulaması gerektirir:

1. **Supabase Dashboard Ayarları:**
   - Supabase Dashboard → **Authentication → Providers → Email** bölümünde "Confirm email" seçeneğini açık tutun
   - (Opsiyonel) `SITE_URL` ve `EMAIL_REDIRECT_URL` alanlarını Flutter uygulamasının desteklediği deep-link'e yönlendirin

2. **Kayıt Akışı:**
   - Kullanıcı formu doldurur, Supabase `signUp` çağrısı yapılır
   - Kayıt sonrası uygulama doğrulama diyalogu gösterir ve Login ekranına geri döner
   - Kullanıcı e-postasını doğrulamadan devam edemez
   - Login ekranı Supabase'ten gelen `email not confirmed` hatalarını yakalar ve kullanıcıyı bilgilendirir
   - Kullanıcı e-postasını doğrulayıp giriş yaptığında, login ekranına verilen `onLoginSuccess` callback sayesinde başlangıçta gitmek istediği akış (ör: randevu oluşturma) tekrar açılır

## Testler

Proje, Flutter Riverpod controller'larını ve utility fonksiyonlarını test etmek için unit testler içermektedir:

```bash
flutter test
```

## Proje Yapısı

```
dent-mobile/
├── lib/
│   ├── config/           # Yapılandırma dosyaları
│   ├── models/           # Veri modelleri
│   ├── providers/        # Riverpod state yönetimi
│   ├── screens/          # UI ekranları
│   ├── services/         # Backend servisleri
│   ├── theme/            # Tema ve stil tanımları
│   ├── utils/            # Yardımcı fonksiyonlar
│   └── widgets/          # Yeniden kullanılabilir widget'lar
├── test/                 # Unit testler
├── assets/               # Görseller ve yapılandırma dosyaları
├── android/              # Android platform dosyaları
├── ios/                  # iOS platform dosyaları
└── pubspec.yaml          # Bağımlılıklar
```

## State Yönetimi (Riverpod)

Flutter uygulamasında tüm state yönetimi [Riverpod](https://riverpod.dev/) kullanılarak yapılmıştır. Her ana ekran ve karmaşık akış (randevu oluşturma, randevu listesi, profil, ayarlar vb.) kendi `StateNotifier` ve `State` çifti ile yönetilir. Bu yaklaşım:

- **Tek Kaynak:** Her veri parçasının tek bir sahibi olmasını sağlar
- **Test Edilebilirlik:** İş mantığını UI'dan ayırarak kolay test yazımına olanak tanır
- **Performans:** Yalnızca değişen widget'ların yeniden build edilmesini sağlar

### Provider'lar

- `createAppointmentControllerProvider` - Randevu oluşturma akışı
- `appointmentsControllerProvider` - Randevu listesi ve yönetimi
- `hospitalsDirectoryControllerProvider` - Hastane dizini
- `doctorsDirectoryControllerProvider` - Doktor dizini
- `profileControllerProvider` - Kullanıcı profili
- `accountSettingsControllerProvider` - Hesap ayarları
- `authControllerProvider` - Kimlik doğrulama (login/register)
- `notificationSettingsControllerProvider` - Bildirim ayarları
- `notificationsControllerProvider` - Bildirim listesi
- `languageSettingsControllerProvider` - Dil ayarları

## CI/CD

Proje, her `push` ve `pull_request` olayında otomatik olarak testleri çalıştıran bir GitHub Actions workflow'una sahiptir. Workflow, `/.github/workflows/ci.yml` dosyasında tanımlanmıştır.

## Katkıda Bulunma

Katkılarınız memnuniyetle karşılanır! Lütfen bir özellik eklemeden veya hata düzeltmeden önce mevcut kod stilini ve mimari prensiplerini inceleyin. Herhangi bir değişiklik için bir `pull request` açmadan önce ilgili testleri yazmayı ve CI'ın yeşil geçtiğinden emin olmayı unutmayın.

## İlgili Projeler

- **Yönetim Paneli:** [dent-panel](https://github.com/CemBlt/dent-panel) - Django tabanlı yönetim paneli

## Lisans

Bu proje özel bir projedir.
