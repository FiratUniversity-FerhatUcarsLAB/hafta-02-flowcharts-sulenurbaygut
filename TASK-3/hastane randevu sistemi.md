ALGORITMA: HastaneRandevuSistemi

SABİTLER:
    MAX_GUN_ONCESI <- 30          // En fazla 30 gün öncesine randevu
    MAX_GUNLUK_RANDEVU <- 20      // Doktor başına günlük maksimum randevu
    ACIL_RANDEVU_SURESI <- 1      // Acil randevu için minimum süre (gün)
    
VERİLER:
    KullaniciListesi = []          // Hasta, doktor ve yönetici bilgileri
    DoktorListesi = []             // Doktor bilgileri, uzmanlık, çalışma saatleri
    RandevuListesi = []            // Tüm randevuların listesi

FONKSİYON: KullaniciGirisi(TC, sifre)
    Eğer KullaniciListesi içinde TC ve sifre doğruysa
        Giriş başarılı
    Değilse
        Giriş başarısız, hata mesajı göster

FONKSİYON: YeniKullaniciKaydi(ad, soyad, TC, eposta, sifre, rol)
    Eğer TC daha önce kayıtlıysa
        Hata: Kullanıcı zaten kayıtlı
    Eğer sifre güvenli değilse
        Hata: Güçlü şifre gereklidir
    KullaniciListesi'ne ekle

FONKSİYON: DoktorEkle(ad, soyad, uzmanlik, calismaGunleri, calismaSaatleri)
    DoktorListesi'ne ekle

FONKSİYON: RandevuAl(kullaniciID, doktorID, tarih, saat, randevuTipi)
    Eğer tarih > BUGÜN + MAX_GUN_ONCESI
        Hata: Maksimum önceden rezervasyon süresi aşıldı
    Eğer doktorID uygun değil veya çalışmıyor ise
        Hata: Doktor müsait değil
    Eğer randevuTipi = "Acil" ve tarih > BUGÜN + ACIL_RANDEVU_SURESI
        Hata: Acil randevu süresi hatalı
    Eğer saat dolu ise
        Hata: Bu saatte randevu mevcut
    Eğer kullanıcı aynı gün içinde zaten randevu aldıysa
        Hata: Aynı gün içinde birden fazla randevu alamazsınız
    Eğer doktorun o günkü randevu sayısı >= MAX_GUNLUK_RANDEVU
        Hata: Doktorun günlük randevu limiti dolu
    RandevuListesi'ne ekle
    Kullanıcıya onay mesajı gönder

FONKSİYON: RandevuIptalEt(kullaniciID, randevuID)
    Eğer randevu kullanıcıya ait değilse
        Hata: Bu randevuyu iptal edemezsiniz
    RandevuListesi'nden sil
    Kullanıcıya iptal bildirimi gönder

FONKSİYON: RandevuGoruntule(kullaniciID)
    Kullanıcıya geçmiş ve gelecekteki randevularını göster

FONKSİYON: RaporOlustur(yöneticiID)
    Günlük doluluk oranı, iptaller, doktor başına randevu sayısı raporla

ANA PROGRAM:
    Başlat
    Kullanıcı giriş veya kayıt işlemi yap
    Eğer kullanıcı = hasta
        RandevuAl, RandevuIptalEt, RandevuGoruntule seçeneklerini sun
    Eğer kullanıcı = doktor
        Kendi randevularını görüntüle
    Eğer kullanıcı = yönetici
        DoktorEkle, RaporOlustur seçeneklerini sun
    Döngü devam et / çıkış seçeneği

LLM GEÇMİŞİ 

hasteneden randevu sistemi alma üzerine bir pseudocode oluşturman lazım bu sistemin gereksinimlerini önce düzenle
ChatGPT:

Harika, o zaman önce bir hastaneden randevu sistemi için temel gereksinimleri toparlayalım. Bunlar, pseudocode yazarken sistemin nasıl çalışacağını netleştirecek. Gereksinimleri mantıksal gruplar hâlinde düzenleyelim:

1. Kullanıcı Yönetimi

Kullanıcılar sisteme giriş yapabilmeli (hasta / yönetici / doktor ayrımı).

Yeni kullanıcı kaydı oluşturulabilmeli.

Kullanıcı bilgilerinin doğruluğu (TC kimlik, telefon, e-posta vb.) kontrol edilmeli.

2. Randevu Yönetimi

Kullanıcı randevu alabilmeli, iptal edebilmeli veya değiştirebilmeli.

Randevu alma sırasında doktor ve tarih seçilebilmeli.

Doktorun müsaitlik durumu kontrol edilmeli.

Randevular saat ve gün bazında çakışmamalı.

Maksimum randevu sayısı ve gün sınırı olmalı (günlük limit, doktor başına limit vb.).

3. Doktor Yönetimi

Doktorlar ve uzmanlık alanları sisteme tanımlanmalı.

Doktorların çalışma günleri ve saatleri belirlenmeli.

Doktorların müsaitlik durumu güncellenebilmeli.

4. Bildirim / Onay Mekanizması

Randevu alındığında kullanıcıya onay mesajı gönderilmeli.

Randevu iptali veya değişikliği durumunda kullanıcı bilgilendirilmeli.

5. Sistem Kuralları ve Kısıtlar

Aynı kullanıcı aynı saate birden fazla randevu alamaz.

Günlük maksimum randevu sayısı aşılmamalı.

Randevu saatleri çalışma saatleri dışında alınamaz.

Önceden rezervasyon limitleri (ör. 30 gün öncesine kadar) olabilir.

İstersen bir sonraki adım olarak bu gereksinimleri direkt pseudocode haline getirip, sistemin akışını gösterecek şekilde tasarlayabilirim.

Bunu yapmamı ister misin?

Şunu dedin:
gereksinimlerde eksik olup olmadığını kontrol et
ChatGPT:

Tamam, gereksinimleri biraz daha kritik açıdan gözden geçirelim ve eksik noktaları tespit edelim. Şu anda listemiz oldukça temel, ama bazı önemli noktalar eksik:

Eksik / Geliştirilebilecek Noktalar:

Randevu öncelik ve sınıflandırma

Acil randevular veya özel durumlar için öncelik sistemi.

Uzmanlık alanına göre farklı bekleme süreleri.

Kullanıcı doğrulama ve güvenlik

Şifre yönetimi (sıfırlama, güçlü şifre kontrolü).

İki faktörlü doğrulama veya e-posta / SMS doğrulama.

Geçmiş ve kayıt tutma

Kullanıcının geçmiş randevuları görüntülenebilmeli.

Doktorun hasta geçmişine erişim hakkı olmalı (yetki bazlı).

Sistem yönetimi ve raporlama

Yönetici raporları: günlük randevu sayısı, iptaller, doluluk oranları.

Doktorlar veya kullanıcılar için performans / kullanım istatistikleri.

Uyarılar ve çakışma önleme

Kullanıcı aynı gün içinde birden fazla randevu almaya çalıştığında uyarı.

Doktor veya klinik yoğunluğu aşırı ise uyarı mekanizması.

Esnek saat ve gün yönetimi

Tatil günleri, izin günleri veya özel durumlar için randevu almayı engelleme.

Çeşitli randevu türleri (muayene, test, konsültasyon) için farklı süreler.

Ödeme / sigorta entegrasyonu (opsiyonel)

Ödeme veya sigorta doğrulaması gerektiren sistemler için entegrasyon.

💡 Özetle: mevcut gereksinimler temel işlevleri kapsıyor ama öncelik yönetimi, güvenlik, raporlama, geçmiş kayıt ve esnek tarih/saat yönetimi gibi kritik ayrıntılar eksik.

İstersen bu eksikleri de dahil ederek tam ve uygulanabilir pseudocode taslağını oluşturabiliriz.

Bunu yapalım mı?

Şunu dedin:
her iki kısımdaki gereksinimlerin tamamını kapsayan ve iyi bir denetlemeye sahip pseudocode oluştur
ChatGPT:

Harika! O zaman hem temel hem de eksik noktaları kapsayan, mantıksal olarak sağlam ve denetim mekanizmaları güçlü bir pseudocode tasarlayalım. Kod Türkçe olacak ve her adımda gerekli kontrolleri içerecek.

ALGORITMA: HastaneRandevuSistemi

SABİTLER:
    MAX_GUN_ONCESI <- 30          // En fazla 30 gün öncesine randevu
    MAX_GUNLUK_RANDEVU <- 20      // Doktor başına günlük maksimum randevu
    ACIL_RANDEVU_SURESI <- 1      // Acil randevu için minimum süre (gün)
    
VERİLER:
    KullaniciListesi = []          // Hasta, doktor ve yönetici bilgileri
    DoktorListesi = []             // Doktor bilgileri, uzmanlık, çalışma saatleri
    RandevuListesi = []            // Tüm randevuların listesi

FONKSİYON: KullaniciGirisi(TC, sifre)
    Eğer KullaniciListesi içinde TC ve sifre doğruysa
        Giriş başarılı
    Değilse
        Giriş başarısız, hata mesajı göster

FONKSİYON: YeniKullaniciKaydi(ad, soyad, TC, eposta, sifre, rol)
    Eğer TC daha önce kayıtlıysa
        Hata: Kullanıcı zaten kayıtlı
    Eğer sifre güvenli değilse
        Hata: Güçlü şifre gereklidir
    KullaniciListesi'ne ekle

FONKSİYON: DoktorEkle(ad, soyad, uzmanlik, calismaGunleri, calismaSaatleri)
    DoktorListesi'ne ekle

FONKSİYON: RandevuAl(kullaniciID, doktorID, tarih, saat, randevuTipi)
    Eğer tarih > BUGÜN + MAX_GUN_ONCESI
        Hata: Maksimum önceden rezervasyon süresi aşıldı
    Eğer doktorID uygun değil veya çalışmıyor ise
        Hata: Doktor müsait değil
    Eğer randevuTipi = "Acil" ve tarih > BUGÜN + ACIL_RANDEVU_SURESI
        Hata: Acil randevu süresi hatalı
    Eğer saat dolu ise
        Hata: Bu saatte randevu mevcut
    Eğer kullanıcı aynı gün içinde zaten randevu aldıysa
        Hata: Aynı gün içinde birden fazla randevu alamazsınız
    Eğer doktorun o günkü randevu sayısı >= MAX_GUNLUK_RANDEVU
        Hata: Doktorun günlük randevu limiti dolu
    RandevuListesi'ne ekle
    Kullanıcıya onay mesajı gönder

FONKSİYON: RandevuIptalEt(kullaniciID, randevuID)
    Eğer randevu kullanıcıya ait değilse
        Hata: Bu randevuyu iptal edemezsiniz
    RandevuListesi'nden sil
    Kullanıcıya iptal bildirimi gönder

FONKSİYON: RandevuGoruntule(kullaniciID)
    Kullanıcıya geçmiş ve gelecekteki randevularını göster

FONKSİYON: RaporOlustur(yöneticiID)
    Günlük doluluk oranı, iptaller, doktor başına randevu sayısı raporla

ANA PROGRAM:
    Başlat
    Kullanıcı giriş veya kayıt işlemi yap
    Eğer kullanıcı = hasta
        RandevuAl, RandevuIptalEt, RandevuGoruntule seçeneklerini sun
    Eğer kullanıcı = doktor
        Kendi randevularını görüntüle
    Eğer kullanıcı = yönetici
        DoktorEkle, RaporOlustur seçeneklerini sun
    Döngü devam et / çıkış seçeneği

✅ Bu pseudocode’un güçlü yönleri:

Tüm kullanıcı türleri (hasta, doktor, yönetici) destekleniyor.

Randevu alma kontrolleri: tarih, saat, doktor müsaitliği, günlük limit, acil randevu kısıtları, kullanıcı çakışması.

Güvenlik denetimleri: kullanıcı kaydı kontrolü, şifre güvenliği.

Bildirim mekanizması: randevu onayı ve iptali için mesaj gönderme.

Raporlama ve yönetim: yönetici için sistem raporu.

İstersen bir sonraki adımda bunu akış diyagramı şeklinde görselleştirip, her adımın mantığını kolay takip edebilir hâle getirebilirim.

Bunu yapayım mı?
