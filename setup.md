## Table of Contents

[Installation and Setup Docker](#installation-and-setup-docker)<br>
[Installation and Setup Build](#installation-and-setup-build)<br>
[Create Wallet](#create-wallet)<br>
[Deploy Bios Contract](#deploy-bios-contract)<br>
[Create a User Account](#create-a-user-account)<br> 


### Installation and Setup Docker

If you have simply needs using docker is the easiest method. To do this we use docker and docker-compose to run Nodeos (and Keosd for local wallet). To set this up the following are used:

```
git clone https://github.com/EOSIO/eos.git --recursive  --depth 1
cd eos/Docker
docker volume create --name=nodeos-data-volume
docker volume create --name=keosd-data-volume
docker-compose up -d
```

Now, set an alias for `cleos` so that you can run commands anywhere

```
alias cleos='docker-compose -f /path/to/home/eos/Docker/docker-compose.yml exec keosd /opt/eosio/bin/cleos -u http://nodeosd:8888 --wallet-url http://localhost:8900'
```

Test that it works

`cleos get info`

### Installation and Setup Build

To better customize and interact with a running EOS node, building and running outside of docker
simplifies things. This assumes running Ubuntu 18.04.

#### Dependencies

```
sudo apt-get update
wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key|sudo apt-key add -
sudo apt-get install clang-4.0 lldb-4.0 libclang-4.0-dev cmake make \
                     libbz2-dev libssl-dev libgmp3-dev \
                     autotools-dev build-essential \
                     libbz2-dev libicu-dev python-dev \
                     autoconf libtool git mongodb
```

Install Boost 1.66

```
cd ~
wget -c 'https://sourceforge.net/projects/boost/files/boost/1.66.0/boost_1_66_0.tar.bz2/download' -O boost_1.66.0.tar.bz2
tar xjf boost_1.66.0.tar.bz2
cd boost_1_66_0/
echo "export BOOST_ROOT=$HOME/boost_1_66_0" >> ~/.bash_profile
source ~/.bash_profile
./bootstrap.sh "--prefix=$BOOST_ROOT"
./b2 install
source ~/.bash_profile
```

Install Mongo-cxx-driver

```
cd ~
curl -LO https://github.com/mongodb/mongo-c-driver/releases/download/1.9.3/mongo-c-driver-1.9.3.tar.gz
tar xf mongo-c-driver-1.9.3.tar.gz
cd mongo-c-driver-1.9.3
./configure --enable-static --enable-ssl=openssl --disable-automatic-init-and-cleanup --prefix=/usr/local
make -j$( nproc )
sudo make install
git clone https://github.com/mongodb/mongo-cxx-driver.git --branch releases/stable --depth 1
cd mongo-cxx-driver/build
cmake -DBUILD_SHARED_LIBS=OFF -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local ..
sudo make -j$( nproc )
```

Install secp256k1-zkp (Cryptonomex branch)

```
cd ~
git clone https://github.com/cryptonomex/secp256k1-zkp.git
cd secp256k1-zkp
./autogen.sh
./configure
make
sudo make install
```

By default LLVM and clang do not include the WASM build target, so you will have to build it yourself:

```
mkdir  ~/wasm-compiler
cd ~/wasm-compiler
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/llvm.git
cd llvm/tools
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/clang.git
cd ..
mkdir build
cd build
cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=.. -DLLVM_TARGETS_TO_BUILD= -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD=WebAssembly -DCMAKE_BUILD_TYPE=Release ../
make -j4 install
```

Now we can install eos and optionally
[eosio-watcher-plugin](https://github.com/eosauthority/eosio-watcher-plugin), which can filter and
POST notifications of activity


```
git clone https://github.com/EOSIO/eos --recursive
git clone https://github.com/eosauthority/eosio-watcher-plugin
mkdir -p ~/eos/build && cd ~/eos/build
cmake -DBINARYEN_BIN=~/binaryen/bin -DWASM_ROOT=~/wasm-compiler/llvm -DOPENSSL_ROOT_DIR=/usr/local/opt/openssl -DOPENSSL_LIBRARIES=/usr/local/opt/openssl/lib -DBUILD_MONGO_DB_PLUGIN=true -DEOSIO_ADDITIONAL_PLUGINS=~/eosio-watcher-plugin ..
make -j$( nproc )
```

### Create Wallet

Now we need to set up a loca wallet. Remember this has nothing to do with the network, it is just
used to store key pairs and sign transactions.

The following command will create a wallet using keosd and save the password (which always starts
with PW\*)

```
cleos wallet create --file=default.wallet
cleos wallet open
cleos wallet list
```

Now since we are running our own chain, we want to make sure we have access to the eosio master
account. So lets import the private key for that account defined in config.ini. This can be found
in the eos repo or you can do:
```
docker exec -it eos_container_name bash
ls ~/.local/share/eosio/nodeos/
```

```
cleos wallet import --private-key PRIVATE_KEY_OF_MASTER
```

### Deploy Bios Contract

```
cleos set contract eosio eos/contracts/eosio.bios/ -p eosio@active
```

### Create a User Account

Now we are going to create a user account. First step is to generate a key pair. Save the private
key. Then we have the eosio account create another account. Behind the scenes it is dedicating
enough RAM, luckily this account comes with a lot of EOS already to let us do whatever we want.


```
cleos create key --to-console
cleos create account eosio user PUBLIC-KEY-CREATED
```                                 
