# DXD

**Constant-product pools on Dogecoin.** The intent is a Doginals inscription (`p:"dxd"`). The DOGE and the tokens sit at a published vault address. An indexer updates the reserves. A separate signer delivers the payout.

DXD is a venue with a hot vault. It is not a covenant, and the program does not hold the coins. Anyone can recompute the reserves from the inscriptions and the vault's outputs. The key holder can still move those outputs. The trustworthy part is that the address, the curve, and the fee are public, and a second indexer can refuse a payout that does not match.

Reference indexer: [dogex](https://github.com/jonheaven/dogex) (the rules in this repo are the ones that indexer implements). Normative text: [spec.md](spec.md).

| | |
| --- | --- |
| Status | Launch draft. Live pools use this wire. |
| Carrier | Doginals inscription, JSON body |
| Marker | `p` = `dxd`, `v` = `1` |
| Curve | `x·y = k` |
| Assets | DOGE, Ðunes, ÐogeTreats, DRC-20 |
| License | [MIT](LICENSE) |
