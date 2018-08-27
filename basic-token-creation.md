## Create token account

This will create an account that we will upload the standard eos token creation contract to. First
we generate a new account with the same public key.


```
cleos create account eosio eosio.token PUBLIC-KEY 
```

Then we push the contract to the blockchain under that account. `-x` sets a timeout window. Without
this you may have to execute this command multiple times.


```
cleos set contract eosio.token contracts/eosio.token/ -x 1000 -p eosio.token
```

Now lets create a pool of tokens called MYTH tokens and give the eosio account (private key should
be in your wallet) control of issuing these tokens.

```
cleos push action eosio.token create '[ "eosio", "1000000000.0000 MYTH"]' \
    -p eosio.token@active
```
