# Beş Uygulama için Gizlilik ve Kullanım Koşulları Uyum Raporu

**Tarih:** 24 Eylül 2026

**Kapsam:** Petopia, Shadow Prompts, Grimoire (Wicca/Witchcraft), ColdLog ve DayTracker uygulama kodlarının mevcut Privacy Policy ve Terms metinleriyle karşılaştırılması. Bu rapor uygulama kodu ve ürün beyanı incelemesidir; resmi hukuki görüş değildir.

## Öncelikli Sonuçlar

### P0 — Analitik ve diagnostik için izin ve kapatma kontrolü

Beş uygulamanın tümünde Aptabase, derlemede uygulama anahtarı varsa başlatılıyor. Mevcut Privacy metinleri de uygulama içinde analitik/diagnostik kapatma kontrolü olmadığını belirtiyor. Sentry bulunan derlemelerde hata ve performans verisi ayrıca gönderilebiliyor.

Apple, anonim olsa bile kullanım verisi toplanırken kullanıcı onayı ve onayı kolayca geri çekme yolu istiyor; ücretli özellikler bu onaya bağlanamaz. Her uygulamada ilk veri gönderiminden önce açık bir seçim gösterin, tercihi cihazda saklayın, Settings içinde kapatma/geri çekme kontrolü sağlayın ve hem Aptabase hem de ilgili Sentry gönderimlerini bu tercihe göre durdurun. Reddetmek uygulamanın veya satın alınmış özelliklerin kullanımını engellememeli. Bu, [App Review Guidelines 5.1.1(i)–(iv)](https://developer.apple.com/app-store/review/guidelines/uk/) ile uyum için en önemli uygulama değişikliğidir.

İlgili başlatma noktaları: Petopia `lib/analytics.ts:132–165`; Shadow Prompts `src/lib/analytics.ts:92–123`; Grimoire `src/lib/analytics.ts:113–135`; ColdLog `src/app/_layout.tsx:38`; DayTracker `src/lib/analytics.ts:43–65`.

### P1 — Privacy ve Terms bağlantılarını Settings içine koyun

Apple Privacy Policy bağlantısının App Store metadatasında ve uygulama içinde kolay erişilir olmasını istiyor. Kod incelemesinde Petopia, Shadow Prompts, Grimoire ve ColdLog bağlantıları paywall/abonelik ekranında bulundu; Settings içinde doğrudan hukuk bağlantısı tespit edilmedi. Bu uygulamalara Settings > Privacy & Legal gibi kalıcı bir bölüm ekleyip hem Privacy Policy hem Terms bağlantısı koyun. DayTracker’da iki bağlantı Settings içinde zaten var (`src/app/(tabs)/settings.tsx:508–513`).

| Uygulama | Mevcut bağlantı yeri | Öneri |
|---|---|---|
| Petopia | Paywall (`app/subscription.tsx:371–381`) | Settings’e Privacy ve Terms satırları ekleyin. |
| Shadow Prompts | Paywall (`app/subscription.tsx:376`); Settings’teki harici bağlantı App Store değerlendirme bağlantısı | Settings’e Privacy ve Terms satırları ekleyin. |
| Grimoire | Abonelik/paywall (`app/subscription.tsx:431`) | Profile/Settings’e Privacy ve Terms satırları ekleyin. |
| ColdLog | Paywall (`src/app/paywall.tsx:410`) | Settings’e Privacy ve Terms satırları ekleyin. |
| DayTracker | Settings ve paywall | Mevcut yapıyı koruyun; mağaza metadata URL’lerini doğrulayın. |

## Uygulama Bazında Yapılacaklar

### 1. Petopia

Pet profilleri, fotoğraflar, veteriner/sağlık kayıtları, bakım planları ve giderler cihazda tutuluyor. Metinler ayrıca döviz kuru isteğinde yalnızca baz para biriminin Frankfurter’a gönderildiğini ve analitik verilerin tür/kategori gibi alanlar içerebildiğini açıklıyor.

- **Fotoğraf iznini daraltın.** `components/forms/PetPhotoPicker.tsx:37–57`, kamera ve fotoğraf kütüphanesi izinlerini her iki akış için birlikte istiyor. Kullanıcı hangi işlemi seçtiyse yalnızca onun iznini isteyin; mümkünse fotoğraf seçiminde sistem picker’ını kullanın. `app.json` içindeki `expo-image-picker` yapılandırmasına Petopia fotoğrafı ekleme ve kamera amacı için açık, ayrı amaç metinleri ekleyin. Apple, mümkün olduğunda tam Photos erişimi yerine sistem picker’ını öneriyor.
- **“Clear Local Data” davranışını netleştirin.** `app/(tabs)/settings.tsx:137–170` ana veritabanını sıfırlıyor; ayrı saklanan bazı tercihler ve kurulum bilgileri kalabiliyor. Tam silme vaat edilecekse bu yerel değerleri ve ilgili dosyaları da kaldırın; değilse düğme adını ana kayıtları sıfırladığını belirtecek şekilde daraltıp mevcut açıklamayı koruyun.
- **Settings’e Privacy ve Terms bağlantısı ekleyin.** Şu an paywall bağlantı işleyicileri var; kullanıcıların paywall’a gitmeden ulaşabileceği bağlantı bulunmalı.
- **Analitik kontrolünü ekleyin.** Aptabase/Sentry için ortak P0 maddesini uygulayın.

### 2. Shadow Prompts

Günlük yazıları ve ses kayıtları yerel SQLite/veri dosyalarında tutuluyor; uygulamanın kendi akışı bunları analitik servislere göndermiyor. Mikrofon amaç metni sesli günlük kaydı için açıkça tanımlanmış.

- **Silme eylemini gerçek silmeye dönüştürün.** `src/data/actions/mutations/journal-entries.mutation.ts:72–95` arşivden gizlenen yazıyı `isDeleted` ile işaretliyor; metin SQLite satırında kalıyor. Mevcut Privacy ve Terms bunu dürüstçe açıklasa da “sil” beklentisiyle uyumsuz. Kullanıcı silme istediğinde metin satırını da kaldırın; ayrıca tüm yerel günlük ve ses dosyalarını temizleyen bir seçenek ekleyin. Dosya silme ve veritabanı hatalarını kullanıcıya bildirin.
- **Hukuk bağlantılarını Settings’e ekleyin.** Şu an Privacy/Terms bağlantıları paywall’da; Settings’teki harici bağlantı App Store değerlendirmesi içindir.
- **Analitik kontrolünü ekleyin.** Aptabase/Sentry olayları günlük metnini ve ses kaydını içermese de otomatik başlatılıyor. Ortak P0 maddesini uygulayın.

### 3. Grimoire (Wicca/Witchcraft)

Günlük, tarot, ritüel ve profil verileri yerel veritabanında tutuluyor. Horoskop istendiğinde seçilen burç Free Horoscope API’ye gönderiliyor; Privacy metni bunu açıklıyor. Tekil günlük girdisinin silme kodu veritabanı satırını gerçekten kaldırıyor (`src/db/repositories/my-space-repository.ts:158–163`).

- **Aptabase olaylarını küçültün.** Ritüel ekranı ve tamamlama akışlarında `ritual_id` gönderiliyor (`app/ritual/[slug].tsx:67, 114, 333`; `app/ritual/complete.tsx:120, 150, 198`). Mevcut Privacy metni burç değişikliği olayından söz ediyor; ritüel kimliklerini açıkça saymıyor. Ruhsal uygulama kullanımını açığa çıkarabilecek bu kimlikleri analitikten çıkarın. Tutacaksanız veri türünü ve amacını Privacy metninde açıkça belirtin ve analitik onayı alın.
- **Settings’e Privacy ve Terms bağlantısı ekleyin.** Mevcut harici hukuk bağlantısı abonelik/paywall ekranında.
- **Tüm yerel kayıtları silme seçeneğini değerlendirin.** Tekil günlük silme var; günlükler, tarot okumaları, ritüel tamamlamaları, favoriler ve önbellekleri tek işlemde kaldıran yerel veri temizleme kontrolü tespit edilmedi. Uygulama hesap/cloud kullanmadığından bu Apple’ın hesap silme şartıyla aynı konu değildir; kullanıcı kontrolünü güçlendiren bir iyileştirmedir.
- **Analitik kontrolünü ekleyin.** Aptabase derleme anahtarıyla otomatik başlatılıyor; ortak P0 maddesini uygulayın.

### 4. ColdLog

Oturumlar ve notlar yerel tutuluyor. HealthKit akışı HRV ve kalp atış hızı verilerini okumak için izin istiyor; kod Apple Health’e yazma izni istemiyor (`src/features/health/hrv.ts:56–88`). HealthKit amaç metni de okunan veri ve kullanım amacını açıkça belirtiyor (`app.config.ts:75–78`).

- **Onboarding’deki kesin sağlık vaadini kaldırın veya kanıtlayıp niteleyin.** `src/app/onboarding/goal.tsx:73–74`, “haftada 11 dakika ... maksimum metabolik ve dayanıklılık faydası” vaat ediyor. Terms ise fayda/sonuç garantisi vermediğini ve uygulamanın tıbbi değerlendirme olmadığını söylüyor. Metni nötr bir hedef takibi anlatımına çevirin; aksi halde ürün ekranı ve Terms birbiriyle çelişiyor.
- **Breathwork başlamadan güvenlik uyarısı gösterin.** `src/app/breathwork.tsx` ekranı nefes/hold akışına doğrudan giriyor; Terms’teki “suda veya su yakınında nefes tutma egzersizi yapmayın” ve acil yardım uyarıları başlangıç öncesi görünür değil. Devam etmeden önce uyarıyı gösterin; kullanıcıya vazgeçme seçeneği verin. Cold exposure/breathwork sağlık iddialarını da ürün içindeki kanıt ve kapsamla eşleştirin.
- **Settings’e Privacy ve Terms bağlantısı ekleyin.** Şu an paywall’da açılıyor.
- **Tüm oturumları temizleme seçeneğini değerlendirin.** Mevcut Privacy metni yalnızca tekil oturum silme olduğunu ve toplu temizleme bulunmadığını belirtiyor. Kullanıcıya oturumlar ve ekli HRV görüntülerini birlikte kaldırma kontrolü eklemek veri silme beklentisini güçlendirir.
- **Analitik kontrolünü ekleyin.** Aptabase, HealthKit izni ve HRV örneği mevcut olup olmadığı gibi durumları; oturum özelliklerinin bazı özetlerini gönderiyor. Metin sayısal HRV/kalp ölçümü gönderilmediğini açıklıyor. Bu sınırı koruyup ortak P0 maddesini uygulayın.

### 5. DayTracker

Etkinlikler cihazda tutuluyor; takvim/rehber içe aktarma kullanıcı başlatınca iOS izni istiyor ve kullanıcı seçtiği kayıtları içe alıyor. `app.json` takvim ve doğum günü içe aktarma amaçlarını açıklayan ayrı izin metinleri içeriyor. JSON yedek kullanıcı tarafından oluşturulup paylaşım ekranından dışarı aktarılıyor.

- **Widget paylaşımını Privacy metnine ekleyin.** `src/lib/widget.ts:6–35` etkinlik adı, emoji, renk, tarih, tekrar kuralı ile tema, glow ve dil ayarlarını iOS widget köprüsüne veriyor; widget bunları cihazdaki App Group/UserDefaults alanında kullanıyor (`plugins/withDaysLeftWidget.js:9–13`). Bu veri sunucuya gitmiyor; yine de ana ekrandaki widget’ın etkinlik verisini cihaz içinde kullandığını açıklayın.
- **Kişiler picker’ını değerlendirin.** Kod yalnızca ad/doğum günü alanlarını içe aktarsa da adayları bulmak için Contacts izniyle kayıtları tarıyor. Birden fazla doğum günü seçme ihtiyacını koruyarak Apple’ın mümkün olduğunda Contacts picker kullanma önerisini değerlendirin; tam erişim gerekiyorsa mevcut just-in-time izin akışını ve veri minimizasyonunu koruyun.
- **Tüm yerel etkinlikleri temizleme seçeneğini değerlendirin.** Kod tekil etkinlik silme, JSON dışa/içe aktarma sağlıyor; Settings’te tüm etkinlikleri ve bunların yerel bildirimlerini temizleyen bir işlem tespit edilmedi. Bu, yerel veri kontrolünü artırır; Privacy metninde mevcut davranışı doğru tarif edin.
- **Settings’teki hukuk bağlantılarını koruyun ve analitik kontrolünü ekleyin.** Privacy/Terms satırları zaten Settings’te. Aptabase ise anahtarlı derlemede otomatik başlıyor; ortak P0 maddesi burada da geçerli.

## Yayın Öncesi Ortak Kontrol Listesi

1. Her uygulamanın App Store Connect Privacy Policy bağlantısını ve varsa Terms/EULA bağlantısını doğru yayımlanmış HTML sayfasına doğrulayın.
2. App Privacy Details etiketlerini yalnızca uygulama koduna göre değil Aptabase, Sentry, RevenueCat ve kullanılan diğer SDK/API’lerin gerçek veri akışına göre güncelleyin. Apple üçüncü taraf SDK’ların veri uygulamalarından da geliştiriciyi sorumlu tutuyor: [Third-party SDK requirements](https://developer.apple.com/support/third-party-sdk-requirements/). Gerekli SDK privacy manifest ve imza koşullarını da arşiv/build raporunda kontrol edin.
3. App Store paywall’larında mağazanın sunduğu ürünle eşleşen fiyat, dönem, otomatik yenileme/tek seferlik satın alma ve satın alımları geri yükleme bilgisini gösterin; Terms bağlantısını satın alma onayından önce erişilebilir tutun.
4. Her değişiklikten sonra ilgili Privacy ve Terms metinlerini yeni gerçek davranışa göre güncelleyin. Özellikle veri alanları, saklama/silme davranışı ve SDK tercihi değişikliklerini metinlerde eşzamanlı tutun.

## İlgili Güncel Metinler

- [Petopia Privacy](privacy.html) · [Terms](terms.html)
- [Shadow Prompts Privacy](privacy-shadow.html) · [Terms](terms-shadow.html)
- [Grimoire Privacy](privacy-grimoire.html) · [Terms](terms-grimoire.html)
- [ColdLog Privacy](privacy-coldlog.html) · [Terms](terms-coldlog.html)
- [DayTracker Privacy](privacy-days-left.html) · [Terms](terms-days-left.html)
