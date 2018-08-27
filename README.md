eos-docs
========

## Installation and Setup

We are using docker and docker-compose to run Nodeos (and Keosd for local wallet). To set this up
the following are used:

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
