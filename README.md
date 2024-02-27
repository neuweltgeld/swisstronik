![image](https://github.com/neuweltgeld/swisstronik/assets/101174090/f2be07d6-d9cc-4969-ae5c-bb2c2f235aa5)

With this guide, you can easily and quickly install validator.
It is a testnet guide, tokens have no financial value.
If you encounter an error during installation, you can ask by tagging ``irlandali_turist`` in the ``#dev-chat channel`` in the Swisstronik discord group.
IMPORTANT : You will need an SGX enabled server for your validator to work.

My server: 
Intel Xeon-E 2386G - 6c/12t - 3.5GHz/4.7GHz
32GB DDR4 ECC 3200MHz
2× 512GB SSD NVMe Soft RAID
Ubuntu 22.04

#### Update and install dependencies

```
sudo apt update
sudo apt upgrade

sudo apt-get install build-essential ocaml ocamlbuild automake autoconf libtool wget python-is-python3 libssl-dev git cmake perl
sudo apt-get install libssl-dev libcurl4-openssl-dev protobuf-compiler libprotobuf-dev debhelper cmake reprepro unzip pkgconf libboost-dev libboost-system-dev libboost-thread-dev lsb-release libsystemd0
sudo apt-get install snapd lz4 -y
```

#### SGX driver install
```
wget https://download.01.org/intel-sgx/sgx-linux/2.22/distro/ubuntu22.04-server/sgx_linux_x64_driver_2.11.54c9c4c.bin 
chmod +x sgx_linux_x64_driver_2.11.54c9c4c.bin
sudo ./sgx_linux_x64_driver_2.11.54c9c4c.bin
```
#### AESM install
```
echo "deb https://download.01.org/intel-sgx/sgx_repo/ubuntu $(lsb_release -cs) main" | sudo tee -a /etc/apt/sources.list.d/intel-sgx.list >/dev/null
curl -sSL "https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key" | sudo -E apt-key add -
sudo apt update
sudo apt install sgx-aesm-service libsgx-aesm-launch-plugin libsgx-aesm-epid-plugin
```
#### SGX install
```
echo "deb https://download.01.org/intel-sgx/sgx_repo/ubuntu $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/intel-sgx.list >/dev/null
curl -sSL "https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key" | sudo -E apt-key add -
sudo apt update
sudo apt install libsgx-launch libsgx-urts libsgx-epid libsgx-quote-ex sgx-aesm-service libsgx-aesm-launch-plugin libsgx-aesm-epid-plugin libsgx-quote-ex libsgx-dcap-ql libsnappy1v5 jq
sudo apt install gcc protobuf-compiler pkg-config libssl-dev
```
#### Install Rust
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
cargo install sgxs-tools
```
#### Checking SGX activation
```
sudo $(which sgx-detect)
```
#### Expected output

![image](https://github.com/neuweltgeld/swisstronik/assets/101174090/30082e13-ae93-4b41-a842-d4c38cee2344)


#### Go installation
```
sudo rm -rvf /usr/local/go/
wget https://go.dev/dl/go1.21.4.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.21.4.linux-amd64.tar.gz
rm go1.21.4.linux-amd64.tar.gz
```
#### Go PATHs
```
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
```

#### Install from binary
```
wget https://github.com/SigmaGmbH/swisstronik-chain/releases/download/v1.0.1/swisstronikd.deb.zip 
unzip swisstronikd.deb.zip  
dpkg -i swisstronik_1.0.1-updated-binaries_amd64.deb 
```
#### Check binary version. Expected output ``v1.0.1``
```
swisstronikd version
```
#### Create ``.swisstronik-enclave`` folder and copy ``enclave.signed.so`` to destination.
```
cd $HOME
mkdir .swisstronik-enclave
cp /usr/lib/enclave.signed.so /root/.swisstronik-enclave/enclave.signed.so
```

#### Create master key before
```
swisstronikd enclave request-master-key rpc.testnet.swisstronik.com:46789
```

#### Gas price and other configs.
```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"7uswtr\"|" $HOME/.swisstronik/config/app.toml
```
```
swisstronikd config chain-id swisstronik_1291-1
swisstronikd config keyring-backend test
swisstronikd config node tcp://localhost:26657
```

#### Node initialisation step. Change the ``MONIKER_NAME`` variable yourself.
```
swisstronikd init MONIKER_NAME --chain-id swisstronik_1291-1
```
#### Get genesis.json and addrbook.json
```
curl -Ls https://snapshot.indonode.net/swisstronik-t/genesis.json > $HOME/.swisstronik/config/genesis.json
curl -Ls https://snapshot.indonode.net/swisstronik-t/addrbook.json > $HOME/.swisstronik/config/addrbook.json
```
#### Add peers 
```
PEERS="58dbd1534053615b55919ea9c60f741061ddbd67@162.19.138.14:10656,1f35bf4128576d94c99297a2e33b06b7ee0ae3d2@146.59.111.161:26656,97aafa915684d2aa65e671b9d42461c9bc4bf5a0@198.244.215.141:26656,56d507abe27605f6cf58d13070a2d15c18adfa27@198.244.215.140:26656,38b85901aa0eccb8c3219ca75ec02761cea26746@198.244.215.142:26656,7c42a55317deda257fee06bc48574fa3444967db@37.59.18.38:27656,a0ef787fefc6b58bbc240bcfff429b59b2398946@141.94.240.117:26456,eee348ce7f8046fed477faee2e658b0f7b08dadd@148.113.9.30:10656,148cee337f690b973160f5558dcddd88280dc70f@148.113.1.30:26656,ae374e4f38be1e271669c5928d43c5914350a92a@57.129.1.46:26656,8918f624aa8b087f68bad6d8270c3953a700a890@141.95.169.103:16656,df5091c793b0345da4948104091adb40c80dea9e@148.113.16.39:26456,54b1092174218aad93a22f7ac2eaa74bb2326bbc@89.58.13.159:6969"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.swisstronik/config/config.toml
sed -i -e "s|^seeds *=.*|seeds = \"\"|" $HOME/.swisstronik/config/config.toml
```

#### Creating service file
```
sudo tee /etc/systemd/system/swisstronikd.service > /dev/null <<EOF
[Unit]
Description=swisstronik testnet node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which swisstronikd) start
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
#### Enable daemon 
```
sudo systemctl daemon-reload
sudo systemctl enable swisstronikd
```

#### Use snapshot to catch blocks
```
cp $HOME/.swisstronik/data/priv_validator_state.json $HOME/.swisstronik/priv_validator_state.json.backup
rm -rf $HOME/.swisstronik/data

curl -L https://snapshot.indonode.net/swisstronik-t/swisstronik-t-latest.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.swisstronik
mv $HOME/.swisstronik/priv_validator_state.json.backup $HOME/.swisstronik/data/priv_validator_state.json
```
#### AND, start the node and check everything allright.
```
sudo systemctl start swisstronikd && sudo journalctl -fu swisstronikd -o cat
```

#### Creating wallet step
```
swisstronikd keys add wallet --keyring-backend test
```

#### Use Faucet to get testnet tokens.
https://faucet.testnet.swisstronik.com/

#### Create the validator.
```
swisstronikd tx staking create-validator \
--amount=900000000000000000uswtr \
--moniker="MONIKER" \
--pubkey=$(swisstronikd tendermint show-validator) \
--chain-id=swisstronik_1291-1 \
--commission-rate=0.1 \
--commission-max-rate=0.1 \
--commission-max-change-rate=0.05 \
--min-self-delegation=1 \
--from=wallet \
--gas="1000000" \
--gas-prices="30000000000uswtr"
```
