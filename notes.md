mkdir testdirectory
cd testdirectory
forge init (or forge init --force)
/src/test.sol (make sol file in src folder)

## Compiling in Foundry

terminal
forge build -or- forge compile

## Deploying to a local blockchain using Anvil

MetaMask -> Settings -> Networks -> Add Network

* Network Name: LocalHost
* New RPC-URL: http://127.0.0.1:8545
* Chain ID: 31337 (Anvil's Chain ID)
* Currency symbol: ETH
* Block explorer URL (Optional): 

Save -> Switch to LocalHost

Copy one of the Private Keys from anvil
Import it into MetaMask

* Import account
* Enter your private key string here: 
  (Paste Private Key from anvil)
* Hit import

## Look into running own Ethereum Node ##

## Deploying to a local blockchain using Forge

CLI:

* run anvil
* hit '+' for new terminal

* Compile Contract: 
    * forge create <path>:<contractname>
    (ex: forge create src/SimpleStorage.sol:SimpleStorage)
            -OR-
    * forge create <contractname>
    (ex: forge create SimpleStorage)

* On bash:
    * forge create <contractname> --interactive
    (ex: forge create SimpleStorage --interactive)
    (ex: Enter private key:)
    (ex: <paste private key>)
    (* never use real private key *)
    (Contract deployed to anvil blockchain)

* Long way:
    * forge create <contractname> --rpc-url <rpc-url> --private-key <paste private key>
    (* never use real private key *)

## Deploy locally using Anvil (the sequel but a good sequel. Like Aliens)

When deploying your contract, you want to make sure you have a continuous reproducible way to deploy your contracts. And when testing your contracts, you want the test to also test the deployment processes as well as the code.

Write a script to deploy your contract

/script
DeploySimpleStorage.s.sol (scripts have .s.sol naming convention)
write a contract (to deploy your smart contract)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

contract DeploySimpleStorage {}
```

[Everything to know about Scripting](https://book.getfoundry.sh/tutorials/solidity-scripting)

Solidity scripting is a way to declaratively deploy contracts using Solidity, instead of using the more limiting and less user friendly 'forge create'.

Solidity scripts are written in Solidity, and they are run on the fast Foundry EVM backend, which provides dry-run capabilities.

[The best practices for Scripts](https://book.getfoundry.sh/tutorials/best-practices#scripts)

* First: import additional code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

import {Script} from "forge-std/Script.sol";
import {SimpleStorage} from "../src/SimpleStorage.sol";

contract DeploySimpleStorage is Script {}
```

* Second: create function named run
          (run will be your main function for your DeployScript)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

import {Script} from "forge-std/Script.sol";
import {SimpleStorage} from "../src/SimpleStorage.sol";

contract DeploySimpleStorage is Script{
    function run() external returns(SimpleStorage){
        vm.startBroadcast();
        SimpleStorage simpleStorage = new SimpleStorage();
        //simpleStorage - variable
        //SimpleStorage - contract
        //'new' keyword creates a new contract
        //so this line sends a transaction to the rpc to create a new simple storage contract
        vm.stopBroadcast();
        return simpleStorage;
    }
}
```

* vm is a special keyword in the forge standard library
* it is only used in Foundry; only works in Foundry
* it is related to cheatcodes

[Foundry Cheatcodes Reference](https://book.getfoundry.sh/cheatcodes/)
[Forge Standard Library References](https://book.getfoundry.sh/reference/forge-std/)

* vm is not valid in regular solidity and that is why we import Script from forge-std and inherit forge STD code to our contract with 'is Script'

* vm.startBroadcast() tells the contract 'everything after this, you should send to the rpc'
* vm.stopBroadcast() tells the contract 'stop broadcasting'

Terminal:

* forge script script/DeploySimpleStorage.s.sol

(Compiling...)
(Compiler run successful!)
(Script ran successfully)
Gas used: <gasused>

== Return ==
0: contract SimpleStorage 0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496
(* in foundry, if you don't specify an RPC URL - foundry will automatically deploy your contract, or run your script, on a temporary anvil chain *)

(If you wish to simulate on-chain transactions pass a RPC URL)

SIMULATION DEPLOYMENT

* anvil
* bash
* forge script script/DeploySimpleStorage.s.sol --rpc-url http://127.0.0.1:8545 

ACTUAL DEPLOYMENT

* anvil
* bash
* forge script script/DeploySimpleStorage.s.sol --rpc-url http://127.0.0.1:8545 --broadcast --private-key 0x2a871d0798f97d79848a013d4936a73bf4cc922c825d33c1cf7073dff6d409c6
(private key is a default anvil private key)

## NEVER USE A .ENV FILE

<!--
ALTERNATIVE TO USING A PRIVATE KEY (ONLY FOR TESTING; NEVER PRODUCTION)

* create new file .env
* goto .gitignore file
* under '# Dotenv file' make sure .env is listed

* in .env file - add environment variables
( environment variables are variables that are sensitive that you don't want in the command line or accidentally publically exposed )
* in terminal: run source .env
( adds the environment variables into your shell )
* echo <environment variable> to see if it is loaded
(Ex: echo $PRIVATE_KEY)
(Ex: echo $RPC_URL)

* forge script script/DeploySimpleStorage.s.sol --rpc-url $RPC_URL --broadcast --private-key $PRIVATE_KEY
-->

## NEVER USE A .ENV FILE

 private key for example: 0x2a871d0798f97d79848a013d4936a73bf4cc922c825d33c1cf7073dff6d409c6

* DO NOT USE TERMINAL IN VSCODE - USE CLI

* cast wallet import <keyname> --interactive
(Ex: cast wallet import testKey -- interactive)
* Enter private key:
(paste key: 0x2a871d0798f97d79848a013d4936a73bf4cc922c825d33c1cf7073dff6d409c6)
* Enter password:
(REMEMBER PASSWORD - VITAL)
testKey keystore was saved successfully. Sender Address: 0xa0ee7a142d267c1f36714e4a8f75612f20a79720

* cast wallet list
(gives list of key(s))
* cd .foundry/keystores/
* ls

* forge script script/DeploySimpleStorage.s.sol:DeploySimpleStorage --rpc-url http://127.0.0.1:8545 --account <keyname> --sender <senderaddress> --broadcast -vvvv
(Ex: forge script script/DeploySimpleStorage.s.sol:DeploySimpleStorage --rpc-url http://127.0.0.1:8545 --account testKey --sender 0xa0ee7a142d267c1f36714e4a8f75612f20a79720 --broadcast -vvvv)
* Enter keystore password:
(enter password - without password, won't run)

## INTERACTING WITH CONTRACT USING CLI

### Setting (Sending) Data

* copy contract address
(Ex: 0x700b6A60ce7EaaEA56F065753d8dcB9653dbAD35)
* cast send <contract address> "<function name + (input parameters)>" <data> --rpc-url <rpc-url> --account <keyname>
(Ex: cast send 0x700b6A60ce7EaaEA56F065753d8dcB9653dbAD35 "store(uint256)" 1110 --rpc-url http://127.0.0.1:8545 --account testKey)
* Enter keystore password:
(enter password)

### Reading (Retrieving) Data

* cast call <contract address> "<function name + (input parameters)>"
(Ex: cast call 0x700b6A60ce7EaaEA56F065753d8dcB9653dbAD35 "retrieve()")
* hex value returned
(Ex: 0x0000000000000000000000000000000000000000000000000000000000000456)
* cast --to-base <returns hex value> dec
(Ex: cast --to-base 0x0000000000000000000000000000000000000000000000000000000000000456 dec)
* <data>
(Ex: 1110)

## Deploying on a testnet (Sepolia)

(NaaS = Node as a Service)

* Alchemy
* Create new App
* Network: Ethereum Sepolia

* need sepolia key 
* need sender for sepolia key
* need sepolia RPC

sepoliaKey keystore was saved successfully. 
Sender Address: 0x6b686347a3f7a3f7fda330af7f1947006c3d452f
(commandList.md)

## hex to dec

cast --to-base <hex value> dec
(ex: cast --to-base 0x714c2 dec) (0x714c2 is the hex of the value of gas)
(ex: 464066)  

## zkSync

* Install: foundry-zksync
* Compile: forge build --zksync
* zkSync foundry (foundryup-zksync)
* Vanilla foundry (foundryup)

zkSync foundry: (At present) forge 0.0.2 (2a54657 2024-12-07T00:26:37.606369492Z)

Vanilla foundry: (At present) forge 0.2.0 (4923529 2024-11-25T00:23:51.014536290Z)

## TEST

* naming convention: <contractNameTest.t.sol>
(Ex: FundMeTest.t.sol)
* import {Test} from "forge-std/Test.sol";
* contract <contract name> is Test {}
(contract inherites from Test)
* console: 
(import {Test, console} from "forge-std/Test.sol";)
* contract:
(import {FundMe} from "../src/FundMe.sol";)
