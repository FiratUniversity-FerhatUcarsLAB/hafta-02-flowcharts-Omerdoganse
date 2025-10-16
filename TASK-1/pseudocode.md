BASLA

DONGU kart_takili_oldugu_surece TEKRARLA
    YAZ "Lütfen PIN kodunuzu giriniz:"
    hatali_sayac ← 0

    DONGU hatali_sayac < 3 ISE
        OKU girilen_pin
        EGER girilen_pin = dogru_pin ISE
            CIKIS DONGU
        DEGILSE
            hatali_sayac ← hatali_sayac + 1
            YAZ "Hatalı PIN! Kalan deneme: ", 3 - hatali_sayac
        SON
    SON

    EGER hatali_sayac = 3 ISE
        YAZ "Kart bloke edildi!"
        BITIR
    SON

    bakiye ← 5000
    gunluk_limit ← 2000

    DONGU islem_tekrari = "E" ISE
        YAZ "Bakiyeniz: ", bakiye
        YAZ "Çekmek istediğiniz tutarı giriniz:"
        OKU tutar

        EGER tutar MOD 20 ≠ 0 ISE
            YAZ "Tutar 20 TL'nin katı olmalıdır."
        DEGILSE EGER tutar > bakiye ISE
            YAZ "Yetersiz bakiye!"
        DEGILSE EGER tutar > gunluk_limit ISE
            YAZ "Günlük limit aşıldı!"
        DEGILSE
            bakiye ← bakiye - tutar
            YAZ tutar, " TL verildi."
            YAZ "Yeni bakiye: ", bakiye
            YAZ "Fiş veriliyor..."
        SON

        YAZ "Başka işlem yapmak ister misiniz? (E/H)"
        OKU islem_tekrari
    SON

    YAZ "Kart iade edildi. Teşekkürler!"

SON DONGU

BITIR
