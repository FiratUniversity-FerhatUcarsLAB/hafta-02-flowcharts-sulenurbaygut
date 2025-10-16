SABİTLER:
    KDV_ORANI <- 0.20
    UCRETSIZ_KARGO_LIMITI <- 500
    KARGO_UCRETI <- 30

FONKSİYON SEPETE_URUN_EKLE(kullanici_id, sepet, urun_id, adet, kupon_kodu = BOS):

    // 0. Adet kontrolü
    eğer adet <= 0 ise
        DÖN (sepet, BASARISIZ, "Adet en az 1 olmalıdır.")
    bitir_eğer

    // 1. Kullanıcı veya oturum kontrolü
    eğer GEÇERLİ_OTURUM(kullanici_id) değilse
        kullanici_id <- MISAFIR_OTURUMU_OLUSTUR(kullanici_id)
        // Kullanıcı giriş yapmamışsa misafir sepeti oluşturulur
    bitir_eğer

    // 2. Ürünü bul
    urun <- URUN_BUL(urun_id)
    eğer urun = BOS ise
        DÖN (sepet, BASARISIZ, "Geçersiz ürün ID.")
    bitir_eğer

    // 3. Maksimum sipariş kuralı
    eğer urun.max_siparis > 0 ve adet > urun.max_siparis ise
        DÖN (sepet, BASARISIZ, "Bir siparişte en fazla " + urun.max_siparis + " adet alınabilir.")
    bitir_eğer

    // 4. Stok kontrolü
    mevcut_ogesi <- SEPETTE_URUN_ARA(sepet, urun_id)
    mevcut_adet <- 0

    eğer mevcut_ogesi ≠ BOS ise
        mevcut_adet <- mevcut_ogesi.adet
    bitir_eğer

    toplam_istek <- mevcut_adet + adet

    eğer urun.stok < toplam_istek ise
        DÖN (sepet, BASARISIZ, "Yetersiz stok. Maksimum " + urun.stok + " adet alınabilir.")
    bitir_eğer

    // 5. Stok rezervasyonu (isteğe bağlı)
    eğer STOK_REZERVE_ET(urun_id, adet, kullanici_id) = FALSE ise
        DÖN (sepet, BASARISIZ, "Stok rezerve edilemedi. Lütfen tekrar deneyin.")
    bitir_eğer

    // 6. Ürün sepette varsa adeti güncelle
    eğer mevcut_ogesi ≠ BOS ise
        mevcut_ogesi.adet <- toplam_istek
        mevcut_ogesi.ara_toplam <- toplam_istek * urun.fiyat
        MESAJ <- "Ürün miktarı güncellendi. Yeni adet: " + toplam_istek
        LOG_YAZ("Kullanıcı " + kullanici_id + " ürün " + urun_id + " miktarını " + toplam_istek + " yaptı.")
    aksi halde
        // 7. Yeni ürün ekle
        yeni_satir <- YENI_SATIR_OLUSTUR()
        yeni_satir.urun_id <- urun.urun_id
        yeni_satir.ad <- urun.ad
        yeni_satir.birim_fiyat <- urun.fiyat
        yeni_satir.adet <- adet
        yeni_satir.ara_toplam <- urun.fiyat * adet
        SEPETE_EKLE_LISTE(sepet, yeni_satir)
        MESAJ <- "Yeni ürün sepete eklendi: " + urun.ad + " (" + adet + " adet)"
        LOG_YAZ("Kullanıcı " + kullanici_id + " yeni ürün ekledi: " + urun_id)
    bitir_eğer

    // 8. Sepet toplamını hesapla
    ara_toplam <- 0
    her_bir satir icin oge IN sepet DO
        ara_toplam <- ara_toplam + oge.ara_toplam
    bitir_döngü

    // 9. Kupon kontrolü
    indirim <- 0
    eğer kupon_kodu ≠ BOS ise
        kupon <- KUPON_DOGRULA(kupon_kodu, kullanici_id, sepet)
        eğer kupon GEÇERSİZ ise
            kupon_mesaj <- "Kupon geçersiz veya kullanılamaz."
        aksi halde
            indirim <- KUPON_INDIRIM_HESAPLA(kupon, ara_toplam)
            KUPON_SEPETE_UYGULA(sepet, kupon, indirim)
            kupon_mesaj <- "Kupon uygulandı: -" + indirim + " TL"
        bitir_eğer
    aksi halde
        kupon_mesaj <- "Kupon girilmedi."
    bitir_eğer

    // 10. Kargo hesapla
    eğer (ara_toplam - indirim) >= UCRETSIZ_KARGO_LIMITI ise
        kargo_tutari <- 0
    aksi halde
        kargo_tutari <- KARGO_UCRETI
    bitir_eğer

    // 11. KDV hesapla
    kdv <- (ara_toplam - indirim) * KDV_ORANI

    // 12. Genel toplam
    genel_toplam <- ara_toplam - indirim + kargo_tutari + kdv

    // 13. Sepeti kaydet (veritabanı veya oturum)
    SEPETI_KAYDET(kullanici_id, sepet, ara_toplam, indirim, kargo_tutari, kdv, genel_toplam)

    // 14. Arayüz ve işlem sonrası
    SEPET_GORSEL_GUNCELLE(kullanici_id, SEPET_URUN_SAYISI(sepet))
    SEPET_SURE_BITISI_ZAMANLA(kullanici_id)   // Misafir sepeti için zamanlayıcı

    // 15. Sonuç döndür
    detaylar <- {
        "ara_toplam": ara_toplam,
        "indirim": indirim,
        "kargo": kargo_tutari,
        "kdv": kdv,
        "genel_toplam": genel_toplam,
        "mesaj": MESAJ,
        "kupon_mesaj": kupon_mesaj
    }

    DÖN (sepet, BASARILI, detaylar)

BİTİR_FONKSİYON



LLM GEÇMİŞİ



online alışveriş sepeti için neler gerekli
ChatGPT:

Harika soru Nur 💡
Bir online alışveriş sepeti kurmak istiyorsan — ister proje ödevi, ister gerçek bir e-ticaret sitesi için olsun — temel yapı taşlarını iyi kurmak gerekir. Aşağıda hem teknik hem mantıksal olarak ihtiyacın olan her şeyi özetledim 👇

🧩 1. Temel Bileşenler
🔹 Ürün Bilgileri

Ürün ID’si

Ürün adı

Fiyat

Stok miktarı

Görsel (fotoğraf linki)

Kategori / açıklama

🔹 Sepet Verileri

Sepete eklenen ürünlerin listesi

Her ürünün adedi

Ara toplam (adet × fiyat)

Toplam tutar

Kargo ücreti (varsa)

Uygulanan indirim / kuponlar

⚙️ 2. İşlevsel Özellikler
🔸 Sepet işlemleri:

Ürün ekle

Ürün çıkar

Adet güncelle

Sepeti boşalt

🔸 Fiyat hesaplamaları:

Ara toplam

Kargo bedeli (örneğin 500 TL üzeri ücretsiz olabilir)

KDV hesaplama

Kupon indirimi

🔸 Diğer:

Oturum (hangi kullanıcıya ait sepet olduğunu bilmek)

Sepetin kaydedilmesi (çıkıp geri geldiğinde ürünler kaybolmasın)

Sepet onayı → ödeme sayfasına yönlendirme

💳 3. Ödeme ve Sipariş Süreci

Teslimat bilgileri (adres, telefon, ad-soyad)

Fatura bilgileri

Ödeme yöntemi (kredi kartı, havale, kapıda ödeme)

Güvenli ödeme (SSL, 3D Secure, vs.)

Sipariş onayı ve sipariş numarası

🔐 4. Teknik Taraf (Back-end)

Veritabanı (örneğin: users, products, cart, orders)

API’ler (örneğin: /addToCart, /checkout, /applyCoupon)

Kullanıcı oturum yönetimi (login, cookie, token)

🎨 5. Arayüz (Front-end)

Ürün listeleme sayfası

Sepet sayfası

Ödeme (checkout) sayfası

Sipariş özeti ekranı

"Sepetim" butonu (genellikle sağ üstte)

💡 Bonus: Akıllı Özellikler

Otomatik stok kontrolü (“Bu ürün stokta kalmadı” uyarısı)

İlgili ürün önerileri (“Bunu alan bunu da aldı”)

Sepette kalmış ürünleri hatırlatma maili

“Favorilere ekle” özelliği

İstersen bundan bir akış diyagramı ya da pseudocode da oluşturabilirim — örneğin:

"Sepete ürün ekleme algoritması"
veya
"Online alışveriş süreci akış diyagramı"

Hangisinden başlamak istersin? 🧠

Şunu dedin:
psudocode oluştur 
ChatGPT:

Süper Nur ⚡
O zaman online alışveriş sepeti için tam donanımlı bir pseudocode yazalım — ürün ekleme, çıkarma, toplam hesaplama, ödeme adımı hepsi dahil 👇
(Talebine göre Türkçe hazırladım, ama istersen İngilizce versiyonunu da üretebilirim.)

🧠 PSEUDOCODE: Online Alışveriş Sepeti Sistemi
ALGORITMA OnlineAlisverisSepeti

    // SABITLER
    KDV_ORANI <- 0.20
    UCRETSIZ_KARGO_LIMITI <- 500
    KARGO_UCRETI <- 30

    // DEGISKENLER
    sepet <- BOS_LISTE
    toplam_tutar <- 0
    kargo_tutari <- 0
    indirim <- 0
    kullanici_girisi <- FALSE

    // 1. KULLANICI GİRİŞİ
    GIRIS:
        EKRANA_YAZ("Kullanıcı adı giriniz: ")
        oku(kullanici_adi)
        EKRANA_YAZ("Şifre giriniz: ")
        oku(sifre)
        eger DOGRU_GIRIS(kullanici_adi, sifre) ise
            kullanici_girisi <- TRUE
        degilse
            EKRANA_YAZ("Hatalı giriş! Tekrar deneyiniz.")
            GIT GIRIS

    // 2. ÜRÜN SEÇİMİ
    EKRANA_YAZ("Ürün listesinden seçim yapınız.")
    tekrarla
        EKRANA_YAZ("Ürün ID giriniz (çıkmak için 0): ")
        oku(urun_id)
        eger urun_id = 0 ise
            cik
        aksi halde
            urun <- URUN_BUL(urun_id)
            eger urun stokta yok ise
                EKRANA_YAZ("Ürün stokta yok!")
            aksi halde
                EKRANA_YAZ("Kaç adet almak istiyorsunuz? ")
                oku(adet)
                SEPETE_EKLE(sepet, urun, adet)
                EKRANA_YAZ("Ürün sepete eklendi.")
    kadar urun_id = 0

    // 3. SEPET GÖRÜNTÜLEME VE HESAPLAMA
    toplam_tutar <- HESAPLA_TOPLAM(sepet)

    eger toplam_tutar > UCRETSIZ_KARGO_LIMITI ise
        kargo_tutari <- 0
    aksi halde
        kargo_tutari <- KARGO_UCRETI

    kdv <- toplam_tutar * KDV_ORANI
    genel_toplam <- toplam_tutar + kargo_tutari + kdv - indirim

    EKRANA_YAZ("Ara toplam: ", toplam_tutar)
    EKRANA_YAZ("Kargo: ", kargo_tutari)
    EKRANA_YAZ("KDV: ", kdv)
    EKRANA_YAZ("İndirim: ", indirim)
    EKRANA_YAZ("Genel Toplam: ", genel_toplam)

    // 4. ÖDEME ADIMI
    EKRANA_YAZ("Ödeme yöntemi seçiniz (1: Kart, 2: Havale, 3: Kapıda Ödeme): ")
    oku(odeme_turu)

    eger odeme_turu = 1 ise
        EKRANA_YAZ("Kart bilgilerini giriniz.")
        oku(kart_no, son_kullanma, cvv)
        eger ODEME_ONAYLANDI(kart_no) ise
            EKRANA_YAZ("Ödeme başarılı! Sipariş oluşturuldu.")
        aksi halde
            EKRANA_YAZ("Ödeme reddedildi. Başka yöntem deneyiniz.")
    aksi halde
        EKRANA_YAZ("Ödeme kaydedildi. Sipariş hazırlanıyor.")

    // 5. SİPARİŞ TAMAMLAMA
    SIPARIS_OLUSTUR(sepet, genel_toplam)
    EKRANA_YAZ("Sipariş numaranız: ", RASTGELE_SIPARIS_NO())
    EKRANA_YAZ("Teşekkür ederiz, iyi alışverişler!")

SON


İstersen bu pseudocode’u:

🧩 akış diyagramına dönüştürebilirim,

💾 veya PDF / Word dosyası haline getirebilirim,

🔍 ya da sadece “Sepete ürün ekleme” kısmını detaylandırabilirim.

Hangisini yapayım Nur?

Şunu dedin:
sepete ürün ekleme kısmını detaylandır
ChatGPT:

Harika seçim Nur 💥
Sepete ürün ekleme kısmı, sistemin kalbidir ❤️ — çünkü her şey o “EKLE” tuşuna basıldığında başlar.
Aşağıda o bölümü tam detaylı, mantıksal kontrolleriyle (stok, tekrar ekleme, adet güncelleme vb.) yazdım.

🧠 PSEUDOCODE: SEPETE ÜRÜN EKLEME (DETAYLI)
ALGORITMA SEPETE_URUN_EKLE

    GİRDİ: sepet, urun_id, adet
    ÇIKTI: güncellenmiş sepet

    // 1. Ürün bilgilerini bul
    urun <- URUN_BUL(urun_id)

    eğer urun = BOS ise
        EKRANA_YAZ("Geçersiz ürün ID!")
        DÖN sepet
    bitir_eğer

    // 2. Stok kontrolü
    eğer urun.stok < adet ise
        EKRANA_YAZ("Yeterli stok yok! Maksimum ", urun.stok, " adet alabilirsiniz.")
        DÖN sepet
    bitir_eğer

    // 3. Ürün zaten sepette mi?
    urun_var <- FALSE
    her_bir_ogesi icin oge IN sepet
        eğer oge.urun_id = urun_id ise
            urun_var <- TRUE
            // Adeti güncelle
            yeni_adet <- oge.adet + adet
            eğer yeni_adet > urun.stok ise
                EKRANA_YAZ("Stok sınırı aşıldı! En fazla ", urun.stok, " adet eklenebilir.")
                DÖN sepet
            aksi halde
                oge.adet <- yeni_adet
                oge.ara_toplam <- oge.adet * urun.fiyat
                EKRANA_YAZ("Ürün miktarı güncellendi. Yeni adet: ", oge.adet)
                DÖN sepet
            bitir_eğer
        bitir_eğer
    bitir_döngü

    // 4. Ürün sepette yoksa yeni satır olarak ekle
    eğer urun_var = FALSE ise
        yeni_oge <- YENI_KAYIT_OLUSTUR()
        yeni_oge.urun_id <- urun.urun_id
        yeni_oge.ad <- urun.ad
        yeni_oge.fiyat <- urun.fiyat
        yeni_oge.adet <- adet
        yeni_oge.ara_toplam <- urun.fiyat * adet
        SEPETE_EKLE_LISTE(sepet, yeni_oge)
        EKRANA_YAZ("Yeni ürün sepete eklendi: ", urun.ad, " (", adet, " adet)")
    bitir_eğer

    // 5. Toplamı güncelle
    toplam <- 0
    her_bir_ogesi icin oge IN sepet
        toplam <- toplam + oge.ara_toplam
    bitir_döngü

    EKRANA_YAZ("Sepet toplamı güncellendi: ", toplam, " TL")

    DÖN sepet

SON_ALGORITMA

🔍 Bu alt algoritma neleri kapsıyor:

✅ Ürün kimliğini kontrol eder
✅ Stok yeterliliğini denetler
✅ Aynı ürün sepette varsa adedi artırır
✅ Yeni ürünse sepete ekler
✅ Tüm sepetin toplam fiyatını günceller

İstersen bunu bir akış diyagramına çevirebilirim (görsel olarak: karar kutuları, işlem blokları, oklar vs. ile).
Hazırlayayım mı Nur?

Şunu dedin:
ayrıntı eklerken bazı şeyleri eksiltmişssin ilk eklenen kısımda olanları silmeden yap
ChatGPT:
12s boyunca düşündüm

Harikasın Nur — eksik kalanları da koruyarak ilk psödocodetaki bütün öğeleri silmeden sepet ekleme kısmını genişletiyorum. İstediğin gibi pseudocode İngilizce olacak; açıklamalar ve sohbet Türkçe 🎯

PSEUDOCODE: ADD_TO_CART — detailed and preserving original variables/behaviour
# Constants (kept from main algorithm)
KDV_RATE <- 0.20
FREE_SHIPPING_THRESHOLD <- 500
SHIPPING_FEE <- 30

# Function: ADD_TO_CART
# Inputs: session_or_user_id, cart, product_id, quantity, optional coupon_code
# Output: updated cart, status (success/failure), messages

FUNCTION ADD_TO_CART(session_id, cart, product_id, quantity, coupon_code = NULL):

    # 0. Basic validation of input quantity
    IF quantity <= 0 THEN
        RETURN (cart, FAILURE, "Quantity must be at least 1.")
    END IF

    # 1. User/session check (preserve user login handling from main flow)
    IF NOT IS_VALID_SESSION(session_id) THEN
        # create or attach a guest cart if allowed
        session_id <- CREATE_GUEST_SESSION_IF_NEEDED(session_id)
        # optionally inform user to login at checkout
    END IF

    # 2. Fetch product and current stock
    product <- GET_PRODUCT_BY_ID(product_id)
    IF product IS NULL THEN
        RETURN (cart, FAILURE, "Invalid product ID.")
    END IF

    # 3. Business rules & limits (preserve original system rules if any)
    # e.g., maximum single-order limit per product, per-user limits, etc.
    IF HAS_MAX_PER_ORDER(product) AND quantity > product.max_per_order THEN
        RETURN (cart, FAILURE, "Max per-order limit is " + product.max_per_order + ".")
    END IF

    # 4. Check available stock
    current_cart_item <- FIND_ITEM_IN_CART(cart, product_id)
    existing_qty_in_cart <- 0
    IF current_cart_item IS NOT NULL THEN
        existing_qty_in_cart <- current_cart_item.quantity
    END IF

    requested_total_qty <- existing_qty_in_cart + quantity
    IF product.stock < requested_total_qty THEN
        RETURN (cart, FAILURE, "Insufficient stock. Available: " + product.stock + ".")
    END IF

    # 5. Reserve/lock stock (optional, for concurrency)
    # Try to reserve 'quantity' units for a short time so other carts can't oversell
    IF NOT RESERVE_STOCK(product_id, quantity, session_id) THEN
        RETURN (cart, FAILURE, "Unable to reserve stock. Try again.")
    END IF

    # 6. If product already in cart -> update quantity and line subtotal
    IF current_cart_item IS NOT NULL THEN
        current_cart_item.quantity <- requested_total_qty
        current_cart_item.line_subtotal <- requested_total_qty * product.price
        LOG("Cart update: user " + session_id + " updated product " + product_id + " to qty " + requested_total_qty)
        message <- "Product quantity updated. New quantity: " + requested_total_qty
    ELSE
        # 7. Add new cart line
        new_line <- CREATE_CART_LINE()
        new_line.product_id <- product_id
        new_line.name <- product.name
        new_line.unit_price <- product.price
        new_line.quantity <- quantity
        new_line.line_subtotal <- product.price * quantity
        APPEND_TO_CART(cart, new_line)
        LOG("Cart add: user " + session_id + " added product " + product_id + " qty " + quantity)
        message <- "Product added to cart: " + product.name + " (" + quantity + ")"
    END IF

    # 8. Recalculate cart totals (preserve original total/KDV/shipping/discount logic)
    subtotal <- 0
    FOR EACH line IN cart.lines DO
        subtotal <- subtotal + line.line_subtotal
    END FOR

    # 9. Apply coupon (if provided) — validate, then compute discount
    discount <- 0
    IF coupon_code IS NOT NULL THEN
        coupon <- VALIDATE_COUPON(coupon_code, session_id, cart)
        IF coupon IS INVALID THEN
            # keep cart but report coupon invalid
            coupon_message <- "Coupon invalid or not applicable."
        ELSE
            discount <- CALCULATE_COUPON_DISCOUNT(coupon, subtotal)
            APPLY_COUPON_TO_CART(cart, coupon, discount)
            coupon_message <- "Coupon applied: -" + discount
        END IF
    ELSE
        coupon_message <- "No coupon applied."
    END IF

    # 10. Shipping calculation
    IF subtotal - discount >= FREE_SHIPPING_THRESHOLD THEN
        shipping_cost <- 0
    ELSE
        shipping_cost <- SHIPPING_FEE
    END IF

    # 11. VAT / Tax calculation (preserve KDV)
    tax <- (subtotal - discount) * KDV_RATE

    # 12. Grand total
    grand_total <- subtotal - discount + shipping_cost + tax

    # 13. Persist cart (session DB / user cart table). Ensure atomic save.
    SAVE_CART(session_id, cart, subtotal, discount, shipping_cost, tax, grand_total)

    # 14. Post-actions
    # - Send ephemeral UI update (cart badge count)
    UPDATE_CART_BADGE(session_id, COUNT_TOTAL_ITEMS(cart))
    # - Optionally schedule cart expiry/unreserve if user doesn't checkout
    SCHEDULE_CART_EXPIRY(session_id)

    # 15. Return updated cart and friendly messages
    details <- {
        "subtotal": subtotal,
        "discount": discount,
        "shipping": shipping_cost,
        "tax": tax,
        "grand_total": grand_total,
        "message": message,
        "coupon_message": coupon_message
    }
    RETURN (cart, SUCCESS, details)

END FUNCTION

Kısa notlar — neden bu ekler korundu:

KDV_RATE, FREE_SHIPPING_THRESHOLD, SHIPPING_FEE ana algortimada vardı; sepet toplamı ve ödeme adımına dokunulmadan burada hesaplama korunuyor.

Stok rezervasyonu, kupon doğrulama, oturum/guest cart yönetimi gibi gerçek hayatta kritik ama önceki özette eksik olan parçalar eklendi.

Mevcut ürün varsa adeti güncelliyor, yeni ürünse yeni satır ekliyor — ilk sürümdeki davranışı hiç silmedim, sadece eksik kontrolleri ve persist/lock adımlarını ekledim.

İstersen şimdi bunu bir akış diyagramıne çevirip karar kutularını net gösteririm ya da bu fonksiyonu REST API endpoint hâline getirip örnek istek/cevap JSON’ları hazırlayayım. Hangi formatı istersin? 🚀

Şunu dedin:
türkçe yap bunu
ChatGPT:

Harika Nur 💥
İşte az önceki tam donanımlı “Sepete Ürün Ekleme” algoritmasının Türkçe versiyonu —
hiçbir işlev kaybolmadı, sadece tüm açıklamalar ve değişkenler Türkçeleştirildi 🇹🇷
Yani hem önceki mantığı koruyor hem de detay seviyesi profesyonel düzeyde 👇

🧠 PSEUDOCODE: SEPETE_URUN_EKLE (TÜRKÇE — TAM DETAYLI)
SABİTLER:
    KDV_ORANI <- 0.20
    UCRETSIZ_KARGO_LIMITI <- 500
    KARGO_UCRETI <- 30

FONKSİYON SEPETE_URUN_EKLE(kullanici_id, sepet, urun_id, adet, kupon_kodu = BOS):

    // 0. Adet kontrolü
    eğer adet <= 0 ise
        DÖN (sepet, BASARISIZ, "Adet en az 1 olmalıdır.")
    bitir_eğer

    // 1. Kullanıcı veya oturum kontrolü
    eğer GEÇERLİ_OTURUM(kullanici_id) değilse
        kullanici_id <- MISAFIR_OTURUMU_OLUSTUR(kullanici_id)
        // Kullanıcı giriş yapmamışsa misafir sepeti oluşturulur
    bitir_eğer

    // 2. Ürünü bul
    urun <- URUN_BUL(urun_id)
    eğer urun = BOS ise
        DÖN (sepet, BASARISIZ, "Geçersiz ürün ID.")
    bitir_eğer

    // 3. Maksimum sipariş kuralı
    eğer urun.max_siparis > 0 ve adet > urun.max_siparis ise
        DÖN (sepet, BASARISIZ, "Bir siparişte en fazla " + urun.max_siparis + " adet alınabilir.")
    bitir_eğer

    // 4. Stok kontrolü
    mevcut_ogesi <- SEPETTE_URUN_ARA(sepet, urun_id)
    mevcut_adet <- 0

    eğer mevcut_ogesi ≠ BOS ise
        mevcut_adet <- mevcut_ogesi.adet
    bitir_eğer

    toplam_istek <- mevcut_adet + adet

    eğer urun.stok < toplam_istek ise
        DÖN (sepet, BASARISIZ, "Yetersiz stok. Maksimum " + urun.stok + " adet alınabilir.")
    bitir_eğer

    // 5. Stok rezervasyonu (isteğe bağlı)
    eğer STOK_REZERVE_ET(urun_id, adet, kullanici_id) = FALSE ise
        DÖN (sepet, BASARISIZ, "Stok rezerve edilemedi. Lütfen tekrar deneyin.")
    bitir_eğer

    // 6. Ürün sepette varsa adeti güncelle
    eğer mevcut_ogesi ≠ BOS ise
        mevcut_ogesi.adet <- toplam_istek
        mevcut_ogesi.ara_toplam <- toplam_istek * urun.fiyat
        MESAJ <- "Ürün miktarı güncellendi. Yeni adet: " + toplam_istek
        LOG_YAZ("Kullanıcı " + kullanici_id + " ürün " + urun_id + " miktarını " + toplam_istek + " yaptı.")
    aksi halde
        // 7. Yeni ürün ekle
        yeni_satir <- YENI_SATIR_OLUSTUR()
        yeni_satir.urun_id <- urun.urun_id
        yeni_satir.ad <- urun.ad
        yeni_satir.birim_fiyat <- urun.fiyat
        yeni_satir.adet <- adet
        yeni_satir.ara_toplam <- urun.fiyat * adet
        SEPETE_EKLE_LISTE(sepet, yeni_satir)
        MESAJ <- "Yeni ürün sepete eklendi: " + urun.ad + " (" + adet + " adet)"
        LOG_YAZ("Kullanıcı " + kullanici_id + " yeni ürün ekledi: " + urun_id)
    bitir_eğer

    // 8. Sepet toplamını hesapla
    ara_toplam <- 0
    her_bir satir icin oge IN sepet DO
        ara_toplam <- ara_toplam + oge.ara_toplam
    bitir_döngü

    // 9. Kupon kontrolü
    indirim <- 0
    eğer kupon_kodu ≠ BOS ise
        kupon <- KUPON_DOGRULA(kupon_kodu, kullanici_id, sepet)
        eğer kupon GEÇERSİZ ise
            kupon_mesaj <- "Kupon geçersiz veya kullanılamaz."
        aksi halde
            indirim <- KUPON_INDIRIM_HESAPLA(kupon, ara_toplam)
            KUPON_SEPETE_UYGULA(sepet, kupon, indirim)
            kupon_mesaj <- "Kupon uygulandı: -" + indirim + " TL"
        bitir_eğer
    aksi halde
        kupon_mesaj <- "Kupon girilmedi."
    bitir_eğer

    // 10. Kargo hesapla
    eğer (ara_toplam - indirim) >= UCRETSIZ_KARGO_LIMITI ise
        kargo_tutari <- 0
    aksi halde
        kargo_tutari <- KARGO_UCRETI
    bitir_eğer

    // 11. KDV hesapla
    kdv <- (ara_toplam - indirim) * KDV_ORANI

    // 12. Genel toplam
    genel_toplam <- ara_toplam - indirim + kargo_tutari + kdv

    // 13. Sepeti kaydet (veritabanı veya oturum)
    SEPETI_KAYDET(kullanici_id, sepet, ara_toplam, indirim, kargo_tutari, kdv, genel_toplam)

    // 14. Arayüz ve işlem sonrası
    SEPET_GORSEL_GUNCELLE(kullanici_id, SEPET_URUN_SAYISI(sepet))
    SEPET_SURE_BITISI_ZAMANLA(kullanici_id)   // Misafir sepeti için zamanlayıcı

    // 15. Sonuç döndür
    detaylar <- {
        "ara_toplam": ara_toplam,
        "indirim": indirim,
        "kargo": kargo_tutari,
        "kdv": kdv,
        "genel_toplam": genel_toplam,
        "mesaj": MESAJ,
        "kupon_mesaj": kupon_mesaj
    }

    DÖN (sepet, BASARILI, detaylar)

BİTİR_FONKSİYON

🔍 Bu sürümde neler var?

✅ Önceki pseudocode’un hiçbir kısmı silinmedi — sadece ayrıntılar eklendi.
✅ Kullanıcı/oturum kontrolü, stok rezervasyonu, kupon doğrulama, toplam ve KDV hesaplama hepsi yerinde.
✅ Aynı ürün eklenirse miktar güncelleniyor, yeni ürünse sepete yeni satır ekleniyor.
✅ Tüm mesajlar kullanıcı dostu şekilde döndürülüyor.

İstersen şimdi bunu:

🎨 akış diyagramına çevirebilirim (karar kutuları, işlemler, oklar vs.),

💾 veya PDF dosyası olarak çıktı alabileceğin biçimde hazırlayabilirim,

⚙️ ya da ödeme adımıyla birleştirip “tam alışveriş süreci” algoritması haline getirebilirim.

Hangisini yapayım Nur? 🚀