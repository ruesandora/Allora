<h1 align="center">Allora</h1>

> Gm, yeni bir Node Point > All ile buluşuyoruz.

> Allora'nın worker/price bot'unu kuruyoruz bugün.

> Benzer içerikler sohbetler ve güncellemler için [telegramdayız](https://t.me/RuesAnnouncement).

<details>
  <summary> <h1> Yatırımı hakkında </summary> </h1>

![image](https://github.com/ruesandora/Allora/assets/101149671/2e54ddbb-d33a-4165-9497-78417dc1c523)

</details>

#

<h1 align="center">Donanım</h1>

> Açıkcası ben sunucu almadım, airchains station'umun içine kurdum.

> Yinede donanım olarak minimum `2 CPU 4 RAM` iyidir.

#

<h1 align="center">Kurulum</h1>

```console
# sunucu güncellememiz
sudo apt update & sudo apt upgrade -y

sudo apt install ca-certificates zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev curl git wget make jq build-essential pkg-config lsb-release libssl-dev libreadline-dev libffi-dev gcc screen unzip lz4 -y

sudo apt install python3
sudo apt install python3-pip

# Dockeri yükleyelim
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

VER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep tag_name | cut -d '"' -f 4)

curl -L "https://github.com/docker/compose/releases/download/"$VER"/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose

sudo groupadd docker
sudo usermod -aG docker $USER
```

#

```console
# go kurulumu
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.22.4.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> $HOME/.bash_profile
source .bash_profile
```

#

<h1 align="center">Allora ve Cüzdan kurulumu</h1>

```console
# allora kurulumu
git clone https://github.com/allora-network/allora-chain.git
cd allora-chain && make all

# cüzdan oluşturma
allorad keys add CÜZDAN_İSMİ
# cüzdanınıza bir isim veriniz
````


> Akabinde keplr'a 24 kelimenizi import ediniz 

> Allora ağını keplr'a [buradan](https://explorer.edgenet.allora.network/wallet/suggest) ekleyelim.

> Allora dashboard'a [buradan](https://app.allora.network?ref=eyJyZWZlcnJlcl9pZCI6IjBlNWRhMjlmLTc3YjItNDQ2NS1hYTcxLTk0NWI3NjRhMTA0ZiJ9) bağlanıyoruz.

> Allora cüzdanımıza token [buradan](https://faucet.edgenet.allora.network/) alıyoruz.

#

<h1 align="center">Workerlerin kurulumu</h1>

```console
cd $HOME
git clone https://github.com/allora-network/basic-coin-prediction-node

cd basic-coin-prediction-node

# dosyaları oluşturalım
mkdir worker-data
mkdir head-data

# dosya izinleri
sudo chmod -R 777 worker-data
sudo chmod -R 777 head-data

# head keyimizi oluşturalım
sudo docker run -it --entrypoint=bash -v ./head-data:/data alloranetwork/allora-inference-base:latest -c "mkdir -p /data/keys && (cd /data/keys && allora-keys)"

# worker keyimizi oluşturalım
sudo docker run -it --entrypoint=bash -v ./worker-data:/data alloranetwork/allora-inference-base:latest -c "mkdir -p /data/keys && (cd /data/keys && allora-keys)"
```

#

> Bu komut ile head keyimzi alacağız.

```console
cat head-data/keys/identity
```

> Çıktının sonu root kısmı ile birleşik gelebilir kopyalarken dikkat edelim.

> Keyimizi saklayalım lütfen.

```console
# docker-compose.yml temizleyeliim
rm -rf docker-compose.yml
```

#

<h1 align="center">Alloraya bağlanalım</h1>

```console
# yeni docker-compose.yml oluşturacağız alttaki komutla
nano docker-compose.yml
```

> Alttaki kod bloğunu direkt sunucumuza komple yapıştıralım

> ve henüz kaydetmeden bu reponun aşağısına gidelim.

```console
version: '3'

services:
  inference:
    container_name: inference-basic-eth-pred
    build:
      context: .
    command: python -u /app/app.py
    ports:
      - "8000:8000"
    networks:
      eth-model-local:
        aliases:
          - inference
        ipv4_address: 172.22.0.4
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/inference/ETH"]
      interval: 10s
      timeout: 10s
      retries: 12
    volumes:
      - ./inference-data:/app/data

  updater:
    container_name: updater-basic-eth-pred
    build: .
    environment:
      - INFERENCE_API_ADDRESS=http://inference:8000
    command: >
      sh -c "
      while true; do
        python -u /app/update_app.py;
        sleep 24h;
      done
      "
    depends_on:
      inference:
        condition: service_healthy
    networks:
      eth-model-local:
        aliases:
          - updater
        ipv4_address: 172.22.0.5

  worker:
    container_name: worker-basic-eth-pred
    environment:
      - INFERENCE_API_ADDRESS=http://inference:8000
      - HOME=/data
    build:
      context: .
      dockerfile: Dockerfile_b7s
    entrypoint:
      - "/bin/bash"
      - "-c"
      - |
        if [ ! -f /data/keys/priv.bin ]; then
          echo "Generating new private keys..."
          mkdir -p /data/keys
          cd /data/keys
          allora-keys
        fi
        # Change boot-nodes below to the key advertised by your head
        allora-node --role=worker --peer-db=/data/peerdb --function-db=/data/function-db \
          --runtime-path=/app/runtime --runtime-cli=bls-runtime --workspace=/data/workspace \
          --private-key=/data/keys/priv.bin --log-level=debug --port=9011 \
          --boot-nodes=/ip4/172.22.0.100/tcp/9010/p2p/head-id \
          --topic=allora-topic-1-worker \
          --allora-chain-key-name=testkey \
          --allora-chain-restore-mnemonic='24 KELIME BURAYA' \
          --allora-node-rpc-address=https://allora-rpc.edgenet.allora.network/ \
          --allora-chain-topic-id=1
    volumes:
      - ./worker-data:/data
    working_dir: /data
    depends_on:
      - inference
      - head
    networks:
      eth-model-local:
        aliases:
          - worker
        ipv4_address: 172.22.0.10

  head:
    container_name: head-basic-eth-pred
    image: alloranetwork/allora-inference-base-head:latest
    environment:
      - HOME=/data
    entrypoint:
      - "/bin/bash"
      - "-c"
      - |
        if [ ! -f /data/keys/priv.bin ]; then
          echo "Generating new private keys..."
          mkdir -p /data/keys
          cd /data/keys
          allora-keys
        fi
        allora-node --role=head --peer-db=/data/peerdb --function-db=/data/function-db  \
          --runtime-path=/app/runtime --runtime-cli=bls-runtime --workspace=/data/workspace \
          --private-key=/data/keys/priv.bin --log-level=debug --port=9010 --rest-api=:6000
    ports:
      - "6000:6000"
    volumes:
      - ./head-data:/data
    working_dir: /data
    networks:
      eth-model-local:
        aliases:
          - head
        ipv4_address: 172.22.0.100


networks:
  eth-model-local:
    driver: bridge
    ipam:
      config:
        - subnet: 172.22.0.0/24

volumes:
  inference-data:
  worker-data:
  head-data:
````

> sunuc içersinde ctrl w ile aratarak `head-id` ve `24 KELIME BURAYA` kısımlarını değiştirelim.

> head-id cat komutumuz ile almıştık, 24 kelimeyi cüzdanı oluştururken aldık.

> Akabinde CTRL X Y ENTER ile kaydetip çıkalım.

<h1 align="center">Workerlerimizi başlatma</h1>

```console
docker compose build
docker compose up -d
```

#

> `docker ps` ile aratıp node-worker'imizin container id'iyi alalım.


<img width="444" alt="Ekran Resmi 2024-06-28 12 59 20" src="https://github.com/ruesandora/Allora/assets/101149671/e69e844d-a3da-4b76-9ead-930ce087afb9">


> `docker ps` ile arattığınızda node-worker çıkmıyorsa `docker ps -a` ile deneyin.
#

> `docker logs -f container_id` komutunu düzenleyip aratalım.

<img width="674" alt="Ekran Resmi 2024-06-28 13 00 23" src="https://github.com/ruesandora/Allora/assets/101149671/389e73a4-9e5f-4701-9b79-d22b7e5654bb">

> Success çıktımızı aldıysak tebrikler.

#

> Allora puanlarımız her gün güncellenerek [buraya](https://app.allora.network?ref=eyJyZWZlcnJlcl9pZCI6IjBlNWRhMjlmLTc3YjItNDQ2NS1hYTcxLTk0NWI3NjRhMTA0ZiJ9) yansıyacaktır.



#

![image](https://github.com/ruesandora/Allora/assets/101149671/5c2f63b6-e831-4d9e-b141-e7edfc1af714)






