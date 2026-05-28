# AutoRange Depth Simulator

A single page, static tool that compares the instantaneous liquidity depth of a concentrated AutoRange band against a full range weighted pool holding the same capital. Paste a token contract address, it fetches the live price, build a pair from two saved tokens, and the model seeds itself from the live pair price.

No backend, no build step, no API key. Everything runs in the browser.

## Deploy to GitHub Pages

1. Create a repo (or use an existing one) and put `index.html` at the root.
2. Push it to `main`.
3. Repo Settings, then Pages, set Source to `Deploy from a branch`, Branch `main`, folder `/ (root)`, Save.
4. Wait about a minute. The site is live at `https://<user>.github.io/<repo>/`.

That is the whole deploy. To serve it locally instead: `python3 -m http.server` from the folder, then open `http://localhost:8000`.

## How it works

Token registry (chain-scoped). Pick a chain, paste an ERC20 contract address, Add. The app calls the DefiLlama coins API and stores the symbol, price, and confidence. Each saved-token chip shows a confidence dot — gold when DefiLlama's price confidence is below 90%, so you can spot thin or stale prices before trusting them. CoinGecko ids are also supported (pick `CoinGecko ID` and enter a slug like `bitcoin`). Supported chains include Ethereum, Arbitrum, Base, Optimism, Polygon, BNB Chain, Avalanche, Gnosis, and Monad. Saved tokens persist in the browser via localStorage.

Two clearly separated jobs:

- **Token library** (managing your tokens) lives in a collapsible drawer at the top, collapsed by default. This is where you add and remove tokens — paste an address, hit Add, or remove a chip. The slim bar summarizes what you have saved (e.g. `5 tokens saved · 2 on Ethereum`). New tokens are added to the active chain (set under Pool setup). Open/closed state is remembered between visits.
- **Pair selection** (choosing what to analyze) lives in the main **Pool setup** panel, because it is the core action you take every time. Pick a Chain, then a Base token (X) and Quote token (Y) from the tokens saved on that chain. A `+ manage tokens` link there opens the library drawer when you need to add something.

The selected chain is the active context: the library chips and the Base/Quote pickers only show tokens on that chain, so you can only build a pair from same-chain tokens (which is what a real pool is).

Pair price is `P = priceX / priceY` in USD, which reads as Y per X. "Use pair" writes that price into the band and, if auto center is on, sets `Pa = P / (1 + w)` and `Pb = P * (1 + w)` so the price sits at the geometric center.

Model. Depth is the liquidity constant `L`. For equal pool value the multiplier at price P is

    E = 2 sqrt(P) / ( 2 sqrt(P) - P / sqrt(Pb) - sqrt(Pa) )

At the band center `P = sqrt(Pa * Pb)` this reduces to

    E = 1 / ( 1 - (Pa / Pb) ^ (1/4) )

The trade probe runs a constant product swap on each book's virtual reserves and reports output, execution price, and slippage. The AutoRange book stops filling once price reaches the band edge.

## Notes

- Default registry seeds WBTC and WETH so the tool is usable on first load.
- Live pricing needs the page served over http or https. Opening the raw file with `file://`, or viewing it inside a chat preview, will block the external request; in that case enter the band manually, the model still works.
- Price source: DefiLlama coins API, `https://coins.llama.fi/prices/current/{chain}:{address}`. Open and free; citing DefiLlama is appreciated.
- To change the default pair, edit the seed block in the `init()` function at the bottom of `index.html`.
