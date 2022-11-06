# Görev 3: Create Your Value Contract
**Kendi kontratınızı oluşturma**

## Resmi Dokümanlar
  * [Inery kontrat yazma](https://docs.inery.io/docs/category/contract-write)
  * [Inery CRUD kontrat](https://docs.inery.io/docs/category/create-crud-contract)

## Root Yetkisi Alma
```
sudo su
cd
```

## Değişkenleri Yükleme
```
source .bashrc
```

## Kullanıcı Adını Değişkene Atama
`HESAP_ADINIZ` Kullanıcı panelinizde yazan hesap adınızı yazıyorsunuz.
```
IneryAccname=HESAP_ADINIZ 
```

## inery.cdt Aracını İndirme
```
git clone https://github.com/inery-blockchain/inery.cdt
```
## Geçici Değişkenleri Yükleme
```
export PATH=$PATH:$HOME/inery.cdt/bin/
```

## incrud Dosyası Oluşturma
```
mkdir -p inrcrud
```

## Kontrat Yazma
```
tee inrcrud/inrcrud.cpp >/dev/null <<EOF
#include <inery/inery.hpp>
#include <inery/print.hpp>
#include <string>

using namespace inery;

using std::string;

class [[inery::contract]] inrcrud : public inery::contract {
  public:
    using inery::contract::contract;


        [[inery::action]] void create( uint64_t id, name user, string data ) {
            records recordstable( _self, id );
            auto existing = recordstable.find( id );
            check( existing == recordstable.end(), "record with that ID already exists" );
            check( data.size() <= 256, "data has more than 256 bytes" );

            recordstable.emplace( _self, [&]( auto& s ) {
               s.id         = id;
               s.owner      = user;
               s.data       = data;
            });

            print( "Hello, ", name{user} );
            print( "Created with data: ", data );
        }

         [[inery::action]] void read( uint64_t id ) {
            records recordstable( _self, id );
            auto existing = recordstable.find( id );
            check( existing != recordstable.end(), "record with that ID does not exist" );
            const auto& st = *existing;
            print("Data: ", st.data);
        }

        [[inery::action]] void update( uint64_t id, string data ) {
            records recordstable( _self, id );
            auto st = recordstable.find( id );
            check( st != recordstable.end(), "record with that ID does not exist" );


            recordstable.modify( st, get_self(), [&]( auto& s ) {
               s.data = data;
            });

            print("Data: ", data);
        }

            [[inery::action]] void destroy( uint64_t id ) {
            records recordstable( _self, id );
            auto existing = recordstable.find( id );
            check( existing != recordstable.end(), "record with that ID does not exist" );
            const auto& st = *existing;

            recordstable.erase( st );

            print("Record Destroyed: ", id);

        }

  private:
    struct [[inery::table]] record {
       uint64_t        id;
       name     owner;
       string          data;
       uint64_t primary_key()const { return id; }
    };

    typedef inery::multi_index<"records"_n, record> records;
 };
EOF
```

## Kontrat Kodunu Derleme
```
inery-cpp inrcrud/inrcrud.cpp -o inrcrud/inrcrud.wasm
```

# Kontratı Yayınlama

## Cüzdan Kilidini Açma
Aşağıdaki koddan sonra size şifrenizi soracak. Şifrenizi yazdığınızda gözükmez.
```
cline wallet unlock -n CUZDAN_ADINIZ
```

## Kontratı Ayarlama
```
cline set contract $IneryAccname ./inrcrud
```

## Kontrat Oluşturma 
```
cline push action $IneryAccname create "[1, $IneryAccname, My first Record]" -p $IneryAccname -j
```

## Kontratı Okuma
```
cline push action $IneryAccname read [1] -p $IneryAccname -j
```

## Kontratı Güncelleme
```
cline push action $IneryAccname update '[ 1,  "My first Record Modified"]' -p $IneryAccname -j
```

## Kontratı Yok Etme
```
cline push action $IneryAccname destroy [1] -p $IneryAccname -j
```


🔴 **İşlemleri gerçekleştirdikten sonra kullanıcı paneline giderek `Create Your Value Contract` başlıklı üçüncü görevi onaylayınız.***

