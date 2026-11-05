# The Keys Were Safe, the Money Wasn't: How Binance Lost $6.3 Billion

*5 November 2026*

## The surprise

On 29 October 2026, Binance lost about $6.3 billion in roughly 20 minutes. It was the biggest crypto theft ever recorded.

When I first read about it, I assumed the hackers had stolen Binance's private keys. That's the obvious thing to steal. But they didn't. Binance says it found no evidence that any private key leaked.

The keys were safe the whole time. The money wasn't. This article explains how both of those things can be true, and why it matters for anyone who builds or protects systems that move money.

## What happened

At 20:05 UTC on 29 October, money started leaving Binance's hot wallets on ten different blockchains at the same time. Most of it was gone within about 20 minutes.

- **How much:** about $6.34 billion by Binance's count. Blockchain analytics firms put it between $6.36 and $6.43 billion.
- **What was saved:** about $2.85 billion was frozen, mostly stablecoins. Companies like Tether and Circle can freeze their own coins, and they acted within about an hour.
- **Who is blamed:** the FBI attributed the theft to TraderTraitor, a North Korean hacking group, two days later. North Korea denies it.
- **Who pays:** Binance says it is covering all losses with its own funds. Customer withdrawals restarted on 1 November.

## How they got in

This was not one mistake. According to Binance's own post-mortem, it was a campaign that took about five months and went through three engineers' computers.

1. **June: a fake download.** A Binance engineer ("Engineer A") searched online for a popular AI coding tool and installed it through a tampered Homebrew download. Binance caught it and replaced his computer. But the attackers had already learned who worked on the wallet team.
2. **September: a fake job interview.** Another engineer ("Engineer B") was contacted on a real job-hunting site by someone posing as a project director. After an interview with a fake HR person, he had a second interview with a fake "CEO" in a chat app. At the end, he was asked to download a coding assignment onto his work computer. It contained new malware that Binance's security tools did not detect.
3. **20 October: moving sideways.** From Engineer B's computer, the attackers used an internal Binance AI tool to take control of Engineer A's computer and a third engineer's. Binance hasn't explained how. Security people call this lateral movement: going from one machine to the next until you find the right access.
4. **The key that opened the door.** Engineer A's computer had a token that allowed deploying software containers into Binance's systems.
5. **29 October: the attack.** Between 17:15 and 19:56 UTC, the attackers used that token to deploy their own container and run a script. That gave them access to the **wallet signing service**. At 20:05, the money started moving.

The attackers never broke the cryptography. They went through people and the tools those people used.

## The trick: making the signer work for them

To move crypto, a transaction needs a valid signature made with a private key. At Binance, a service called the signing service creates those signatures. It uses MPC (multi-party computation), which means the key is split into pieces and is never stored in one place. That's why there was no key to steal.

But the attackers didn't need to steal it. **They took control of the signing service, so Binance's own system used the keys for them.**

Think of a bank manager whose signature nobody can copy. A thief sneaks into the office and puts a payment order on the manager's desk. The manager signs it, because papers on that desk usually come from trusted staff. The bank sees a real signature and sends the money.

The blockchain works like that bank. It checks only one thing: is the signature valid? It does not check who asked for it, or why. The signatures were real, so the blockchain accepted every transaction.

### Why pausing withdrawals didn't stop it

On most chains, the attackers simply had the signer approve normal transfers. On Ethereum, they used a newer feature called EIP-7702. It lets a wallet hand over control to a smart contract. The signer approved 50 of these authorizations, which pointed one of Binance's wallets at a contract the attackers controlled.

That authorization lived on the blockchain itself, not inside Binance's systems. So when Binance paused withdrawals, the attacker's contract kept working. And because of MPC, there was no single key Binance could quickly use to cancel it. The last stolen transfer went out at 23:56 UTC. Binance only removed the authorization at 00:10.

## What should change

If I could add one protection, I would limit what Binance's internal AI tool could reach. The attackers first broke into Engineer B's computer, but he didn't work on the wallets, so on its own that was a dead end. The AI tool is what let them jump to the wallet engineers' computers. AI tools are still new, and companies keep giving them more access because they're useful. But the more an AI tool can reach, the more an attacker can reach through it. Any AI tool connected to engineers' machines should be treated like a privileged account, with the smallest access it needs.

**Protecting the keys is not enough, because the attackers never needed them. They just needed Binance to sign.**

## Sources

Facts are as of 5 November 2026; the investigation is ongoing.

- [Security Affairs: Binance post-mortem details](https://securityaffairs.com/202369/cyber-crime/binance-hack-how-fake-job-interviews-and-ai-abuse-led-to-a-6-34-billion-attack.html)
- [The Hacker News: how fake job interviews and AI tool abuse led to the theft](https://thehackernews.com/2026/11/binance-reveals-how-fake-job-interviews.html)
- [Stingrai: Binance Hack (October 2026)](https://www.stingrai.io/blog/binance-hack-october-2026)
- [Elliptic: How North Korea is laundering the $6.4 billion stolen from Binance](https://www.elliptic.co/insights/how-north-korea-is-laundering-the-6-4-billion-stolen-from-binance/)
- [SecurityWeek: FBI flags over 600 addresses tied to the Binance heist](https://www.securityweek.com/fbi-flags-over-600-addresses-tied-to-north-koreas-binance-heist/)
- [Arab News (AFP): North Korea rejects FBI allegations](https://www.arabnews.pk/world/n-korea-slams-fbi-allegations-over-binance-hack-as-grave-provocation-3005940)

*Research and drafting assisted by AI.*
