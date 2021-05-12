# truffle-eip155-issues
Reproduction for Truffle EIP-155 issues

## Fixed!
In `hdwallet-provider` v1.4.0, which the `main` branch has been updated to.

## Pre-requisites
- Truffle v5.3.0
- geth v1.10.1

## Steps to reproduce
1. Startup a local geth dev node with HTTP RPC enabled, **and the network ID different from the chain ID**.
```
$ geth --dev --http --networkid 1338 console
INFO [03-31|00:33:18.012] Starting Geth in ephemeral dev mode...
INFO [03-31|00:33:18.013] Maximum peer count                       ETH=50 LES=0 total=50
INFO [03-31|00:33:18.014] Set global gas cap                       cap=25000000
INFO [03-31|00:33:19.941] Using developer account                  address=0x1CD1278Eb886198180657397B6d39802a9e07e41
INFO [03-31|00:33:19.941] Allocated trie memory caches             clean=154.00MiB dirty=256.00MiB
INFO [03-31|00:33:19.941] Writing custom genesis block
INFO [03-31|00:33:19.942] Persisted trie from memory database      nodes=12 size=1.82KiB time="73.867µs" gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
INFO [03-31|00:33:19.942] Initialised chain configuration          config="{ChainID: 1337 Homestead: 0 DAO: <nil> DAOSupport: false EIP150: 0 EIP155: 0 EIP158: 0 Byzantium: 0 Constantinople: 0 Petersburg: 0 Istanbul: 0, Muir Glacier: 0, Berlin: 0, YOLO v3: <nil>, Engine: clique}"
INFO [03-31|00:33:19.942] Initialising Ethereum protocol           network=1338 dbversion=<nil>
<snip>
INFO [03-31|00:33:20.085] Sealing paused, waiting for transactions
Welcome to the Geth JavaScript console!

instance: Geth/v1.10.1-stable-c2d2f4ed/darwin-amd64/go1.16
coinbase: 0x1cd1278eb886198180657397b6d39802a9e07e41
at block: 0 (Thu Jan 01 1970 09:00:00 GMT+0900 (JST))
 datadir:
 modules: admin:1.0 clique:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0
```
2. Send some ETH from the developer account to a test account of your choice.
```
> eth.sendTransaction({from:eth.accounts[0],to:"<YOUR ADDRESS HERE>",value:web3.toWei(100,"ether")})
INFO [03-31|00:43:46.126] Setting new local account                address=0x1CD1278Eb886198180657397B6d39802a9e07e41
INFO [03-31|00:43:46.127] Submitted transaction                    hash=0xd8cd62650e5c70be3ed454703b179a4b1cc7f335a60affe1bf07e8bfe0f31dd5 from=0x1CD1278Eb886198180657397B6d39802a9e07e41 nonce=0 recipient=0xBaC1Cd4051c378bF900087CCc445d7e7d02ad745 value=100000000000000000000
"0xd8cd62650e5c70be3ed454703b179a4b1cc7f335a60affe1bf07e8bfe0f31dd5"
> INFO [03-31|00:43:46.127] Commit new mining work                   number=1 sealhash="bfe5aa…6a236d" uncles=0 txs=1 gas=21000 fees=2.1e-14 elapsed="374.987µs"
INFO [03-31|00:43:46.127] Successfully sealed new block            number=1 sealhash="bfe5aa…6a236d" hash="fe45c4…0013d7" elapsed="466.986µs"
INFO [03-31|00:43:46.128] 🔨 mined potential block                  number=1 hash="fe45c4…0013d7"
INFO [03-31|00:43:46.128] Commit new mining work                   number=2 sealhash="413e6e…01abf9" uncles=0 txs=0 gas=0     fees=0       elapsed="320.316µs"
INFO [03-31|00:43:46.128] Sealing paused, waiting for transactions

> eth.getBalance("<YOUR ADDRESS HERE>")
100000000000000000000
>
```

In a separate terminal window:
1. Clone the repo
2. Put your private key or mnemonic into a `.secret` file in the root folder of the repo.
3. `yarn install` to pull in the dependencies.
4. `truffle migrate` and observe the EIP-155 failure:
```
$ truffle migrate

Compiling your contracts...
===========================
> Compiling ./contracts/Migrations.sol
> Artifacts written to /Users/jeff/projects/curvegrid/truffle-eip155-issues/build/contracts
> Compiled successfully using:
   - solc: 0.5.16+commit.9c3226ce.Emscripten.clang

Starting migrations...
======================
> Network name:    'development'
> Network id:      1337
> Block gas limit: 0xaf79e0


1_initial_migration.js
======================

   Deploying 'Migrations'
   ----------------------

Error:  *** Deployment Failed ***

"Migrations" -- invalid sender.

    at /usr/local/lib/node_modules/truffle/build/webpack:/packages/deployer/src/deployment.js:365:1
    at processTicksAndRejections (node:internal/process/task_queues:94:5)
    at Migration._deploy (/usr/local/lib/node_modules/truffle/build/webpack:/packages/migrate/Migration.js:74:1)
    at Migration._load (/usr/local/lib/node_modules/truffle/build/webpack:/packages/migrate/Migration.js:61:1)
    at Migration.run (/usr/local/lib/node_modules/truffle/build/webpack:/packages/migrate/Migration.js:212:1)
    at Object.runMigrations (/usr/local/lib/node_modules/truffle/build/webpack:/packages/migrate/index.js:150:1)
    at Object.runFrom (/usr/local/lib/node_modules/truffle/build/webpack:/packages/migrate/index.js:110:1)
    at Object.run (/usr/local/lib/node_modules/truffle/build/webpack:/packages/migrate/index.js:87:1)
    at runMigrations (/usr/local/lib/node_modules/truffle/build/webpack:/packages/core/lib/commands/migrate.js:263:1)
    at Object.run (/usr/local/lib/node_modules/truffle/build/webpack:/packages/core/lib/commands/migrate.js:228:1)
    at Command.run (/usr/local/lib/node_modules/truffle/build/webpack:/packages/core/lib/command.js:140:1)
Truffle v5.3.0 (core: 5.3.0)
Node v15.11.0
$
```

In the geth console:
```
WARN [04-03|03:04:02.906] Served eth_sendRawTransaction            conn=127.0.0.1:55049 reqid=1980185469547951 t="98.893µs"  err="invalid sender"
```
