# Umee Mainnet full node setup guide

1. Update the system:
```
sudo apt update && sudo apt upgrade --yes
```
2. Install core tools:
```
sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils htop net-tools lsof --yes
```
3. Install Go:
```
cd $HOME
wget -c -O go1.20.4.linux-amd64.tar.gz https://golang.org/dl/go1.20.4.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.20.4.linux-amd64.tar.gz && sudo rm go1.20.4.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
sudo cp $(which go) /usr/local/bin
go version
```
4. Clone the Umee software:
```
cd $HOME
git clone https://github.com/umee-network/umee.git
cd umee
git pull
```
5. Checkout to the actual tag:
```
git checkout v4.4.2
```
6. Build the software and copy ``umeed`` binary file to ``/usr/local/bin``:
```
make build
sudo cp $HOME/umee/build/umeed /usr/local/bin
umeed version
```
7. Initialize ``umeed``:
```
umeed init <your_moniker> --chain-id umee-1 
```
8. Get the genesis file:
```
wget -O $HOME/.umee/config/genesis.json https://github.com/umee-network/mainnet/raw/main/genesis.json
```
9. Add peers to ``$HOME/.umee/config/config.toml``:
```
peers="06169345b4ae8a59f5132bae78a63733767dc952@51.195.60.123:26656,8b6baf477cd6c5fde18573a57767e0bb0083a8ce@116.202.36.138:26656,f00230b900b2e03a0ebfb0cec024bc0229f4043f@135.181.223.194:26656,31c2b4851604cb0f88909116bc2029b2af392767@194.163.166.56:26656,e324ca5fad08769325921ed042b76bdb1df41e12@162.55.131.220:26656,4720fe172f90026e72723c38d75f4f20611bc792@88.198.70.2:26656,7d2b275cea5dc30a90c9657220b2ef9cf02dfe87@157.90.179.182:26656,d9c0fc2da0bf7b22b92f3cd89b4e98ff089fe446@65.21.132.226:56656,ae41472c094737bef61450c11f1b4978c0a3550d@18.144.151.186:26656,f6b22c8d26370afd0b3e5e78697e19f7a2fb8c73@144.217.74.27:26656,d0659fc256c3e6f99def7a7b16500097065a67e9@195.201.170.172:26656,5ec673b49eea3198f7c0df0782d62e0b7a7d5b9f@51.195.60.117:26656,cce3ded2638edcaf804e4fa18a4a988cd19e9ee1@148.251.152.54:26656,66377bf9c7d2106f8fb2814d105b934e2cf9bde8@78.46.66.6:26656,6dfab3a8a1d692c6270758757cb2026005a10622@65.108.106.252:26656,b7c7e560f13988dc00c6892c813ff6c459521917@44.231.119.182:26656,60349afbb66bfa51d466a1807b6034c8a8446b41@34.215.214.32:26656,96391162797cbdf10982cda8866913be471fbdd4@44.230.43.94:26656,9f86f8acfa46ac5380796328fe0d7daff5038f56@3.37.216.115:26656,629ce04f882462999de6791b0c4010dba5dafaaf@142.132.201.53:26656,77F54319D6F62C17036CA71B3F88365F652BF79F@169.197.142.149:26656,912b7279934187f8c94eacdc21a2e0bdee245eef@54.241.232.181:26656,94a928e1f5ebbc5fae12400c7d8bbdad8b197ad2@52.79.49.253:26656,870c0a786dc941f8ebecd2772c41c014b6cf8899@51.210.118.65:26656,47dd32dc5aa926ff76d8e53a4bc1fcf596cb254c@38.242.205.238:26656,efbcd2de6981fa7f692771e1b845c780c310e2fe@176.9.17.230:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.umee/config/config.toml
```
10. Configure the background service:
```
sudo tee /etc/systemd/system/umeed.service > /dev/null <<'EOF'
[Unit]
Description=Umeed daemon
After=network-online.target

[Service]
User=root
ExecStart=($which umeed) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
11.  Enable service and start your node:
```
sudo systemctl daemon-reload
sudo systemctl enable umeed
sudo systemctl restart umeed
journalctl -u umeed -f -o cat
```
12. If you want to sync your full node quickly you can use state sync guide here: https://github.com/mediumwe11/umee/blob/main/statesync.md
