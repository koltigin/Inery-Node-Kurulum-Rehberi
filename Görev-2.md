# Görev 2: Make your own currency and transfer to someone
**Kendi tokeninizi oluşturma ve başkalarına gönderme**

## Root Yetkisi Alma
```
sudo su
cd
```

## Değişkenleri Yükleme
```
source .bashrc
```

## Cüzdan Kilidini Açma
Aşağıdaki koddan sonra size şifrenizi soracak. Şifrenizi yazdığınızda gözükmez.
```
cline wallet unlock -n CUZDAN_ADINIZ
```

## token.wasm ve token.abi Dosyalarını Oluşturma
Root dizininde iki dosya oluşturuyoruz.
```
cline get code inery.token -c token.wasm -a token.abi --wasm
```

## token.wasm Dosyasını Hesabımızla İlişkilendirme
```
cline set code -j YourAccountName token.wasm
```

## token.abi Dosyasını Hesabımızla İlişkilendirme
token.wasm dosyasını
```
cline set abi YourAccountName token.abi
```

## Token Oluşturma
`HESAP_ADINIZ`hesap adınızı yazıyorsunuz.
`TOKEN_MIKTARI` token miktarını buna benzer bir şekilde yazıyorsunuz; 555000.0000
`TOKEN_KODU` buraya da tokeninizin adını yazıyorsunuz. Türkçe karakter kullnmayınız.
`TOKEN_ACIKLAMASI` bu bölüme Türkçe karakter kullanmadan açıklama yazabilirsiniz.

```
cline push action inery.token create '["HESAP_ADINIZ", "TOKEN_MIKTARI TOKEN_KODU"], "TOKEN_ACIKLAMASI"]'-p HESAP_ADINIZ
```
Aşağıdaki örnekteki gibi yazıyoruz:
```
cline push action inery.token create '["mehmet", "5333000.0000 MHMT" , "creating my first tokens"]' -p mehmet
```

## Token Basma
```
cline push action inery.token issue '["HESAP_ADINIZ", "TOKEN_MIKTARI TOKEN_KODU", "TOKEN_BASIM_ACIKLAMASI"]' -p HESAP_ADINIZ
```

Aşağıdaki örnekteki gibi yazıyoruz:
```
cline push action inery.token issue '["mehmet", "5333000.0000 MHMT", "Issuing some MHMT token"]' -p mehmet
```

## Token Transfer Etme
```
cline push action inery.token transfer '["HESAP_ADINIZ", "GONDERILECEK_HESAP_ADI", "MIKTAR TOKEN_KODU", "MESAJINIZ"]' -p HESAP_ADINIZ
```
Aşağıdaki örnekteki gibi yazıyoruz:
```
cline push action inery.token transfer '["mehmet", "inery", "500.0000 MHMT", "Sana 500 MHMT tokeni gonderiyorum"]' -p mehmet
```

🔴 **`inery` hsabı dahil 10 farklı kişiye tokenlerimizi gönderiyoruz.**

🔴 **Gönderim işlemini gerçekleştirdikten sonra kullanıcı paneline giderek `Make your own currency and transfer to someone` başlıklı ikinci görevi onaylayınız.***

