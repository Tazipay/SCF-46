# TaziPay: Planned Stellar Integrations

> **Status:** planned. These integrations are built and tested on Stellar **testnet** first, then launched on **mainnet**.

| Integration                  | Role in TaziPay                                                    | Standards / interfaces                                            |
| ---------------------------- | ------------------------------------------------------------------ | ----------------------------------------------------------------- |
| **Stellar settlement layer** | Receive and hold every deposit as native USDC on Stellar           | SEP-23 muxed accounts, sponsored reserves, fee bumps, Horizon/RPC |
| **Circle CCTP**              | Bring native USDC from other chains to Stellar (burn and mint)     | Circle attestation API                                            |
| **NEAR Intents (1Click)**    | Route non-USDC crypto into native USDC on Stellar                  | 1Click exact-output quotes                                        |
| **Privy**                    | Provide the user's signing key for their co-signed Stellar account | Embedded wallet / key management                                  |
| **MoneyGram Access**         | Cash pickup and cross-border cash-out                              | SEP-10 (auth), SEP-24 (hosted withdrawal)                         |
| **DeFindex**                 | Opt-in USDC savings vault                                          | Soroban smart contracts                                           |

## Stellar settlement layer and direct USDC intake

- **Accounts:** a settlement account (holds USDC), a sponsor account (pays reserves for user accounts) and a fee account (pays fees through fee-bump transactions). Users never need to hold XLM.
- **Direct deposits:** each payment gets its own **muxed address (SEP-23)** on the settlement account, so incoming USDC is matched to the right payment by muxed ID with no memo errors.
- **Confirmation:** TaziPay's own Horizon/RPC monitoring watches the settlement account. A deposit posts READY only after it is final on chain and matched.

## Circle CCTP: native USDC from other chains

- The user sends native USDC on a CCTP-supported chain.
- TaziPay burns it on the source chain, **verifies the Circle attestation**, then mints native USDC to the Stellar settlement account.
- Replays are idempotent: the same burn never posts twice.

## NEAR Intents (1Click): everything that isn't USDC

- Other supported assets are routed through a **1Click exact-output quote** that delivers native USDC on Stellar.
- Every quote has a **deadline and slippage bound**. A missed deadline or failed fill is **refunded**, and the ledger records the refund.

## Privy: co-signed user accounts

- For cash-out and savings, TaziPay moves the reserved USDC into the **user's own Stellar account**.
- The account is **2-of-2 multisig**: the user's key held in Privy, plus TaziPay's key in **AWS KMS**. The master key weight is 0.
- TaziPay **sponsors the account reserves** and the USDC trustline, and pays fees with fee bumps. Users don't manage XLM or a seed phrase.
- Neither TaziPay nor the user can move the funds alone, and anyone can check the signers on the ledger (for example on Stellar Expert).

## MoneyGram Access: cash-out

- The user signs **SEP-10** authentication from the co-signed account (both keys plus `client_domain`).
- The user completes MoneyGram's **hosted SEP-24 withdrawal** and collects cash with a reference number, or sends money across borders.
- Corridors, limits and fees come **live from MoneyGram**, never hard-coded.
- The ledger **reserves the funds first**. Before TaziPay co-signs, its policy engine checks the destination, memo and amount against MoneyGram's transaction record.
- An abandoned cash-out returns its funds to READY.

## DeFindex: opt-in savings

- **Off by default.** Deposit and withdrawal only.
- Users can move part of a READY balance **above a reserve they set** into a pinned DeFindex USDC vault on Soroban, and withdraw it before spending or cashing out.
- Vault shares stay in the user's co-signed account.
- Controls: vault **allowlist**, user and portfolio caps, valuation from the vault share price. New deposits stop if the vault or its strategies change or the valuation goes stale.
- The value can go down as well as up, and the app says so.
