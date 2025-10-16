ALGORİTMA ÜniversiteDersKaydı

GİRİŞ: öğrenci_no

// 1. Öğrenci doğrulama
EĞER ÖğrenciAktifDeğil(öğrenci_no) OLURSA
    YAZDIR "Öğrenci aktif değil, ders kaydı yapılamaz."
    DUR
SON

EĞER GirişBilgileriGeçersiz(öğrenci_no) OLURSA
    YAZDIR "Giriş bilgileri geçersiz."
    DUR
SON

// 2. Ödeme ve borç kontrolü
EĞER BorçVar(öğrenci_no) OLURSA
    YAZDIR "Ödeme yapılmamış, ders kaydı engellendi."
    DUR
SON

// 3. Ders listesi gösterimi
mevcut_dersler <- MevcutDersleriGetir(öğrenci_no)
YAZDIR mevcut_dersler

// 4. Öğrenci ders seçimi
seçilen_dersler <- ÖğrenciSeçimiAl()

// 5. Önkoşul ve eşkoşul kontrolü
HER ders İÇİN seçilen_dersler YAP
    EĞER ÖnKoşulEksik(öğrenci_no, ders) OLURSA
        YAZDIR "Önkoşul eksik: " + ders
        SEÇİLEN_DERSLERDEN_SİL(ders)
    SON
    EĞER EşKoşulEksik(öğrenci_no, ders) OLURSA
        YAZDIR "Eşkoşul eksik: " + ders
        SEÇİLEN_DERSLERDEN_SİL(ders)
    SON
SON

// 6. Kredi ve kontenjan kontrolü
toplam_kredi <- KrediHesapla(seçilen_dersler)
EĞER toplam_kredi > MAKS_KREDİ VEYA toplam_kredi < MİN_KREDİ OLURSA
    YAZDIR "Kredi sınırları aşıldı."
    DUR
SON

HER ders İÇİN seçilen_dersler YAP
    EĞER DersDolu(ders) OLURSA
        YAZDIR "Kontenjan dolu: " + ders
        BeklemeListesineEkle(öğrenci_no, ders)
        SEÇİLEN_DERSLERDEN_SİL(ders)
    SON
SON

// 7. Zaman çakışması kontrolü
EĞER SaatÇakışmasıVar(seçilen_dersler) OLURSA
    YAZDIR "Ders saatleri çakışıyor."
    DUR
SON

// 8. Danışman onayı kontrolü
HER ders İÇİN seçilen_dersler YAP
    EĞER DanışmanOnayıGerekli(ders) VE DanışmanOnayıYok(öğrenci_no, ders) OLURSA
        YAZDIR "Danışman onayı gerekli: " + ders
        SEÇİLEN_DERSLERDEN_SİL(ders)
    SON
SON

// 9. Dersleri kaydet
DersleriKaydet(öğrenci_no, seçilen_dersler)
YAZDIR "Ders kaydı başarıyla tamamlandı."

SON


LLM GEÇMİŞİ 

üniversite ders kayıtları için gereksinimleri belirlemen lazım ama ayrıntılı şekilde
ChatGPT:

Tamam, üniversite ders kayıtları için gereksinimleri ayrıntılı şekilde sıralayalım. Burada hem teknik hem idari hem de öğrenciyi ilgilendiren noktaları kapsayacağız.

1. Öğrenci Bilgileri Gereksinimleri

Kayıtlı Olma Durumu: Öğrencinin aktif kayıtlı olması gerekir. Mezun, dondurulmuş veya kaydı silinmiş öğrenciler ders kaydı yapamaz.

Öğrenci Numarası ve Kimlik: Benzersiz öğrenci kimliği ile giriş yapılmalıdır.

Program/Bölüm Bilgisi: Öğrencinin kayıtlı olduğu program veya bölüm bilgisi sistemde mevcut olmalı.

Sınıf Düzeyi: Ders seçimi, öğrencinin sınıf düzeyi ve önkoşulları ile uyumlu olmalıdır.

2. Akademik Gereksinimler

Ön Koşul Kontrolleri:

Seçilen dersin önkoşulu varsa, öğrenci bu dersleri daha önce başarmış olmalı.

Yanlış ders seçimini önleyecek kontrol mekanizması olmalı.

Kredi Limiti:

Öğrencinin dönemlik maksimum ve minimum kredi yükü belirlenmeli (ör: 18-24 kredi).

Ders Tekrarı Kontrolü:

Daha önce başarıyla tamamlanmış derslerin tekrarı sistem tarafından uyarılmalı veya engellenmeli.

3. Zamanlama ve Çakışma Kontrolleri

Ders Çakışması:

Öğrencinin seçtiği derslerin haftalık saatleri çakışmamalı.

Hafta ve Gün Planlaması:

Ders saatleri, laboratuvar veya uygulamalı dersler sistemde doğru şekilde tanımlanmalı.

4. Teknik Sistem Gereksinimleri

Giriş Sistemi:

Öğrenci, güvenli bir şekilde sisteme giriş yapabilmeli (şifre, 2FA opsiyonel).

Ders Listesi Görüntüleme:

Derslerin adı, kodu, kredisi, kontenjan durumu ve önkoşulları listelenmeli.

Kontenjan Kontrolü:

Ders kontenjanı dolmuşsa kayıt engellenmeli veya bekleme listesine alınmalı.

Kaydetme ve Onaylama:

Öğrenci seçimlerini kaydedebilmeli ve kayıt tamamlandıktan sonra onay almalı.

Geri Alma/Değişiklik:

Belirli tarihler arasında ders ekleme, bırakma veya değiştirme yapılabilmeli.

5. Finansal Gereksinimler

Ödeme Kontrolü:

Öğrenci ücretini ödememişse veya borcu varsa ders kaydı engellenebilir.

Burs/İndirim Durumu:

Burslu veya indirimli öğrencilerin durumu sisteme yansıtılmalı.

6. Yasal ve İdari Gereksinimler

Kayıt Tarihleri:

Sistem ders kayıt tarihlerini doğru şekilde uygulamalı, erken veya geç kayıt girişine izin vermemeli.

İzinler ve Onaylar:

Bazı özel dersler için danışman onayı veya bölüm başkanı onayı gerekebilir.

Raporlama:

Sistemde öğrencinin kayıt durumu ve seçilen derslerin raporlanabilir olması gerekir.

7. Kullanıcı Deneyimi ve Bildirimler

Uyarı ve Hata Mesajları:

Ders çakışması, kontenjan doluluğu, önkoşul eksikliği gibi durumlarda kullanıcıya açık ve anlaşılır mesaj gösterilmeli.

Kayıt Özeti:

Dersler seçildikten sonra öğrencinin ders planının bir özetini görebilmesi gerekir.

Mobil/Online Erişim:

Sisteme hem bilgisayar hem de mobil cihazlardan erişim sağlanmalı.

İstersen bunu bir sonraki adımda pseudocode veya mermaid akış diyagramı şeklinde görselleştirip ders kayıt sürecini algoritma olarak gösterebiliriz.

Bunu yapalım mı, yoksa önce gereksinimleri daha da detaylandırmamı ister misin?

Şunu dedin:
daha da detaylandır
ChatGPT:

Harika! O zaman üniversite ders kayıt gereksinimlerini daha mikro seviyede, olası tüm durumları kapsayacak şekilde detaylandıralım. Ben bunu modüler ve mantıksal başlıklar altında vereceğim, böylece hem sistem hem kullanıcı perspektifini görebiliriz.

1. Öğrenci Bilgileri ve Hesap Doğrulama

Kayıt Durumu:

Aktif, dondurulmuş, kaydı silinmiş, mezun durumları kontrol edilmeli.

Sistemde yanlış kayıtlı öğrenciler tespit edilmeli.

Kimlik Doğrulama:

Öğrenci numarası + şifre veya SSO (Single Sign-On) ile giriş zorunlu.

İki aşamalı doğrulama (2FA) opsiyonel ama güvenlik için tavsiye edilir.

Program ve Sınıf Bilgisi:

Lisans / yüksek lisans / doktora düzeyi.

Bölüm, yıl ve dönem bilgisi, ders seçimi için filtreleme amaçlı kullanılmalı.

2. Akademik Gereksinimler

Kredi Yükü:

Minimum ve maksimum kredi limiti kontrolü.

İstisnai durumlar (danışman onayı ile fazla kredi alma) sistemde izin verilmeli.

Önkoşul ve Eşkoşul Kontrolü:

Önkoşul dersleri tamamlanmamışsa kayıt engellenmeli.

Eşkoşul dersleri birlikte alınmazsa sistem uyarı vermeli.

Ders Tekrarı ve Muafiyet:

Daha önce başarıyla tamamlanan dersler tekrarlanamaz.

Yatay geçiş veya muafiyet durumu varsa bu dersler kayıt sırasında dikkate alınmalı.

Mezuniyet Planı ile Uyum:

Seçilen dersler öğrencinin mezuniyet planına uygun olmalı.

3. Ders ve Zamanlama Gereksinimleri

Ders Çakışmaları:

Ders saatleri veya laboratuvar uygulamaları çakışmamalı.

Öğrenciye alternatif ders saatleri önerilebilmeli.

Ders Kapasitesi / Kontenjan:

Ders kontenjanı dolmuşsa kayıt engellenmeli.

Bekleme listesi ve otomatik kayıt olanağı sağlanmalı.

Ders Tipi Ayrımı:

Teorik ders, uygulamalı ders, laboratuvar, staj veya proje dersleri ayrı kategorilerde gösterilmeli.

4. Teknik Sistem Gereksinimleri

Arayüz ve Kullanıcı Deneyimi:

Ders listesi, kredi, saat, öğretim üyesi bilgisi açıkça görüntülenmeli.

Kayıt sırasında yapılan hatalar (önkoşul eksik, kontenjan dolu) kullanıcıya anında bildirilmelidir.

Kayıt Onayı ve Kaydetme:

Seçimler kaydedilmeli ve öğrenci onayı alınmalı.

Sistemde kayıt tarihçesi ve değişiklik geçmişi saklanmalı.

Çoklu Platform Uyumluluğu:

Web, mobil tarayıcı ve gerekirse mobil uygulama ile uyumlu olmalı.

Güvenlik:

Öğrenci verileri güvenli bir şekilde saklanmalı (şifreleme, SSL, veri yedekleme).

5. Finansal ve İdari Gereksinimler

Ödeme Kontrolü:

Kayıt öncesi öğrenci ücret borcu veya taksit durumu kontrol edilmeli.

Ödeme tamamlanmamışsa ders kaydı engellenmeli.

Burs ve İndirimler:

Burslu öğrenciler veya indirimli ödeme durumu sisteme yansıtılmalı.

Danışman Onayı:

Bazı dersler için danışman veya bölüm başkanının onayı gerekebilir.

Yasal Süreler:

Ders ekleme/bırakma tarihleri sisteme tanımlanmalı ve geç kayıt engellenmeli.

6. Bildirim ve Raporlama Gereksinimleri

Hata ve Uyarılar:

Önkoşul eksik, kontenjan dolu, ders çakışması gibi durumlar açıkça belirtilmeli.

Özet ve Onay Raporu:

Kayıt tamamlandıktan sonra öğrenciye ders programı ve kredi durumu raporlanmalı.

İdari Raporlama:

Fakülte, bölüm ve üniversite düzeyinde ders kayıt istatistikleri alınabilmeli.

7. Ekstra Gereksinimler

Özel Durumlar:

Sağlık, engellilik veya pandemi gibi olağanüstü durumlarda özel ders planlama imkanı.

Bekleme Listesi Yönetimi:

Ders kontenjanı açıldığında otomatik kayıt için bekleme listesi mekanizması.

Ders Programı Görselleştirme:

Öğrencinin seçtiği dersler haftalık takvim şeklinde gösterilmeli.

Eğer istersen bunu bir sonraki adımda pseudocode veya akış diyagramı olarak net bir algoritmaya çevirebiliriz. Bu şekilde hem sistem hem süreç bir bakışta görülebilir ve eksiklik kontrolü kolaylaşır.

Bunu hemen yapalım mı?

Şunu dedin:
pseudocode oluştur
ChatGPT:

Tamam! Önce ayrıntılı gereksinimlere dayalı olarak üniversite ders kayıt sistemi için bir pseudocode tasa
ChatGPT:

Anladım, o zaman tamamen Türkçe sözde kod olarak yazalım, teknik terimler yerine açıklayıcı ifadeler kullanalım ve akış diyagramı formatına dönük kısımları çıkaralım. İşte detaylı Türkçe sözde kod:

ALGORİTMA ÜniversiteDersKaydı

GİRİŞ: öğrenci_no

// 1. Öğrenci doğrulama
EĞER ÖğrenciAktifDeğil(öğrenci_no) OLURSA
    YAZDIR "Öğrenci aktif değil, ders kaydı yapılamaz."
    DUR
SON

EĞER GirişBilgileriGeçersiz(öğrenci_no) OLURSA
    YAZDIR "Giriş bilgileri geçersiz."
    DUR
SON

// 2. Ödeme ve borç kontrolü
EĞER BorçVar(öğrenci_no) OLURSA
    YAZDIR "Ödeme yapılmamış, ders kaydı engellendi."
    DUR
SON

// 3. Ders listesi gösterimi
mevcut_dersler <- MevcutDersleriGetir(öğrenci_no)
YAZDIR mevcut_dersler

// 4. Öğrenci ders seçimi
seçilen_dersler <- ÖğrenciSeçimiAl()

// 5. Önkoşul ve eşkoşul kontrolü
HER ders İÇİN seçilen_dersler YAP
    EĞER ÖnKoşulEksik(öğrenci_no, ders) OLURSA
        YAZDIR "Önkoşul eksik: " + ders
        SEÇİLEN_DERSLERDEN_SİL(ders)
    SON
    EĞER EşKoşulEksik(öğrenci_no, ders) OLURSA
        YAZDIR "Eşkoşul eksik: " + ders
        SEÇİLEN_DERSLERDEN_SİL(ders)
    SON
SON

// 6. Kredi ve kontenjan kontrolü
toplam_kredi <- KrediHesapla(seçilen_dersler)
EĞER toplam_kredi > MAKS_KREDİ VEYA toplam_kredi < MİN_KREDİ OLURSA
    YAZDIR "Kredi sınırları aşıldı."
    DUR
SON

HER ders İÇİN seçilen_dersler YAP
    EĞER DersDolu(ders) OLURSA
        YAZDIR "Kontenjan dolu: " + ders
        BeklemeListesineEkle(öğrenci_no, ders)
        SEÇİLEN_DERSLERDEN_SİL(ders)
    SON
SON

// 7. Zaman çakışması kontrolü
EĞER SaatÇakışmasıVar(seçilen_dersler) OLURSA
    YAZDIR "Ders saatleri çakışıyor."
    DUR
SON

// 8. Danışman onayı kontrolü
HER ders İÇİN seçilen_dersler YAP
    EĞER DanışmanOnayıGerekli(ders) VE DanışmanOnayıYok(öğrenci_no, ders) OLURSA
        YAZDIR "Danışman onayı gerekli: " + ders
        SEÇİLEN_DERSLERDEN_SİL(ders)
    SON
SON

// 9. Dersleri kaydet
DersleriKaydet(öğrenci_no, seçilen_dersler)
YAZDIR "Ders kaydı başarıyla tamamlandı."

SON


Bu sürüm tamamen Türkçe ve anlaşılır, hem akademik hem idari tüm kontrolleri kapsıyor.

İstersen bir adım daha ileri gidip her bir kontrol için alt işlemleri ve hata mesajlarını daha ayrıntılı yazarak süreci tam anlamıyla “simülasyon” gibi detaylandırabiliriz.

Bunu da yapmamı ister misin?

ChatGPT hata yapabilir. Önemli bilgileri kontrol edin.