# <h1 align="center"> Nibiru Testnet Node Turkish </h1> 
![nibi-logo-on-white-pink](https://user-images.githubusercontent.com/91866065/208937132-0f1e2186-0967-4f9e-aee7-a1c9f722cad7.png)

## Linkler:
 * [Nibiru Resmi Twitter](https://twitter.com/NibiruChain)
 * [Nibiru Resmi Discord](https://discord.gg/nibiru)
 * [Nibiru Explorer](https://testnet-2.nibiru.fi/)
 
## Minimum sistem gereksinimleri

* 4 cores (max. clock speed possible)

* 16GB RAM

* 500GB+ of NVMe or SSD disk

## Başlangıç
```
sudo su
cd
sudo apt update && sudo apt upgrade -y
sudo apt-get install curl gcc make jq screen -y
screen -S nibiru
```
#### * İşimiz bittiğinde screen'den çıkmak için CTRL+A+D tuş kombinasyonlarını kullanın. Screen'e tekrar girmek için `screen -r nibiru` kodunu kullanın.

## Go'yu yükleyin
```
wget https://golang.org/dl/go1.19.3.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.3.linux-amd64.tar.gz
rm go1.19.3.linux-amd64.tar.gz
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
```

#### * Go versiyonunu kontrol edin. (Çıktı:`go version go1.19.3 linux/amd64`)
```
go version
```
![goversion](https://user-images.githubusercontent.com/91866065/208239917-629f76d2-419f-4372-a933-4c8f1b63ba54.png)

## Nibiru'yu yükleyin
```
git clone https://github.com/NibiruChain/nibiru && cd nibiru
git fetch origin --tags
git checkout v0.16.2
make install
```

#### * Nibiru'nun versiyonunu kontrol edin. (Çıktı:`0.16.2`)
```
nibid version
```
![nibid](https://user-images.githubusercontent.com/91866065/208937500-91a9e253-539a-449a-aca7-12df2e9b4d55.png)

## Node'u başlatmak için gerekli adımlar

#### * <> işaretlerini kaldırın.
```
nibid init <NODEADI> --chain-id nibiru-testnet-2
```

#### * Yeni cüzdan oluşturun veya eski cüzdanınızı kurtarın.
```
nibid keys add <CUZDANADI>
nibid keys add <CUZDANADI> --recover
```

#### * Genesis dosyasını yükleyin ve ikinci komut ile çıktıyı kontrol edin. (Çıktı:`5cedb9237c6d807a89468268071647649e90b40ac8cd6d1ded8a72323144880d`)
```
curl -s https://rpc.testnet-2.nibiru.fi/genesis | jq -r .result.genesis > $HOME/.nibid/config/genesis.json
shasum -a 256 $HOME/.nibid/config/genesis.json
```

#### * Minimum gas değerini ayarlayın.
```
sed -i 's/minimum-gas-prices =.*/minimum-gas-prices = "0.025unibi"/g' $HOME/.nibid/config/app.toml
```

#### * Seed node'ları tanımlayın.
```
sed -i 's|seeds =.*|seeds = "'$(curl -s https://networks.testnet.nibiru.fi/nibiru-testnet-2/seeds)'"|g' $HOME/.nibid/config/config.toml
```

#### * Servis dosyasını oluşturalım.
```
sudo tee  /etc/systemd/system/nibid.service /dev/null <<EOF
[Unit]
Description=Nibiru Service
After=network.target
[Service]
Type=simple
User=$USER
WorkingDirectory=$HOME
ExecStart=$HOME/go/bin/nibid start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

#### * Node'u başlatın. journalctl ekranından çıkmak için CTRL+C basın.
```
sudo systemctl daemon-reload && systemctl enable nibid && sudo systemctl restart nibid && journalctl -o cat -fu nibid
```

#### * Senkronize durumunu kontrol edin. `"catching_up": false` çıktısını alana kadar diğer adıma geçmeyin.
```
nibid status 2>&1 | jq .SyncInfo
```

#### * Faucet'ten token talep edin.
```
FAUCET_URL="https://faucet.testnet-2.nibiru.fi/"
ADDR=<ADRESINIZ>
curl -X POST -d '{"address": "'"$ADDR"'", "coins": ["10000000unibi","100000000000unusd"]}' $FAUCET_URL
```

#### * Validator'u oluşturun. (Validator'u oluşturmadan önce cüzdanınızda NIBI bulunmalı)
```
nibid tx staking create-validator \
--amount 10000000unibi \
--commission-max-change-rate "0.1" \
--commission-max-rate "0.20" \
--commission-rate "0.0" \
--min-self-delegation "1" \
--website="https://github.com/mulosbron/NibiruTestnetNodeTurkish" \
--pubkey=$(nibid tendermint show-validator) \
--moniker <NODEADI> \
--chain-id nibiru-testnet-2 \
--from <CUZDANADI> \
-y
```

#### * Son olarak yukarıdaki komutun verdiği hash'i explorer'da aratın. Sonuç `success` ise olmuştur.

# <h1 align="center">[Mulosbron's Validator]() </h1>
