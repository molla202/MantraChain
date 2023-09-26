
![image](https://github.com/molla202/MantraChain/assets/91562185/fd76840c-6f3f-4fdb-8650-58e84ae353fa)


### Update ve gereklilikleri kuralum
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```
### Go kurulumu
```
cd $HOME
! [ -x "$(command -v go)" ] && {
VER="1.19.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
}
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```
### Varyasyonları ayarlayalım
```
Not: isim yazılacak yerler var unutmayınız.
echo "export WALLET="cüzdan-adı-yaz"" >> $HOME/.bash_profile
echo "export MONIKER="moniker-adı-yaz"" >> $HOME/.bash_profile
echo "export MANTRA_CHAIN_ID="mantrachain-1"" >> $HOME/.bash_profile
echo "export MANTRA_PORT="28"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
### Dosyaları çekelim
```
cd $HOME
sudo wget -O /usr/lib/libwasmvm.x86_64.so https://github.com/CosmWasm/wasmvm/releases/download/v1.3.0/libwasmvm.x86_64.so
wget https://github.com/MANTRA-Finance/public/raw/main/mantrachain-testnet/mantrachaind-linux-amd64.zip
unzip mantrachaind-linux-amd64.zip
rm mantrachaind-linux-amd64.zip
mv mantrachaind $HOME/go/bin
```
### İnit işlemi
Not: adınızı yazmayı unutmayınız
```
mantrachaind config node tcp://localhost:${MANTRA_PORT}657
mantrachaind config keyring-backend os
mantrachaind config chain-id mantrachain-1
mantrachaind init "buraya-yazılacak" --chain-id mantrachain-1
```
```
### Genesis ve adressbook indirelim
wget -O $HOME/.mantrachain/config/genesis.json https://testnet-files.itrocket.net/mantra/genesis.json
wget -O $HOME/.mantrachain/config/addrbook.json https://testnet-files.itrocket.net/mantra/addrbook.json

### Seed ve peer ayarları
SEEDS="a9a71700397ce950a9396421877196ac19e7cde0@mantra-testnet-seed.itrocket.net:22656"
PEERS="1a46b1db53d1ff3dbec56ec93269f6a0d15faeb4@mantra-testnet-peer.itrocket.net:22656,9c1c4e34e90273f9e99f4c0ea319efd08f872510@167.235.14.83:32656,c533d7ee2037ee6d382f773be04c5bbf27da7a29@34.70.189.2:26656,a435339f38ce3f973739a08afc3c3c7feb862dc5@35.192.223.187:26656,114988f9a053f594ab9592beb79b924430d355ba@34.123.40.240:26656,0e687ef17922361c1aa8927df542482c67fb7571@35.222.198.102:26656,a9a71700397ce950a9396421877196ac19e7cde0@65.108.231.124:22656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.mantrachain/config/config.toml

### Port ayarları
sed -i.bak -e "s%:1317%:${MANTRA_PORT}317%g;
s%:8080%:${MANTRA_PORT}080%g;
s%:9090%:${MANTRA_PORT}090%g;
s%:9091%:${MANTRA_PORT}091%g;
s%:8545%:${MANTRA_PORT}545%g;
s%:8546%:${MANTRA_PORT}546%g;
s%:6065%:${MANTRA_PORT}065%g" $HOME/.mantrachain/config/app.toml

### Port ayarları
sed -i.bak -e "s%:26658%:${MANTRA_PORT}658%g;
s%:26657%:${MANTRA_PORT}657%g;
s%:6060%:${MANTRA_PORT}060%g;
s%:26656%:${MANTRA_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${MANTRA_PORT}656\"%;
s%:26660%:${MANTRA_PORT}660%g" $HOME/.mantrachain/config/config.toml

### Puring
sed -i -e "s/^pruning *=.*/pruning = \"nothing\"/" $HOME/.mantrachain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.mantrachain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.mantrachain/config/app.toml

### Gas ve prometeus ayarları
sed -i 's/minimum-gas-prices =.*/minimum-gas-prices = "0.0uaum"/g' $HOME/.mantrachain/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.mantrachain/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.mantrachain/config/config.toml
```
### Servis olusturalım
```
sudo tee /etc/systemd/system/mantrachaind.service > /dev/null <<EOF
[Unit]
Description=Mantra node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.mantrachain
ExecStart=$(which mantrachaind) start --home $HOME/.mantrachain
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
### Snap atalım...
```
mantrachaind tendermint unsafe-reset-all --home $HOME/.mantrachain
if curl -s --head curl https://testnet-files.itrocket.net/mantra/snap_mantra.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl https://testnet-files.itrocket.net/mantra/snap_mantra.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.mantrachain
    else
  echo no have snap
fi
```
### Servisi ve nodu başlatalım
```
sudo systemctl daemon-reload
sudo systemctl enable mantrachaind
sudo systemctl restart mantrachaind
sudo journalctl -u mantrachaind -fo cat
```
