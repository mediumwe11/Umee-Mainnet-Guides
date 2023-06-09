# Umee Mainnet state sync guide

1. Stop Umee background service and reset database:
```
systemctl stop umeed
umeed tendermint unsafe-reset-all --home $HOME/.umee --keep-addr-book
```
2. Use variables to connect to RPC node and set the number and the hash of the block for state sync:
```
PEER="d5f320c6e1443160c887deab487f7aa3830322ff@194.60.201.146:26656"
SNAP="http://194.60.201.146:26657"
LATEST_HEIGHT=$(curl -s $SNAP/block | jq -r .result.block.header.height)
TRUST_HEIGHT=$((LATEST_HEIGHT - 1000))
TRUST_HASH=$(curl -s "$SNAP/block?height=$TRUST_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP,$SNAP\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$TRUST_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.umee/config/config.toml
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEER\"/" $HOME/.umee/config/config.toml
```
3. Run Umee background service:
```
sudo systemctl start umeed
sudo journalctl -u umeed -f -o cat
```
