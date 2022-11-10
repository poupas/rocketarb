# rocketarb
Arbitrage rETH mint/burn with minipool deposit/withdrawal

## Arb the rETH premium when you create a minipool!

How to? Here are the steps... Message me on the RocketPool Discord if you have
any problems!

1. Dry run creating the minipool the normal way with the Rocketpool smartnode
   (>= 1.7.0). Stop before the final `ARE YOU SURE...` prompt and cancel it.
2. Ensure your node's RPC port is exposed locally as described
   [here](https://docs.rocketpool.net/guides/node/advanced-config.html#execution-client).
3. Install the requirements on your node machine: `nodejs` (>= 18), and `npm`.
4. Clone this repo (`git clone https://github.com/xrchz/rocketarb`), `cd
   rocketarb`, and `npm install` to download the js dependencies.
5. Run `./rocketarb.js`. It should be fine with no arguments. Pass the `--help`
   to see more options if something goes wrong.

## What does it do?
- Ask the smartnode to create a minipool deposit transaction from your node
  wallet address.
- Create a transaction to call the rocketarb contract, which will flash loan 16
  ETH, deposit it in the Rocket Pool deposit pool (using the newly created
  space from the minipool deposit), sell the minted rETH using 1Inch, repay the
  flash loan, and send any profit back to your node wallet.
- Submit the two transactions above in a bundle using Flashbots.

This way you get to benefit from the rETH premium by the space temporarily
created in the deposit pool by your new minipool.

## Tips

- The smartnode needs to be at least version 1.7.0.
- Try `--rpc http://<your node local ip>:8545` if the default
  (`http://localhost:8545`) does not work.
- Use the `--resume` option to try submitting the flashbots bundle again (in
  case it failed) without recreating the transactions. (The bundle gets saved
  in `bundle.json` by default.)
- The gas fee needs to be attractive enough for Flashbots to accept the bundle:
  the target block base fee per gas is burned and the block proposer receives
  any additional fee per gas up to the specified maximum priority fee per gas
  limited by the specified maximum fee per gas. The priority fee is what makes
  a bundle attractive. `rocketarb` uses the same maximum fees for both the
  deposit and arbitrage transactions, and the total gas will be about 2.7
  million (approximately: 2 million for the deposit, 700k for the arbitrage --
  these vary and can be hard to predict exactly).
- `rocketarb` will try to ensure to refund at least some (700k gas worth by
  default, change it with the `--gas-refund` option) of your gas costs with the
  arbitrage profits. This is ensured by making the arbitrage transaction revert
  (`not enough profit`) if it does not produce at least this much profit.
- If your bundle is not getting included (`BlockPassedWithoutInclusion`) most
  likely the transactions are reverting with some failure (try the `--dry-run`
  option to investigate), or the gas fees are too low for the current base fee.
- Every time we ask the smartnode for a deposit transaction, it increments its
  internal validator index (saved in your node's wallet file). If you need to
  re-run (e.g., to use a different maximum fee per gas) and don't want to waste
  the index, you can manually decrement it by editing the wallet file. Wasting
  indices is not a problem for your node, however - the generated validator
  keys for an unused index will simply remain unused.
