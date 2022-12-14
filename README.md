# Nolus-Rila Testnet
Nolus testnet kurulum rehberi.





### sistem gereksinimler (ekibin tavsiye ettiği minimum gereksinimler)
```
2+ vCPU
4+ GB RAM
120+ GB SSD
```
### sunucuyu güncelleme ve gerekli kütüphanelerin yüklenmesi
```
sudo apt update && sudo apt upgrade -y
```
```
sudo apt install curl tar wget tmux htop net-tools clang pkg-config libssl-dev jq build-essential git make ncdu -y
```
### go kurulumu
```
cd $HOME
wget -O go1.18.4.linux-amd64.tar.gz https://golang.org/dl/go1.18.4.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.4.linux-amd64.tar.gz && rm go1.18.4.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
go version
```
### binary dosyası kurulumu
```
git clone https://github.com/Nolus-Protocol/nolus-core
```
```
cd nolus-core
```
```
git checkout v0.1.39
```
```
make install
```


### initalize
* monikerismi belirlemeyi unutmayın
```
nolusd init monikerismi
```
### genesis dosyası
```
wget https://raw.githubusercontent.com/Nolus-Protocol/nolus-networks/main/testnet/nolus-rila/genesis.json
```
```
mv ./genesis.json ~/.nolus/config/genesis.json
```
### minimum gas price
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025unls\"/" $HOME/.nolus/config/app.toml
```
### peer ekleme
```
PEERS="56cee116ac477689df3b4d86cea5e49cfb450dda@54.246.232.38:26656,56f14005119e17ffb4ef3091886e6f7efd375bfd@34.241.107.0:26656,7f26067679b4323496319fda007a279b52387d77@63.35.222.83:26656,7f4a1876560d807bb049b2e0d0aa4c60cc83aa0a@63.32.88.49:26656,3889ba7efc588b6ec6bdef55a7295f3dd559ebd7@3.249.209.26:26656,de7b54f988a5d086656dcb588f079eb7367f6033@34.244.137.169:26656"
```
```
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.nolus/config/config.toml
```

### pruning 
* bu kısım opsiyonel, daha az disk alanı kaplaması için bunu yapabilirsiniz ama işlemci ve ram daha çok çalışacağını unutmayın.

```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.nolus/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.nolus/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.nolus/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.nolus/config/app.toml
```
### indexeri kapatmak
* bu da opsiyonel, pruning gibi
```
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.nolus/config/config.toml
```

### service dosyası
```
sudo tee /etc/systemd/system/nolusd.service > /dev/null <<EOF
[Unit]
Description=nolus node
After=network.target
[Service]
User=root
Type=simple
ExecStart=$(which nolusd) start --home $HOME/.nolus
Restart=on-failure
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
### servisi aktif etme ve başlatma
```
systemctl daemon-reload
```
```
systemctl enable nolusd
```
```
systemctl start nolusd
```
### service durumu
```
systemctl status nolusd
```
### servisi yeniden başlatmak için
```
systemctl restart nolusd
```
### log görüntülemek için
```
journalctl -u nolusd -fo cat
```
### validatör oluşturmak için nodun ağ ile senkronize olmasını beklemek gerekiyor
### güncel blok yüksekliğini takip etmek için: [explorer](https://explorer-rila.nolus.io/nolus-rila)
### senkronizasyon durumu için
```
nolusd status 2>&1 | jq .SyncInfo
```
* bu komutta `"catching_up": false` çıktısı senkronize olduğunu gösterir
### cüzdan oluşturma
* cüzdanismini belirlemeyi unutmayın
```
nolusd keys add cüzdanismi
```
### cüzdanınız varsa recover etme
```
nolusd keys add cüzdanismi --recover
```
* [faucet](https://faucet-rila.nolus.io/)ten token almayı unutmayın
* cüzdanınızdaki bakiyeyi öğrenmek için:
```
nolusd query bank balances cüzdanadresi
```
### senkronize olduysanız ve tokeniniz varsa validatör uluşturabilirsiniz
* komutta `cüzdanismi`ve `monikerismi` değiştirmeyi unutmayın.
```
nolusd tx staking create-validator \
--amount 95000000unls \
--from cüzdanismi \
--commission-max-change-rate "0.01" \
--commission-max-rate "0.2" \
--commission-rate "0.07" \
--min-self-delegation "1" \
--pubkey $(nolusd tendermint show-validator) \
--moniker monikerismi \
--chain-id nolus-rila \
--fees 550unls
```
* çıktıda verdiği txhashi explorerda aratarak bakabilirsiniz.
### node silmek için
```
sudo systemctl stop nolusd
sudo systemctl disable nolusd
rm -rf /etc/systemd/system/nolusd.service
sudo rm -rf $(which nolusd)
sudo rm -rf $HOME/.nolus
sudo rm -rf $HOME/nolus
```
## yararlı komutlar için: [link](https://forum.rues.info/index.php?threads/cosmos-aglarinda-kullanilan-ortak-komutlar.2540/)



[Explorer](https://explorer-rila.nolus.io/nolus-rila)

[Türkçe Telegram](https://t.me/+PK3xWbHLTX44ZTdk)

[Discord](https://discord.gg/nolus-protocol)

[Twitter](https://twitter.com/nolusprotocol)

[Telegram](https://t.me/NolusProtocol)

[Resmi Döküman](https://docs-nolus-protocol.notion.site/Run-a-Node-58c9af73bf5945988e902b4b8741f918)




























