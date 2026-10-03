# Defiantly-Arbitraging: Simulation-Only DEX Arbitrage Scanner

A Python tool that watches the **WETH/USDC** pools on **Uniswap V2** and **SushiSwap**, detects price gaps between them, and calculates how much profit an arbitrage trade *would* have made after gas.

**It is simulation-only.** It never sends a transaction, never signs anything, and needs no private key or wallet. It only reads on-chain pool reserves.

## How it works

Each poll, the scanner:

1. **Reads reserves** from both liquidity pools through an Ethereum RPC endpoint.
2. **Compares prices.** WETH is bought on the cheaper pool and sold on the pricier one.
3. **Finds the best trade size.** Profit from a constant-product (x·y = k) pool is concave in trade size, so a ternary search finds the input amount that maximizes gross profit.
4. **Subtracts estimated gas.** Gas cost is estimated from the current gas price and a configurable gas-units figure.
5. **Logs the result** to `results.csv` (timestamp, block, both prices, direction, best size, gross, gas, net).

The swap math follows the Uniswap V2 formula, including the 0.3% fee on each leg.

## Setup

```bash
git clone https://github.com/booneyjam/Defiantly-Arbitraging.git
cd Defiantly-Arbitraging
pip install -r requirements.txt
export RPC_URL="https://your-ethereum-rpc"
```

## Usage

```bash
python scanner.py --polls 1000 --interval 12
```

| Flag | Default | Meaning |
|---|---|---|
| `--polls` | 100 | How many times to check the pools |
| `--interval` | 12.0 | Seconds between checks (about one Ethereum block) |
| `--out` | results.csv | Output file |

## Limitations

- Gas is a rough estimate (`GAS_UNITS = 250_000`), not a simulated transaction.
- Ignores slippage from other traders, MEV competition, and failed or front-run trades. A positive net number here is not a guarantee of real profit.
- Covers one token pair on two venues. Extending to more pairs or DEXs is the natural next step.
- Verify the pool addresses on Etherscan before running.

## Disclaimer

Educational project. Not financial advice.

## License

MIT
