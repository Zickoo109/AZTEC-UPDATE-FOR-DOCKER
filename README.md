# AZTEC NODE UPDATE
## 1. install aztec tools
```bash
bash -i <(curl -s https://install.aztec.network)
```
```bash
echo 'export PATH="$HOME/.aztec/bin:$PATH"' >> ~/.bashrc

source ~/.bashrc
```
```bash
aztec-up
```
## 2. intall foundry
```bash
curl -L https://foundry.paradigm.xyz | bash
```
```bash
source ~/.bashrc
```
```bash
foundryup
```
## 3. Now run this script by hendrix
```bash
#!/bin/bash

clear
echo "pls provide your old validator info."
read -sp "   put your OLD Sequencer Private Key (will not be shown): " OLD_PRIVATE_KEY && echo
read -p "   put your sepolia rpc only: " ETH_RPC
echo "Goodd. Starting.." && echo " "

rm -f ~/.aztec/keystore/key1.json
echo ":BE READY to write down your private key both eth and BLS and your eth address."
read -p "   Press [Enter] to generate your new keys..."
aztec validator-keys new --fee-recipient 0x0000000000000000000000000000000000000000000000000000000000000000 && echo " " 
KEYSTORE_FILE=~/.aztec/keystore/key1.json
NEW_ETH_PRIVATE_KEY=$(jq -r '.validators[0].attester.eth' $KEYSTORE_FILE)
NEW_BLS_PRIVATE_KEY=$(jq -r '.validators[0].attester.bls' $KEYSTORE_FILE)
NEW_PUBLIC_ADDRESS=$(cast wallet address $NEW_ETH_PRIVATE_KEY)

echo "good! Your new keys are below. SAVE THIS INFO SECURELY!"
echo "   - NEW ETH Private Key: $NEW_ETH_PRIVATE_KEY"
echo "   - NEW BLS Private Key:  $NEW_BLS_PRIVATE_KEY"
echo "   - NEW Public Address:   $NEW_PUBLIC_ADDRESS"
echo " "

echo "You need to send 0.2 to 0.5 Sepolia eth to this new address:"
echo "   $NEW_PUBLIC_ADDRESS"
read -p "   After the funding tnx is confirmed, press [Enter] to continue.." && echo " "


echo "Approving STAKE spending..."
cast send 0x139d2a7a0881e16332d7D1F8DB383A4507E1Ea7A "approve(address,uint256)" 0xebd99ff0ff6677205509ae73f93d0ca52ac85d67 200000ether --private-key "$OLD_PRIVATE_KEY" --rpc-url "$ETH_RPC" && echo " "

echo "joining the testnet yey..."
aztec add-l1-validator \
  --l1-rpc-urls "$ETH_RPC" \
  --network testnet \
  --private-key "$OLD_PRIVATE_KEY" \
  --attester "$NEW_PUBLIC_ADDRESS" \
  --withdrawer "$NEW_PUBLIC_ADDRESS" \
  --bls-secret-key "$NEW_BLS_PRIVATE_KEY" \
  --rollup 0xebd99ff0ff6677205509ae73f93d0ca52ac85d67 && echo " "

echo "All done! u have successfully joined the new testnet now rerun your node with your new pvt and address"
```
**follow the onscreen instruction accordinly**
**your OLD Sequencer Private Key will not be shown after pasting but that is fine**
**save the new key that will be generated as you'll need it for the next step**

*for ETH RPC when the script ask for one: use `https://sepolia.drpc.org`* dont use this RPC to run your node. its only for registration purpose

### for those who run the node with docker

### i. Open 
```bash
nano .env
```
Replace `validator-Private-Keys` value with the `new ETH private key` that you saved from running the script in step 3. that is your new private key.

Replace `coinbase address` with the new public address u saved

  **👉 Save with `CTRL+X`, press `Y`, then Enter.**

  ### ii. stop container and update image tag
  ```bash
  cd aztec && \
docker compose down -v && \
sed -i 's|image: aztecprotocol/aztec:.*|image: aztecprotocol/aztec:2.1.2|' docker-compose.yml && \
docker compose pull && \
docker compose up -d
```

### And if you run your node with CLI then rerun with 
```bash
aztec start --node --archiver --sequencer \
  --network testnet \
  --l1-rpc-urls Eth_Sepolia_RPC \
  --l1-consensus-host-urls Eth-beacon_sepolia_RPC \
  --sequencer.validatorPrivateKeys 0xYourETHkey \
  --sequencer.coinbase YourNewPublicAddress \
  --sequencer.governanceProposerPayload 0xDCd9DdeAbEF70108cE02576df1eB333c4244C666 \
  --p2p.p2pIp Your_ip
```

## 6. check the attester pub address(i.e the new public address on *https://dashtec.xyz/queue* to confirm your validator registration

### ALL DONE
