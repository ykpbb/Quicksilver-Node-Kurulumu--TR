# Quicksilver-Node-Kurulumu--TR

Explorer: https://quicksilver.explorers.guru/

# SİSTEM GEREKSİNİMLERİ
## Minimum Sistem Gereksinimleri
- 3x CPU; saat hızı ne kadar yüksek olursa o kadar iyi
- 4GB RAM
- 80GB Disk
- Kalıcı İnternet bağlantısı (testnette minimum 10Mbps trafik olacak - üretim için en az 100Mbps bekleniyor)

## Önerilen Sistem Gereksinimleri
- 4x CPU; saat hızı ne kadar yüksek olursa o kadar iyi
- 8GB RAM
- 200GB DİSK (SSD veya NVME)
- Kalıcı İnternet bağlantısı (testnette minimum 10Mbps trafik olacak - üretim için en az 100Mbps bekleniyor)

# MANUEL NODE KURULUMU
Full node kurulumu için aşağıdaki yönergeleri adım adım takip edin.

## Değişkenleri Ayarlama
Buraya, gezginde görünecek olan takma adınızı (doğrulayıcı) girmelisiniz.
```
NODENAME=<MONIKER_ADINIZ>
```
Değişkenleri sisteme kaydedin ve içe aktarın.
```
QUICKSILVER_PORT=11
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export QUICKSILVER_CHAIN_ID=killerqueen-1" >> $HOME/.bash_profile
echo "export QUICKSILVER_PORT=${QUICKSILVER_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Paketleri Güncelleyin
```
sudo apt update && sudo apt upgrade -y
```

## Bağımlılıkları Yükleyin
```
sudo apt install curl build-essential git wget jq make gcc tmux -y
```

## Go Yükleyin
```
ver="1.18.2"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
go version
```

## İkili Dosyaları İndirin ve Oluşturun
```
cd $HOME
rm quicksilver -rf
git clone https://github.com/ingenuity-build/quicksilver.git --branch v0.4.0
cd quicksilver
make build
sudo chmod +x ./build/quicksilverd && sudo mv ./build/quicksilverd /usr/local/bin/quicksilverd
```

## Uygulamayı Yapılandırın
```
quicksilverd config chain-id $QUICKSILVER_CHAIN_ID
quicksilverd config keyring-backend test
quicksilverd config node tcp://localhost:${QUICKSILVER_PORT}657
```

## Uygulamayı Başlatın
```
quicksilverd init $NODENAME --chain-id $QUICKSILVER_CHAIN_ID
```

## Genesis ve Addrbook İndirin
```
wget -O ~/.paloma/config/genesis.json https://raw.githubusercontent.com/palomachain/testnet/master/paloma-testnet-5/genesis.json
wget -O ~/.paloma/config/addrbook.json https://raw.githubusercontent.com/palomachain/testnet/master/paloma-testnet-5/addrbook.json
```

## Seeds ve Peers Ayarlayın
```
SEEDS="dd3460ec11f78b4a7c4336f22a356fe00805ab64@seed.killerqueen-1.quicksilver.zone:26656"
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.quicksilverd/config/config.toml
```

## Özel Bağlantı Noktalarını Ayarlayın
```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PALOMA_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PALOMA_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PALOMA_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PALOMA_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PALOMA_PORT}660\"%" $HOME/.paloma/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PALOMA_PORT}317\"%; s%^address = \":8080\"%address = \":${PALOMA_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PALOMA_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PALOMA_PORT}091\"%" $HOME/.paloma/config/app.toml
```

## Pruning Yapılandırın
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.quicksilverd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.quicksilverd/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.quicksilverd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.quicksilverd/config/app.toml
```

## Minimum Gas Fiyatını Ayarlayın
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0ugrain\"/" $HOME/.paloma/config/app.toml
```

## Prometheus Etkinleştirin
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.quicksilverd/config/config.toml
```

## Zincir Verilerini Sıfırlayın
```
quicksilverd tendermint unsafe-reset-all --home $HOME/.quicksilverd
```

## Servis Oluşturun
```
sudo tee /etc/systemd/system/quicksilverd.service > /dev/null <<EOF
[Unit]
Description=quicksilver
After=network-online.target

[Service]
User=$USER
ExecStart=$(which quicksilverd) --home $HOME/.quicksilverd start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Kaydolun ve Hizmeti Başlatın
```
sudo systemctl daemon-reload
sudo systemctl enable palomad
sudo systemctl restart palomad && sudo journalctl -u palomad -f -o cat
```


