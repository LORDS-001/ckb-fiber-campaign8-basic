# Build on CKB: Campaign #8 — Unlock Encrypted Data with a Fiber Payment (Basic)

**Participant:** L.O.R.D.S · Lagos, Nigeria
**Completed:** Tuesday, 24 November 2026
**Tutorial:** [Unlock Encrypted Data with a Fiber Payment](https://fiber.world/docs/build/encrypted-data-payment)
**Network:** CKB Testnet (Testnet CKB only)
**Environment:** Chrome on Windows, browser-based Fiber WASM nodes

---

## Summary

I completed the pay-to-unlock flow with two browser-based Fiber nodes. Seller C encrypted my own sample text with AES-256-GCM and used the AES key as the payment preimage of a 1 CKB invoice. Buyer A paid through Bottle B, and the payment revealed the key, which decrypted the report. I also independently verified that the revealed key hashes to the invoice's payment hash.

| Item | Value |
|---|---|
| Sample text | `DevOps delivery for client by L.O.R.D.S, Lagos, 24 Nov 2026` |
| Invoice amount | 1 CKB |
| Routing fee (Bottle B) | 0.001 CKB |
| Total paid | 1.001 CKB |
| Payment hash | `0x1115a70202218b56ef0d80ed2bb04c4c2f1cd9b909e94304699efd26f24b4709` |
| Revealed preimage (AES key) | `0x9030b367810e7c4c395f56460f508ea2a5f1e56617abb35ffeef12c896f6feb5` |
| Invoice status | Paid |

---

## 1. Setup: both channels at CHANNEL_READY

Buyer A and Seller C each opened a channel with Bottle B (500 CKB per channel: 100 reserved for settlement, 400 spendable). Both channels passed all six opening stages and reached `CHANNEL_READY`.

![Both channels at CHANNEL_READY](screenshots/1-setup.png)

| Channel | Balances after opening |
|---|---|
| A–B | Buyer A 400 CKB · Bottle B 150 CKB |
| C–B | Bottle B 150 CKB · Seller C 400 CKB |

I also tried the optional liquidity demo, sending 5 CKB from Seller C to Bottle B, which increased Seller C's inbound liquidity on the B–C channel to about 155 / 395.

---

## 2. Encrypt and create the invoice

As Seller C, I encrypted my own sample text. The browser generated a fresh 256-bit AES key and a 96-bit IV, and the AES key became the payment preimage of a 1 CKB invoice.

![Original text and ciphertext](screenshots/step2-original-text.png)

![Invoice created with payment hash](screenshots/invoice-created.png)

---

## 3. Before payment: decryption rejected

The encrypted offer (ciphertext, IV, invoice, payment hash and last hop) was delivered to Buyer A. Selecting **Decrypt without key** was rejected: AES-GCM refused a substitute key.

![Encrypted offer and Decryption rejected](screenshots/2-before-payment.png)

---

## 4. After payment: 1 CKB paid

Buyer A paid the invoice through Bottle B. Seller C received 1 CKB; Bottle B earned a 0.001 CKB routing fee. Before paying, Buyer A's app validated the invoice's network, amount and payment hash against the offer. The payment hash matches the invoice from Step 2.

![Payment completed with payment hash](screenshots/3-after-payment.png)

| Channel | Balance change |
|---|---|
| A–B | Buyer A 398.999 CKB (−1.001) · Bottle B 151.001 CKB (+1.001) |
| B–C | Bottle B 154 CKB (−1) · Seller C 396 CKB (+1) |

---

## 5. After decryption: original text recovered

Seller C revealed the preimage to claim the payment, Fiber returned it to Buyer A, and Buyer A used it as the AES key. The decrypted result matches my original sample text from Step 2 exactly.

![Decrypted result matching the original](screenshots/4-after-decryption.png)

---

## ⭐ Bonus: independent key verification

The tutorial states that the payment hash is the SHA-256 fingerprint of the preimage. I checked this myself in the browser console using the Web Crypto API:

```js
const hex = "9030b367810e7c4c395f56460f508ea2a5f1e56617abb35ffeef12c896f6feb5";
const bytes = new Uint8Array(hex.match(/../g).map(h => parseInt(h, 16)));
const hash = await crypto.subtle.digest("SHA-256", bytes);
console.log([...new Uint8Array(hash)].map(b => b.toString(16).padStart(2, "0")).join(""));
```

**Output:** `1115a70202218b56ef0d80ed2bb04c4c2f1cd9b909e94304699efd26f24b4709`, an exact match with the invoice payment hash.

![SHA-256 of the preimage equals the payment hash](screenshots/bonus-key-verification.png)

The node logs in the console also showed the route built for `100000000` shannons (1 CKB) with a maximum fee of `500000` shannons (0.005 CKB).

---

## Personal reflection

<!-- Paste your final reviewed reflection here -->

---

## Appendix: Friction log

| Time (WAT) | Step | What happened | Resolution |
|---|---|---|---|
| Mon 9:52–10:03 pm | Setup: start nodes | Both WASM nodes failed with `Cannot resolve cell dep for type id`; each retry failed on a different contract ID | Turned Wi-Fi off and on; both nodes started instantly (~11 min lost) |
| Mon ~10:04–10:08 pm | Setup: faucet | Copying the address before clicking **Get Testnet CKB** meant the paste failed on the faucet | Clicked the button first, then copied; 10,000 Testnet CKB per node |
| Mon ~10:21 pm | Setup: channels | Unsure whether reloading during channel opening was safe; network issues stopped the channels that night | Restarted the nodes Tuesday; both reached `CHANNEL_READY` at ~1:19 pm |
| Tue ~1:28 pm | Transfer | Offer delivered to Buyer A automatically; unsure whether I'd missed a step | Confirmed and continued |
| Tue 1:19–1:33 pm | Steps 2–5 | Encrypt, invoice, transfer, pay and decrypt completed with no issues | ~14 minutes on a stable connection |

![Node error at startup](screenshots/node-error.png)

### Suggested improvements
1. **Plain-language node error with a Retry button** when the node can't reach CKB Testnet.
2. **Pre-fill the funding address** when **Get Testnet CKB** opens the faucet.
3. **A clear "Offer delivered to Buyer A ✓" confirmation** at the Transfer step.
4. **A note on whether reloading is safe** while channels are opening.
5. **Relabel "Seller's result"** on the decrypt step as "Buyer A's decrypted report."

---

*All activity used CKB Testnet funds only. No private keys, seed phrases or browser-storage data are included in this repository.*
