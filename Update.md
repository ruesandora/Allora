# Update



```console
# bu komut ile allora containerlerin idleri alıyoruz.
docker ps

docker stop İD
docker rm İD
# idleri düzenleyerek allora için olanları temizliyoruz.
```

<img width="427" alt="Ekran Resmi 2024-07-03 11 48 09" src="https://github.com/ruesandora/Allora/assets/101149671/d4c0c0fe-6382-4253-88a6-e591feb5748f">

#

Şimdi bizim Basic-coin-prediction-node dizinine girmemiz lazım - Rues'in repoda cd komutu ile ana dizine dönmeden Allora dizini içerisinde kurmuşsundur vb. ihtimaller  için 2 tarafıda ekledik.

Ya Allora-Chain dizininin içinde olacak Yada Ana dizinde bulunacak.

Terminalinize girdiniz. " ls " komutunu girdiniz eğer Basic-coin-prediction-node Dizini yoksa Allora-Chain dizininin içindedir. 

Eğer "ls" yaptığınızda Basic-coin-prediction-node dizini bulunuyorsa direkt dizine girebilirsiniz.


Basic-coin-prediction-node Dizinine Gir 

```console
cd basic-coin-prediction-node
```

-------

Eğer Ana Dizinde Yoksa Allora Chain Dizini üzerinden Basic-coin-prediction-node dizinine girin.

Allora-Chain Dizinine Gir
#

```console
cd allora-chain 
```

Basic-coin-prediction-node Dizinine Gir 

```console
cd basic-coin-prediction-node
```

------------

Docker-compose.yml Dosyasının İçine Girelim

```console
nano docker-compose.yml 
```

![image](https://github.com/RPCdotcom/Update/assets/141464235/e81d4ce7-9a61-406e-b7b8-d8ee7e32752f)

Buradaki TimeOut Sizde 5'dir bunu 10'a çıkaralım. 

![image](https://github.com/RPCdotcom/Update/assets/141464235/9aaa8e6f-6e70-4393-af1b-a2d450b9bd12)

Burayıda düzenleyelim 
```console
--topic=allora-topic-1-worker \
```
connect refused hatası alanlar sonuna \ koymayı unutmayın
Böyle Gözükecek : 

![image](https://github.com/RPCdotcom/Update/assets/141464235/e1c96158-7060-4a4b-9379-f661648a1312)

CTRL X - CTRL Y - Enter İle Kaydedelim.

Containerleri Sildik - Geri Buildleyelim 
```console
docker compose build
docker compose up -d
```

Kontrol Edelim : 

```console
curl --location 'http://localhost:6000/api/v1/functions/execute' \
--header 'Content-Type: application/json' \
--data '{
    "function_id": "bafybeigpiwl3o73zvvl6dxdqu7zqcub5mhg65jiky2xqb4rdhfmikswzqm",
    "method": "allora-inference-function.wasm",
    "parameters": null,
    "topic": "1",
    "config": {
        "env_vars": [
            {
                "name": "BLS_REQUEST_PATH",
                "value": "/api"
            },
            {
                "name": "ALLORA_ARG_PARAMS",
                "value": "ETH"
            },
            {
                "name": "ALLORA_BLOCK_HEIGHT_CURRENT",
                "value": "null"
            }
        ],
        "number_of_nodes": -1,
        "timeout": 10
    }
}' | jq
```
Almanız Gereken Sonuç: 
```console
{
  "code": "200",
  "request_id": "03001a39-4387-467c-aba1-c0e1d0d44f59",
  "results": [
    {
      "result": {
        "stdout": "{\"value\":\"2564.021586281073\"}",
        "stderr": "",
        "exit_code": 0
      },
      "peers": [
        "12D3KooWG8dHctRt6ctakJfG5masTnLaKM6xkudoR5BxLDRSrgVt"
      ],
      "frequency": 100
    }
  ],
  "cluster": {
    "peers": [
      "12D3KooWG8dHctRt6ctakJfG5masTnLaKM6xkudoR5BxLDRSrgVt"
    ]
  }
}
```

Şimdi Sadece 200 Aldık Diye bitmiyor - Loglarda Güzel Devam etmeli
```console
docker ps 
```

Buradan node-worker Container ID Aldın 
```console
docker logs -f id
```

Loglar bir süre sonra akmaya başlayacak şunu gibi görünecek : 

![image](https://github.com/RPCdotcom/Update/assets/141464235/d93434f4-7210-4d43-8431-fd86ee7ac279)
