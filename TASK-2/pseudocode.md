BASLA

YAZ "E-ticaret Sepet & Ödeme Sistemi başlatılıyor."

/* Kullanıcı Girişi */
YAZ "Lütfen kullanıcı adı giriniz:"
OKU kullanici_adi
YAZ "Lütfen şifre giriniz:"
OKU sifre

EGER dogru_giris_mi(kullanici_adi, sifre) ISE
    logged_in ← TRUE
    YAZ "Giriş başarılı."
DEGILSE
    logged_in ← FALSE
    YAZ "Giriş başarısız. Misafir olarak devam ediliyor."
SON

/* Ürün gezinme ve ekleme */
DONGU gezinme = "DEVAM" ISE
    YAZ "Kategoriler listeleniyor..."
    DONGU kategori_secimi != "ÇIK" ISE
        YAZ "Kategori seçiniz veya 'ÇIK' ile çıkınız:"
        OKU kategori_secimi
        EGER kategori_secimi = "ÇIK" ISE
            CIKIS DONGU
        SON
        YAZ "Seçilen kategori ürünleri listeleniyor..."
        YAZ "Ürün ID girin veya 'GERI' ile kategori seçimine dönün:"
        OKU urun_id
        EGER urun_id = "GERI" ISE
            SON
        SON

        YAZ "Sepete eklemek ister misiniz? (E/H)"
        OKU ekle
        EGER ekle = "E" ISE
            YAZ "İstenen miktarı giriniz:"
            OKU miktar
            /* Stok kontrolü */
            EGER stok_sorgula(urun_id) >= miktar ISE
                sepete_ekle(urun_id, miktar)
                YAZ "Ürün sepete eklendi."
            DEGILSE
                YAZ "Yetersiz stok! Mevcut stok: ", stok_sorgula(urun_id)
            SON
        SON
    SON
    YAZ "Kategori gezinmesini bitirmek istiyor musunuz? (E/H)"
    OKU gezinme
SON

/* Sepeti görüntüleme ve düzenleme */
DONGU sepet_islem = "E" ISE
    YAZ "Sepetiniz gösteriliyor..."
    sepetteki_urunleri_goster()
    YAZ "Sepeti düzenlemek ister misiniz? (Miktar/Sil/H) :"
    OKU secim
    EGER secim = "Miktar" ISE
        YAZ "Hangi ürün ID'si?"; OKU duzen_id
        YAZ "Yeni miktarı girin:"; OKU yeni_miktar
        EGER stok_sorgula(duzen_id) >= yeni_miktar ISE
            sepet_guncelle_miktar(duzen_id, yeni_miktar)
            YAZ "Miktar güncellendi."
        DEGILSE
            YAZ "Yetersiz stok. İşlem iptal edildi."
        SON
    DEGILSE EGER secim = "Sil" ISE
        YAZ "Hangi ürün ID'si silinecek?"; OKU sil_id
        sepet_urunu_sil(sil_id)
        YAZ "Ürün silindi."
    SON

    YAZ "İndirim kodu uygulamak ister misiniz? (E/H)"
    OKU indirim_secim
    EGER indirim_secim = "E" ISE
        YAZ "İndirim kodunu girin:"; OKU kod
        EGER kod_gecerli_mi(kod) ISE
            indirim_tutari ← kod_indirim_tutari(kod)
            YAZ "İndirim uygulandı: ", indirim_tutari, " TL"
            indirim_uygulandi ← TRUE
        DEGILSE
            YAZ "Geçersiz indirim kodu."
            indirim_uygulandi ← FALSE
        SON
    SON

    /* Toplam hesaplama */
    toplam ← sepet_toplam()
    EGER indirim_uygulandi ISE
        toplam ← toplam - indirim_tutari
    SON

    /* Minimum 50 TL kontrolü */
    EGER toplam < 50 ISE
        YAZ "Sepet toplamı 50 TL'den az. Sipariş oluşturulamaz."
        YAZ "Daha fazla ürün eklemek ister misiniz? (E/H)"
        OKU sepet_islem
        SON
    SON

    /* Kargo hesaplama */
    EGER toplam >= 200 ISE
        kargo ← 0
        YAZ "Kargo: Ücretsiz (Toplam ≥ 200 TL)"
    DEGILSE
        kargo ← kargo_hesapla(toplam)
        YAZ "Kargo ücreti: ", kargo, " TL"
    SON

    /* Ödeme toplamı */
    odenecek ← toplam + kargo
    YAZ "Ödenecek tutar: ", odenecek, " TL"

    /* Ödeme yöntemi seçimi */
    YAZ "Ödeme yöntemi seçiniz: (Kart/Havale/Kapıda)"
    OKU odeme_yontemi

    EGER odeme_yontemi = "Kart" ISE
        YAZ "Kart bilgilerini giriniz..."
        OKU kart_no; OKU son_kullanma; OKU cvv
        EGER kart_onayla(kart_no, son_kullanma, cvv, odenecek) ISE
            odeme_durumu ← "Başarılı"
        DEGILSE
            odeme_durumu ← "Başarısız"
        SON
    DEGILSE EGER odeme_yontemi = "Havale" ISE
        YAZ "Banka bilgileri gösterildi. Havale yapıldıktan sonra onay beklenir."
        odeme_durumu ← "Beklemede"
    DEGILSE
        YAZ "Kapıda ödeme seçildi."
        odeme_durumu ← "Beklemede"
    SON

    /* Sipariş onayı */
    EGER odeme_durumu = "Başarılı" ISE
        siparis_id ← siparis_olustur(kullanici_adi, sepet, odenecek)
        YAZ "Ödeme başarılı. Sipariş onaylandı. Sipariş ID: ", siparis_id
        sepeti_temizle()
        sepet_islem ← "H" /* döngüyü bitir */
    DEGILSE
        YAZ "Ödeme başarılı değil/henüz onaylanmadı. İşlemi tekrar etmek ister misiniz? (E/H)"
        OKU sepet_islem
    SON

SON /* sepet_islem döngüsü bitiş */

YAZ "İşlem tamamlandı. Hoşçakalın."

BITIR
