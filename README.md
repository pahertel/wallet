# wallet

## Transaction proof
![wallet_screenprint](https://user-images.githubusercontent.com/71287557/109401787-ae965680-791e-11eb-9860-1c5afeeef53a.png)

## Wallet Code

import subprocess
import json
import os
from constants import *
import bit
from bit import PrivateKeyTestnet
from web3 import Web3
from eth_account import Account

w3 = Web3(Web3.HTTPProvider("http://127.0.0.1:8545"))

mnemonic = os.getenv('MNEMONIC', 'enter position alley alpha cave miracle tunnel illness rain pulp survey salon')

def derive_wallets(mnemonic,coin):
    
    command = f'php hd-wallet-derive.php -g --mnemonic="{mnemonic}" --cols=path,address,privkey,pubkey --coin={coin} --numderive=3 --format=json'

    p = subprocess.Popen(command, stdout=subprocess.PIPE, shell=True)
    output, err = p.communicate()
    p_status = p.wait()

    keys = json.loads(output)
    
    return keys

coins = {}

coins['eth'] = derive_wallets(mnemonic,ETH)
coins['btc-test'] = derive_wallets(mnemonic,BTCTEST)

print(coins)

def priv_key_to_account(coin, priv_key):
    
    if coin == 'eth':
        return Account.privateKeyToAccount(priv_key)
    
    elif coin == "btc-test":
        return bit.PrivateKeyTestnet(priv_key)


def create_tx(coin, account, to, amount):
    
    if coin == 'eth':
        
        gasEstimate = w3.eth.estimateGas(
        {"from": account.address, "to": to, "value": amount})
        
        transaction = {
            "nonce": w3.eth.getTransactionCount(account.address),
            "gasPrice": w3.eth.gasPrice,
            "gas": gasEstimate,
            "to": to, 'from': account.address, 'value': amount
        }
        
        return transaction
            
    elif coin == 'btc-test':
        set_transation = bit.PrivateKeyTestnet.prepare_transaction(account.address, [(to, amount, BTC)])
            
        return set_transation
            

def send_tx(coin, account, to, amount, private_key):
            
    if coin == 'eth':
        raw_tx = create_tx(coin, account, to, amount)
        signed_tx = w3.eth.account.sign_transaction(raw_tx, private_key)
        return w3.eth.sendRawTransaction(signed_tx.rawTransaction)
        
    elif coin == 'btc-test':
        raw_tx = create_tx(coin, account, to, amount)
        signed_tx = account.sign_transaction(raw_tx)
        return bit.network.NetworkAPI.broadcast_tx_testnet(signed_tx)




## Install
Clone: hd-wallet-derive tool.
dan-da/hd-wallet-derive: A command-line tool that derives bip32 addresses and private keys. (github.com)

web3.py git bash install: pip install web3

bit git bash install: pip install bit

## Description
###What does it do:
To store and send cryptocurrencies from one place.

###How to use it:
Open git bash
cd to hd-wallet
run: python
run: from wallet import *
to send ether: send_tx(ETH, priv_key_to_account(ETH,'0x...'), '0x....', "ETH amount", '0x....')
to send bitcoin test: send_tx(BTCTEST, priv_key_to_account(BTCTEST,'....'), '......', "BTCTEST amount", None)
