## Sistemi Güncelleme
```
sudo apt-get update && sudo apt install git && sudo apt install screen
```

## Gerekli Kütüphanelerin Kurulması
```
sudo apt-get install -y make bzip2 automake libbz2-dev libssl-dev doxygen graphviz libgmp3-dev \
autotools-dev libicu-dev python2.7 python2.7-dev python3 python3-dev \
autoconf libtool curl zlib1g-dev sudo ruby libusb-1.0-0-dev \
libcurl4-gnutls-dev pkg-config patch llvm-7-dev clang-7 vim-common jq libncurses5
```

## Inery Dosyalarının İndirilmesi ve Kurulması
```
git clone https://github.com/inery-blockchain/inery-node
cd inery-node/inery.setup
chmod +x ine.py
./ine.py --export
cd; source .bashrc; cd -
```

## Config Dosyasını Düzenleme
`inery-node/inery.setup/tools/` dizininde yer alan `config.json` dosyasını isterseniz winscp gibi bir programla ya da aşağıdaki kodla terminal üzerinden düzenleyebilirsiniz.

```
sudo nano tools/config.json
```

Açılan dosyada aşağıdaki yerleri kendinize göre dolduruyorusunuz.
  * `AccountName` hesap adınız
  * `PublicKey` Inery kullanıcı panelinizde sol blokta yer alan kod
  * `PrivateKey` yine Inery kullanıcı panelinizde sol blokta yer alan kod
  * `IP` ip adresiniz

```
"MASTER_ACCOUNT": {     
"NAME": "AccountName",     
"PUBLIC_KEY": "PublicKey",     
"PRIVATE_KEY": "PrivateKey",     
"PEER_ADDRESS": "IP:9010",     
"HTTP_ADDRESS": "0.0.0.0:8888",     
"HOST_ADDRESS": "0.0.0.0:9010" }
```
🔴 **Dosyamızı `ctrl x y enter` diyerek kaydediyoruz.**

## Node'u Başlatma
Inery adında bir screen açıyoruz ve master komutu ile node'u başlatıyoruz.
```
screen -S inery
./ine.py --master
```

🔴 **Bu aşamadan sonra screen'den `ctrl a d` diyerek çıkıyoruz ve snapshot yükleme adımına geçiyoruz.**

## Snaphot Yükleme :warning:
:rotating_light:	:warning: :warning: Aşağıdaki kodda `snapshot-` ile başlayıp `.bin`ile başlayan dosyayı bu [adresten](https://snapshot.inery.io/) kontrol edin. Gerekirse günceliyle değiştirin. :warning: :warning: :rotating_light:
```
cd
curl -L https://snapshot.inery.io/snaps/snapshot-0078027ea3001d094a20c4846c791fd0b586fecb6139094f7917bb34df12dcbf.bin > inery_snap.bin
chmod +x inery_snap.bin
sudo mv inery_snap.bin $HOME/inery-node/inery.setup/master.node/blockchain/data/snapshots/
```

Aşağıdaki kodları girerek `snapshots.sh` dosyasının içeriğini dolduruyoruz.
```
cd $HOME/inery-node/inery.setup/master.node/
nano snapshots.sh
```
Dosya içeriğini doldurmadan önce aşağıdaki kodu not defterine yapıştırıp değişmesi gereken yerlere kendi bilgilerinizi girin. Aşağıdaki kodda `#BURASI DUZENLENECEK` yazan yerleri düzenleme yaparken silersiniz.

* NODE_ADINIZ 
* DNS_IP_ADRESINIZ
* PUBLIC_KEY
* PRIVATE_KEY

```
#!/bin/bash
DATADIR="./blockchain"
if [ ! -d $DATADIR ]; then
    mkdir -p $DATADIR;
fi

nodine --snapshot $DATADIR"/data/snapshots/inery_snap.bin" \
--plugin inery::producer_plugin \
--plugin inery::producer_api_plugin \
--plugin inery::chain_plugin \
--plugin inery::chain_api_plugin \
--plugin inery::http_plugin \
--plugin inery::history_api_plugin \
--plugin inery::history_plugin \
--plugin inery::net_plugin \
--plugin inery::net_api_plugin \
--data-dir $DATADIR"/data" \
--blocks-dir $DATADIR"/blocks" \
--config-dir $DATADIR"/config" \
--access-control-allow-origin=* \
--contracts-console \
--http-validate-host=false \
--verbose-http-errors \
--p2p-max-nodes-per-host 100 \
--connection-cleanup-period 10 \
--master-name NODE_ADINIZ \ #BURASI DUZENLENECEK
--http-server-address 0.0.0.0:8888 \
--p2p-listen-endpoint DNS_IP_ADRESINIZ:9010 \ #BURASI DUZENLENECEK
--p2p-peer-address tas.blockchain-servers.world:9010 \
--signature-provider PUBLIC_KEY=KEY:PRIVATE_KEY \ #BURASI DUZENLENECEK
--p2p-peer-address sys.blockchain-servers.world:9010 \
--p2p-peer-address master1.blockchain-servers.world:9010 \
--p2p-peer-address master2.blockchain-servers.world:9010 \
--p2p-peer-address master3.blockchain-servers.world:9010 \
>> $DATADIR"/nodine.log" 2>&1 & \
echo $! > $DATADIR"/ined.pid"
```

```
chmod +x snapshots.sh
cd; source .bashrc; cd -
./snapshots.sh
```

# Node'u Başlatma ve Loglara Bakma 
Tekrar daha önce açtığımız screen içine giriyoruz.
```
screen -r inery
```
Ardından loglara bakıyoruz.
```
cd master.node/blockchain
tail -f nodine.log
```

🔴 **Bloklar eşitlenmeden diğer adımlara geçmiyoruz**

## **Görev 1:** Master Node Kaydetme

### Cüzdan Şifresi Oluşturma
`CUZDAN_ADINIZ` yazan yere Inery kullanıcı adımızı yazıyoruz. root dizininde oluşan bu dosya içerisinde cüzdan şifreniz oluşacak. 
```
cd;  cline wallet create --file CUZDAN_ADINIZ.txt
```
🔴 **Cüdan Adını Değiştiriyoruz.**
```
cd $HOME/inery-wallet
mv default.wallet CUZDAN_ADINIZ.wallet
```

### Cüzdan Kilidini Açma
Aşağıdaki koddan sonra size şifrenizi soracak. Sifreniz yukarıda oluşturduğumuz dosyanın içerisinde yer alıyor. Şifrenizi yazdığınızda gözükmez.
```
cline wallet unlock -n CUZDAN_ADINIZ
```

### Cüzdanımızı Import Ediyoruz
`ACCOUNT_PRIVATE_KEY` bölümüne panelimizde bulunan keyi yazıyoruz.
```
cline wallet import --private-key ACCOUNT_PRIVATE_KEY
```

### Hesap Kaydını Yapma
`ACCOUNT_NAME` hesap adınız.
`ACCOUNT_PUBLIC_KEY` kullanıcı panelinizde bulunuyor
`SERVER_IP_ADDRESS` server IP adresiniz.
```
cline master bind ACCOUNT_NAME ACCOUNT_PUBLIC_KEY SERVER_IP_ADDRESS:9010
```

### Hesap Onaylama
`ACCOUNT_NAME` hesap adınız.
```
cline master approve ACCOUNT_NAME
```

### Master Node'unuzu Kontrol Etme
[Buradaki](https://explorer.inery.io/) adresten adınızı aratınız. 
🔴 **Adınızı gördükten sonra kullanıcı panelinize giderek `Master Approval` başlıklı birinci görevi onaylayınız.**

# Notlar

## Cüzdan Kilidini Açma
Serverınıza bağlandığınızda herhangi bir işlem yapmadan önce aşağıdaki kodları kullanarak önce değişkenleri yükleyiniz yoksa cline not found uyarısı alır işlemlerinizi yapamazsınız sonrasında ise cüzdanınızın kilidini açınız. 
```
source .bashrcd
```
```
cline wallet unlock -n CUZDAN_ADINIZ
```

## Bakiye Kontrol Etme
`ACCOUNT_NAME` hesap adınız.
```
cline get currency balance inery.token ACCOUNT_NAME
```

## Node'u Silme
```
cd inery-node/inery.setup/master.node
./stop.sh
cd
rm inery-node -rf
rm inery-wallet -rf
pkill nodine
```


