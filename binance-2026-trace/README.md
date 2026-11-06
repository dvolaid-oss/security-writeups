# Tracing Exploiter 8: From Binance's Hot Wallets to a $438M Holding Wallet

*6 November 2026*

## Summary

Using only public Etherscan data, I traced one attacker address from the 29 October 2026 Binance theft back to its source. The trail links a $438M holding wallet directly to an EIP-7702 "sweep" executed through one of Binance's own hot wallets.

Key findings:

1. **Exploiter 8** (`0x8532FA14…900543D4`) holds about **197,460 ETH (≈ $438M)**. No outgoing transfers were observed in the history I reviewed; it appears to be a holding wallet.
2. Its first and largest inflow, **195,460 ETH**, came from **Exploiter 2** at **20:38:47 UTC**, 33 minutes after the theft began.
3. Exploiter 2 was funded mainly by a **167,464.24 ETH internal transfer from Binance 14**, triggered by a `sweep()` call while Binance 14 was delegated via EIP-7702 to an attacker contract (**Exploiter 24**). A further 28,005.6 ETH came from Binance 16.
4. Binance 14 continued to be swept after the initial drain (1,699.92 ETH and 369.83 ETH), consistent with Binance's statement that pausing withdrawals did not stop the delegated wallet.

## Scope and method

- **Scope:** one address (Exploiter 8) and the wallet that funded it. This is not a full trace; Elliptic counts about 12,000 wallets involved in the laundering.
- **Starting point:** the FBI alert of 31 October 2026, which lists 628 addresses linked to the theft.
- **Tools:** Etherscan only (normal transactions, internal transactions, cross-chain view, transaction input data). No paid analytics.
- **Approach:** review each wallet's oldest transactions, reconcile inflows against balances, and test hypotheses against published figures from TRM Labs and Elliptic.
- **Data as of:** 6 November 2026. Balances may have changed since.

## Finding 1: Exploiter 8 is a holding wallet

| Item | Observation |
| --- | --- |
| Address | `0x8532FA14ACAf483729C449b8458aedDb900543D4` |
| Etherscan label | Binance Hot Wallet Exploiter 8 (tags: Exploit, Heist) |
| Balance | 197,460.016 ETH (≈ $438.1M at $2,218.62/ETH) |
| Normal transactions | 622; newest and oldest pages reviewed, all inbound |
| Internal transactions | 0 |
| Cross-chain activity | None recorded by Etherscan's bridge view |
| Token holdings | $16.71 across 16 tokens (spam) |

Almost all of the 622 transactions are dust or spam: 0 ETH or fractions of a cent, sent by unrelated addresses and bots. The spam began within minutes of the wallet's first inflow.

One sender is an **address-poisoning** attempt: `0x853244c7…1DBd543D4` copies the prefix and suffix of `0x8532FA14…900543D4`, so anyone copying an address from the transaction history could pick the wrong one. Even attacker wallets attract other scammers.

**Assessment (medium confidence):** Exploiter 8 is a holding wallet. This fits Elliptic's report that a small number of large attacker wallets hold most of the stolen ETH and have not moved since 30 October. I have not confirmed it is one of the nine wallets Elliptic describes.

## Finding 2: Exploiter 8 was funded by Exploiter 2 during the attack

Exploiter 8's first transaction is also its largest:

| Item | Observation |
| --- | --- |
| Transaction | `0x551d7961652e6d9835b11988908adf86ebe480badb4ce2d7d52193b2b1b02c76` |
| Block / time | 26300842 / 29 Oct 2026, 20:38:47 UTC |
| From | Binance Hot Wallet Exploiter 2 (`0x5227f603C1eE34CE3C06eFe6477003BDfCCB2f8c`) |
| Value | 195,460 ETH (≈ $433.9M) |
| Fee | 0.0000389 ETH (≈ $0.09) |

The transfer happened 33 minutes after outflows began at 20:05 UTC, so the attacker was already consolidating funds into holding wallets while the theft was in progress. The round amount suggests a chosen, likely scripted, transfer.

Exploiter 2's own history shows how it was funded and used:

- **Inflows:** 28,005.6 ETH from **Binance 16** (a Binance hot wallet, block 26300772) and, in its internal transactions, 167,464.24 ETH from **Binance 14** (block 26300770).
- **Test transfers:** two 0.5 ETH sends to another Exploiter wallet (blocks 26300794 and 26300806) immediately before the 195,460 ETH transfer.
- **Token movements:** calls to the Tether USDT and Ripple RLUSD contracts, indicating stablecoin transfers.
- **Swaps:** many later internal inflows from Uniswap V3: Router 2 (about 5 to 231 ETH each), consistent with converting other stolen tokens into ETH.

**Reconciliation:** the two Binance inflows total 195,469.84 ETH, which covers the 195,460 ETH sent to Exploiter 8 plus the test transfers and fees. Before checking the internal transactions, I predicted this from TRM Labs' reported 167,464 ETH sweep; the internal transfer confirmed it.

## Finding 3: the 167,464 ETH came from an EIP-7702 sweep of Binance 14

An externally owned wallet cannot normally produce internal transactions; only contract code does. Binance 14 produced one, so I examined the parent transaction of the 167,464.24 ETH transfer (`0x53bd38b816…`, block 26300770):

| Field | Value | Interpretation |
| --- | --- | --- |
| Delegated Address | `0x63e842B054Bd624a465454D533C33F573e9c6fAe` (Exploiter 24) | Binance 14 was delegated via EIP-7702 to an attacker contract |
| Function | `sweep(address pool, address[] depositors)` | The contract exposes a sweep function |
| Input [0], `pool` | `0x5227f603…bdfccb2f8c` | Destination: Exploiter 2 |
| Input [4] | `0xdac17f958d2ee523a2206206994597c13d831ec7` | The Tether USDT contract, so tokens were targeted as well as ETH |
| Txn type | 2 (EIP-1559) | A call that *used* an existing delegation, not the authorization itself |

The call instructed Binance's own wallet to transfer its holdings to Exploiter 2, and the wallet complied because it was executing the attacker's code. This matches TRM Labs' description of 50 pre-signed EIP-7702 authorizations turning a Binance hot wallet into a sweeper, and the presence of the USDT contract fits TRM's report that the sweep also took tokens.

Two further internal transfers from Binance 14 to Exploiter 2 (1,699.92 ETH in block 26300968 and 369.83 ETH in block 26301167) show the delegated wallet being swept again as new funds arrived. Binance has said the delegation operated independently of its withdrawal system and was only revoked at 00:10 UTC on 30 October.

**Assessment (high confidence):** the bulk of Exploiter 8's ETH originates from an EIP-7702 sweep of Binance 14, routed through Exploiter 2. The on-chain evidence is consistent with Binance's statement that no private key was leaked: the wallet's own signing authority was redirected rather than its key stolen.

## Fund flow and indicators

```
Binance 14 --(EIP-7702 sweep) 167,464.24 ETH--+
Binance 16 --28,005.6 ETH------------------------+--> Exploiter 2 --195,460 ETH--> Exploiter 8
Uniswap V3 --swapped tokens into ETH-------------+                              (~197,460 ETH held)
```

| Role | Address / transaction | Notes |
| --- | --- | --- |
| Holding wallet | `0x8532FA14ACAf483729C449b8458aedDb900543D4` | Exploiter 8; ~197,460 ETH as of 6 Nov 2026 |
| Distribution wallet | `0x5227f603C1eE34CE3C06eFe6477003BDfCCB2f8c` | Exploiter 2; received sweep and Binance 16 funds |
| Sweeper contract | `0x63e842B054Bd624a465454D533C33F573e9c6fAe` | Exploiter 24; EIP-7702 delegate of Binance 14 |
| Address poisoning | `0x853244c7…1DBd543D4` | Look-alike of Exploiter 8; not attacker infrastructure |
| Key transfer | `0x551d7961652e6d9835b11988908adf86ebe480badb4ce2d7d52193b2b1b02c76` | 195,460 ETH, Exploiter 2 to Exploiter 8 |

## Limitations and open questions

- **Unreconciled balance:** Exploiter 8 received 195,460 ETH from Exploiter 2 but holds about 197,460 ETH. The source of the remaining ~2,000 ETH is not yet identified; I reviewed only the first and last pages of its 622 transactions.
- **Holding status:** "no outgoing transfers" is based on the pages reviewed, not a full export of the transaction history.
- **Sweep input:** the encoded `depositors` list appears to contain 65 entries; I have not decoded it fully.
- **Labels:** wallet names such as "Exploiter 8" are Etherscan labels, and attribution to North Korea's TraderTraitor rests on the FBI and analytics firms, not on this analysis.
- **What to watch:** if this ETH moves, the pattern seen after Bybit (swaps into Bitcoin, then OTC cash-out) would be the expected next step.

## Notes for anyone repeating this

- On high-profile addresses, most transactions are spam. Start from the oldest transactions, not the newest.
- ETH moved by contract code appears under internal transactions, not the main transaction list.
- Reconcile inflows against balances; a gap usually points to the next thing to check.
- Historical transactions keep evidence that current wallet state does not: Binance 14 shows no delegation today, but the sweep transaction still records it.

## Sources

On-chain data from Etherscan, checked 6 November 2026. Balances change; check the links for the current state.

- [Etherscan: Exploiter 8](https://etherscan.io/address/0x8532FA14ACAf483729C449b8458aedDb900543D4)
- [Etherscan: Exploiter 2](https://etherscan.io/address/0x5227f603C1eE34CE3C06eFe6477003BDfCCB2f8c)
- [Etherscan: Exploiter 2 internal transactions](https://etherscan.io/txsInternal?a=0x5227f603C1eE34CE3C06eFe6477003BDfCCB2f8c)
- [Etherscan: Exploiter 2 to Exploiter 8 transfer](https://etherscan.io/tx/0x551d7961652e6d9835b11988908adf86ebe480badb4ce2d7d52193b2b1b02c76)
- [Etherscan: Exploiter 24 (sweeper contract)](https://etherscan.io/address/0x63e842B054Bd624a465454D533C33F573e9c6fAe)
- [FBI: North Korea Responsible for $6 Billion+ Binance Attack](https://www.fbi.gov/investigate/cyber/alerts/2026/north-korea-responsible-for-6-billion-binance-attack)
- [Stingrai: Binance Hack (October 2026)](https://www.stingrai.io/blog/binance-hack-october-2026), summarizing TRM Labs' on-chain findings
- [Elliptic: How North Korea is laundering the $6.4 billion stolen from Binance](https://www.elliptic.co/insights/how-north-korea-is-laundering-the-6-4-billion-stolen-from-binance/)
- [Security Affairs: Binance post-mortem details](https://securityaffairs.com/202369/cyber-crime/binance-hack-how-fake-job-interviews-and-ai-abuse-led-to-a-6-34-billion-attack.html)

*On-chain research done by me; writing assisted by AI.*
