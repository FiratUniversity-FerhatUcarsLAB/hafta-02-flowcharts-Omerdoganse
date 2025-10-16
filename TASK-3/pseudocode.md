BASLA
YAZ "Hastane Randevu & Tahlil Sonuçları Sistemi başlatılıyor."

/* --- Ortak: Hasta Kimlik Doğrulama (TC No) --- */
DONGU kimlik_deneme_sayaci < 3 ISE
    YAZ "Lütfen TC Kimlik Numaranızı giriniz:"
    OKU tc_no
    EGER tc_dogrula(tc_no) ISE
        YAZ "TC doğrulandı. Hoşgeldiniz."
        kimlik_dogrulandi ← TRUE
        CIKIS DONGU
    DEGILSE
        kimlik_deneme_sayaci ← kimlik_deneme_sayaci + 1
        YAZ "TC doğrulama başarısız. Kalan deneme: ", 3 - kimlik_deneme_sayaci
    SON
SON DONGU

EGER kimlik_dogrulandi ≠ TRUE ISE
    YAZ "Kimlik doğrulama başarısız. Sistemden çıkılıyor."
    BITIR
SON

/* --- Ana Menü: İşlem Seçimi --- */
DONGU ana_menu = "E" ISE
    YAZ "İşlem seçiniz: 1 - Randevu Al, 2 - Tahlil Sonucu Gör, 3 - Çıkış"
    OKU islem_secim

    EGER islem_secim = "1" ISE
        GIT RANDEVU_MODULU
    DEGILSE EGER islem_secim = "2" ISE
        GIT TAHLIL_MODULU
    DEGILSE EGER islem_secim = "3" ISE
        YAZ "Sistemden çıkılıyor. İyi günler."
        ana_menu ← "H"
    DEGILSE
        YAZ "Geçersiz seçim. Lütfen tekrar deneyin."
    SON

SON DONGU

BITIR

/* -------------------- Modül 1: Randevu Alma -------------------- */
LABEL RANDEVU_MODULU
BASLA_RANDEVU:
YAZ "Randevu Modülü: Poliklinik seçiniz."

/* Poliklinik seçimi döngüsü */
DONGU pol_dongu = "E" ISE
    poliklinikler ← poliklinik_liste_getir()
    YAZ "Mevcut poliklinikler listeleniyor:"
    poliklinikleri_goster(poliklinikler)
    YAZ "Poliklinik kodu girin veya 'ÇIK' ile ana menüye dönün:"
    OKU poliklinik_kodu

    EGER poliklinik_kodu = "ÇIK" ISE
        YAZ "Randevu modülünden çıkılıyor."
        CIKIS DONGU
    SON

    /* Doktor listesi ve seçim (döngü) */
    doktorlar ← doktor_listesi_getir(poliklinik_kodu)
    YAZ "Seçilen polikliniğin doktorları listeleniyor:"
    doktorlari_goster(doktorlar)

    YAZ "Doktor ID girin veya 'GERI' ile poliklinik seçimine dönün:"
    OKU doktor_id
    EGER doktor_id = "GERI" ISE
        SON /* pol_dongu içinde tekrar poliklinik seçimi */
    SON

    /* Uygun saatlerin gösterimi ve seçim (döngü) */
    uygun_saatler ← doktor_uygun_saatler_getir(doktor_id)
    DONGU saat_dongu = "E" ISE
        YAZ "Mevcut uygun saatler:"
        saatleri_goster(uygun_saatler)
        YAZ "Saat seçin (ör. 09:30) veya 'GERI' ile doktor seçimine dönün:"
        OKU secilen_saat

        EGER secilen_saat = "GERI" ISE
            CIKIS DONGU
        SON

        EGER saat_musait_mi(doktor_id, secilen_saat) ISE
            YAZ "Seçilen saat müsait. Randevu onaylıyor musunuz? (E/H)"
            OKU onay
            EGER onay = "E" ISE
                randevu_id ← randevu_olustur(tc_no, poliklinik_kodu, doktor_id, secilen_saat)
                YAZ "Randevunuz onaylandı. Randevu ID: ", randevu_id
                /* SMS gönderimi */
                EGER telefon_var_mi(tc_no) ISE
                    sms_gonder(tc_no, "Randevunuz onaylandı. ID: " + randevu_id + " Saat: " + secilen_saat)
                    YAZ "Onay SMS'i gönderildi."
                DEGILSE
                    YAZ "Kayıtlı telefon numarası bulunamadı. SMS gönderilemedi."
                SON
                /* Randevu sonrası çıkış veya yeni işlem */
                YAZ "Başka işlem yapmak ister misiniz? (E/H)"
                OKU devam_randevu
                EGER devam_randevu = "E" ISE
                    GIT BASLA_RANDEVU
                DEGILSE
                    GIT ANA_MENU_RETURN
                SON
            DEGILSE
                YAZ "Randevu onayı iptal edildi. Lütfen başka saat seçin."
            SON
        DEGILSE
            YAZ "Seçilen saat dolu veya geçersiz. Lütfen başka saat seçin."
        SON
    SON DONGU /* saat_dongu */

SON DONGU /* pol_dongu */

LABEL ANA_MENU_RETURN
YAZ "Ana menüye dönülüyor."
GIT ANA_MENU_END

/* -------------------- Modül 2: Tahlil Sonuçları -------------------- */
LABEL TAHLIL_MODULU
BASLA_TAHLIL:
YAZ "Tahlil Sonuçları Modülü."

/* Kimlik zaten doğrulandı önceki kısımda; opsiyonel tekrar kontrol */
EGER kimlik_dogrulandi ≠ TRUE ISE
    YAZ "Lütfen önce kimlik doğrulaması yapınız."
    GIT ANA_MENU_END
SON

/* Tahlil var mı kontrolü */
EGER tahlil_var_mi(tc_no) ISE
    YAZ "Tahlil kayıtlarınız bulunuyor. Hangi tahlili görmek istersiniz?"
    tahlil_listesi ← tahlil_listesi_getir(tc_no)
    tahlilleri_goster(tahlil_listesi)

    DONGU tahlil_secim_dongu = "E" ISE
        YAZ "Tahlil ID girin veya 'ÇIK' ile ana menüye dönün:"
        OKU tahlil_id
        EGER tahlil_id = "ÇIK" ISE
            CIKIS DONGU
        SON

        /* Sonuç hazır mı kontrolü */
        EGER sonuc_hazir_mi(tahlil_id) ISE
            sonuc ← tahlil_sonuc_getir(tahlil_id)
            YAZ "Tahlil sonucu gösteriliyor:"
            tahlil_sonuclarini_goster(sonuc)

            YAZ "Sonuç PDF indirilsin mi? (E/H)"
            OKU pdf_secim
            EGER pdf_secim = "E" ISE
                pdf_dosyasi ← tahlil_pdf_olustur(tahlil_id)
                YAZ "PDF hazırlanıyor... İndirilebilir dosya: ", pdf_dosyasi
            SON
        DEGILSE
            YAZ "Sonuçlar henüz hazır değil. Lütfen daha sonra tekrar kontrol edin."
            YAZ "SMS ile bilgilendirilmek ister misiniz sonuç hazır olduğunda? (E/H)"
            OKU sms_bilgi
            EGER sms_bilgi = "E" ISE
                EGER telefon_var_mi(tc_no) ISE
                    sms_kayit_bildirimi_ekle(tc_no, tahlil_id)
                    YAZ "Hazır olduğunda SMS ile bilgilendirileceksiniz."
                DEGILSE
                    YAZ "Telefon numarası kaydı yok. SMS yapılamıyor."
                SON
            SON
        SON

        YAZ "Başka tahlil görüntülemek ister misiniz? (E/H)"
        OKU devam_tahlil
        EGER devam_tahlil = "H" ISE
            CIKIS DONGU
        SON
    SON DONGU /* tahlil_secim_dongu */

DEGILSE
    YAZ "Herhangi bir tahlil kaydı bulunmamaktadır."
    YAZ "Yeni randevu almak ister misiniz? (E/H)"
    OKU randevu_yonlendirme
    EGER randevu_yonlendirme = "E" ISE
        GIT RANDEVU_MODULU
    SON
SON

YAZ "Tahlil modülü işlemi tamamlandı."
GIT ANA_MENU_RETURN

/* -------------------- Ana Menüye Dönüş Noktası -------------------- */
LABEL ANA_MENU_END
YAZ "Ana menüye döndünüz. Lütfen tekrar işlem seçiniz."

/* Program akışı burada ana menü döngüsüne geri döner (ana menü başlangıcındaki DONGU) */

BITIR
