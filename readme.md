# YES Token Snapshot

This repo includes an independently produced snapshot of YES token balances across various protocols and contracts at a block height of [Operation YEV](https://mirror.xyz/0xe7AD459A24A10C5E94B76CcD24da62A8394eBf5f/egAUJ7mxagSBF_s9tsFqV4R_ATLMh_GfjasRe4YITKM) on the Blast chain. It unwraps token balances from liquidity pools, lending markets, and other contracts to attribute the underlying YES tokens to their ultimate owners.

## Overview

The script performs the following main steps:

1. Fetches YES token transfer events from the token's deployment block up to the specified snapshot block.
2. Calculates the spot YES token balances by processing the transfer events. 
3. Identifies contracts holding YES tokens and unwraps them using protocol-specific logic to determine the ultimate owners of the tokens.
4. Applies some manual mappings and exclusions for special cases.
5. Captures YES token balances of team wallets used in the Operation YEV.
6. Outputs the consolidated snapshot data, attributing YES balances to owner addresses. This data can be used by the team for the final allocation.

## Requirements

- Install Python 3.10+

`pyenv` is a good option, it allows you to install multiple versions
```sh
curl https://pyenv.run | bash
pyenv install 3.11
pyenv global 3.11
```

- Install `uv` ([docs](https://github.com/astral-sh/uv))
```sh
curl -LsSf https://astral.sh/uv/install.sh | sh
```

- Install dependencies
```sh
uv venv
source .venv/bin/activate  # may differ for your shell
uv pip sync requirements.txt
```

- Blast archive node provider URL
    - A paid Quicknode works, you may need to adjust the batch size and rate limits down on a public node.

## Development

Run `nb-clean clean -o baseline-snapshot.ipynb` before making a commit. This would retain the outputs while removing metadata like execution counts and last execution time from the notebook and will make the diff smaller.

You can also install a local git filter with `nb-clean add-filter --preserve-cell-outputs` that would do this automatically when staging the notebook.

## Configuration

To specify the RPC provider, create an `ape-config.yaml` file with the following content:

```yaml
geth:
  blast:
    mainnet:
      # replace with your archive node
      uri: https://rpc.blast.io
```

## Usage

It is preferred to run the notebook in an interactive environment with VS Code or Jupyer Lab.

```
jupyter lab baseline-snapshot.ipynb
```

Alternatively you can execute it from the console. No output is displayed in this case.

```
jupyter execute baseline-snapshot.ipynb
```

The script will fetch data from the chain, process it, and output a snapshot CSV file called `snapshot_data.csv`.

The CSV has the following columns:
- `source`: The original address that held the YES tokens (EOA or contract)
- `protocol`: The protocol or contract type where the balance was attributed from
- `user`: The final owner address that the balance is attributed to
- `token`: The token address. Non-YES tokens are captured for LP positions, make sure you filter by token
- `balance`: The token balance at the snapshot block

## Supported Protocols & Contracts

The script has unwrapping logic to attribute tokens for the following protocols and contract types:

- Thruster V2 & V3 liquidity pools
- Baseline credit accounts
- Ambient
- Alien
- Ajna 
- Few
- Maybe
- Roguex 
- Wasabi
- Safe multisig wallets
- Unknown contracts with an owner function
- Manual attributions specified in the script

If the script encounters a contract it can't classify, it prints a warning. Unattributed contract balances are assigned to the contract address itself in the final snapshot.

Note: There are no unattributed contracts remaining.

## Calculations

The key calculations involve:

- Deriving spot balances by processing net transfers
- Calculating shares of pool balances based on LP tokens
- Calculating balances for lending positions
- Unwrapping balances attributed to contracts for a second time

Decimals and units are handled carefully. Raw token amounts are preferred till the final balance calculation. The final balances are output in human-readable format.

## Team Wallet Balances

A separate `unwrap_team` function snapshots the balances and credit accounts of a hardcoded list of team wallet addresses. These balances and accounts were sacrificed in the Operation YEV, so they are snapshotted one block prior than everything else.
