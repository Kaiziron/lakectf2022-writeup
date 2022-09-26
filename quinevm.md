# QuinEVM [16 solves] [388 points] (Unintended solution)

### Description
```
Do you even quine bro?

nc chall.polygl0ts.ch 4800
```

Do you even quine bro?
Nah, I dont. But I solved it :) (Unintended solution)

The objective of this challenge is to create a contract that return itself (quine) and the contract size has to be less than 8 bytes. We have to build the contract with EVM bytecode directly.




### quinevm.py :
```python
#!/usr/bin/env -S python3 -u

LOCAL = False # Set this to true if you want to test with a local hardhat node, for instance

#############################

import os.path, hashlib, hmac
BASE_PATH = os.path.abspath(os.path.dirname(__file__))

if LOCAL:
    from web3.auto import w3 as web3
else:
    from web3 import Web3, HTTPProvider
    web3 = Web3(HTTPProvider("https://rinkeby.infura.io/v3/9aa3d95b3bc440fa88ea12eaa4456161"))

web3.codec._registry.register_decoder("raw", lambda x: x.read(), label="raw")

def gib_flag():
    with open(os.path.join(BASE_PATH, "flag.txt")) as f:
        print(f.read().strip())

def verify(addr):
    code = web3.eth.get_code(addr)
    if not 0 < len(code) < 8:
        return False

    contract = web3.eth.contract(addr, abi=[ { "inputs": [], "name": "quinevm", "outputs": [ { "internalType": "raw", "name": "", "type": "raw" } ], "stateMutability": "view", "type": "function" } ])
    return contract.caller.quinevm() == code

if __name__ == "__main__":
    addr = input("addr? ").strip()
    if verify(addr):
        gib_flag()
    else:
        print("https://i.imgflip.com/34mvav.jpg")
```

I solved this challenge with unintended solution

I saw this message in discord : 
![](https://i.imgur.com/miGzfuY.png)

They solved it at 11:00 am (HK time), as contracts are deployed in rinkeby testnet, which is public, I explored blocks in a 5 minute timeframe before they solve it

Not much contracts are deployed during in that timeframe, this contract is the only contract that has deployed another contract : 

https://rinkeby.etherscan.io/address/0x03001a67c0841b681266baf4efe2e36896592506#internaltx

![](https://i.imgur.com/M0gkrSh.png)

This is the contract deployed by that factory contract : 

https://rinkeby.etherscan.io/address/0x0ab5eaef94f507c53c95056eb85b9a2ca90ab2f9#code

![](https://i.imgur.com/LpyBQWQ.png)

It has only 7 bytes, so its likely the quine contract deployed by that team

```
# nc chall.polygl0ts.ch 4800
addr? 0x0aB5eaeF94f
507C53c95056eb85B9A2Ca90Ab2F9
EPFL{https://github.com/mame/quine-relay/issues/11}
```

### Flag : 

```EPFL{https://github.com/mame/quine-relay/issues/11}```