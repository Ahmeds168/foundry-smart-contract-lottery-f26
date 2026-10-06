# Foundry Smart Contract Lottery

A provably fair lottery written in Solidity and built with Foundry. Players pay an entrance fee to join a round, and after a set interval the contract asks Chainlink VRF for a random number and pays the whole prize pool to one winner. Nobody, including the deployer, can predict or influence who wins.

Deployed on Sepolia: [`0xdAEdf1079F23C1D4DBf25c94BC924764c6d8b84A`](https://sepolia.etherscan.io/address/0xdAEdf1079F23C1D4DBf25c94BC924764c6d8b84A)

## How it works

1. A player calls `enterRaffle()` and sends at least the entrance fee.
2. Once the interval has passed and the raffle has players and a balance, `checkUpkeep()` returns `true`.
3. Anyone calls `performUpkeep()`. The raffle closes to new entries and requests a random number from Chainlink VRF v2.5.
4. The VRF coordinator calls back with the random number. The contract picks a winner, sends them the full balance, and reopens for the next round.

The contract follows the Chainlink Automation interface (`checkUpkeep` / `performUpkeep`). Chainlink has since retired the Automation service, so on Sepolia `performUpkeep` is triggered by hand with `cast`, as shown below. It can be driven by a Chainlink CRE workflow or any other scheduler without changing the contract.

## Requirements

- [Git](https://git-scm.com/)
- [Foundry](https://getfoundry.sh/) (`forge`, `cast`, `anvil`)
- `make`

## Setup

```bash
git clone https://github.com/Ahmeds168/foundry-smart-contract-lottery-f26.git
cd foundry-smart-contract-lottery-f26
make install
make build
```

## Testing

```bash
make test
```

Run a single test, with traces:

```bash
forge test --mt testRaffleRevertsWhenYouDontPayEnough -vvvv
```

Check coverage:

```bash
forge coverage
```

## Deploying to Sepolia

Create a `.env` file in the project root. It is git-ignored and must never be committed.

```bash
SEPOLIA_RPC_URL=your_rpc_url
ETHERSCAN_API_KEY=your_etherscan_api_key
```

Import your deployer key into Foundry's encrypted keystore, so the private key never sits in a file:

```bash
cast wallet import default --interactive
```

Then deploy and verify:

```bash
make deploy-sepolia
```

The raffle needs a funded Chainlink VRF subscription with the deployed contract added as a consumer. Subscriptions are managed at [vrf.chain.link](https://vrf.chain.link), and test LINK is available from [faucets.chain.link](https://faucets.chain.link).

## Interacting with the contract

Set the address and load your environment first:

```bash
source .env
RAFFLE=0xdAEdf1079F23C1D4DBf25c94BC924764c6d8b84A
```

Read the entrance fee (in wei) and enter with that amount:

```bash
cast call $RAFFLE "getEntranceFee()(uint256)" --rpc-url $SEPOLIA_RPC_URL
cast send $RAFFLE "enterRaffle()" --value <FEE_IN_WEI> --rpc-url $SEPOLIA_RPC_URL --account default
```

Check whether a round is ready to be drawn, then draw it:

```bash
cast call $RAFFLE "checkUpkeep(bytes)(bool,bytes)" 0x --rpc-url $SEPOLIA_RPC_URL
cast send $RAFFLE "performUpkeep(bytes)" 0x --rpc-url $SEPOLIA_RPC_URL --account default
```

See the result. State `0` means open and `1` means a winner is being calculated.

```bash
cast call $RAFFLE "getRaffleState()(uint8)" --rpc-url $SEPOLIA_RPC_URL
cast call $RAFFLE "getRecentWinner()(address)" --rpc-url $SEPOLIA_RPC_URL
```

## Built with

- [Foundry](https://getfoundry.sh/) for building, testing, and deployment
- [Chainlink VRF v2.5](https://docs.chain.link/vrf) for verifiable randomness
- [Solmate](https://github.com/transmissions11/solmate) and [foundry-devops](https://github.com/Cyfrin/foundry-devops) as supporting libraries

## Acknowledgements

Built while following the [Cyfrin Updraft](https://updraft.cyfrin.io/) Foundry course.
