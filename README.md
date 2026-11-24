# Unlock Encrypted Data with a Fiber Payment (Basic)

**Tutorial:** [Unlock Encrypted Data with a Fiber Payment](https://fiber.world/docs/build/encrypted-data-payment)

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

![Both channels at CHANNEL_READY](<screenshots/Screenshot (262).png>)

| Channel | Balances after opening |
|---|---|
| A–B | Buyer A 400 CKB · Bottle B 150 CKB |
| C–B | Bottle B 150 CKB · Seller C 400 CKB |

I also tried the optional liquidity demo, sending 5 CKB from Seller C to Bottle B, which increased Seller C's inbound liquidity on the B–C channel to about 155 / 395.

---

## 2. Encrypt and create the invoice

As Seller C, I encrypted my own sample text. The browser generated a fresh 256-bit AES key and a 96-bit IV, and the AES key became the payment preimage of a 1 CKB invoice.

![Original text and ciphertext](<screenshots/Screenshot (289).png>)

![Invoice created with payment hash](<screenshots/Screenshot (292).png>)

---

## 3. Before payment: decryption rejected

The encrypted offer (ciphertext, IV, invoice, payment hash and last hop) was delivered to Buyer A. Selecting **Decrypt without key** was rejected: AES-GCM refused a substitute key.

![Encrypted offer and Decryption rejected](<screenshots/Screenshot (295).png>)

---

## 4. After payment: 1 CKB paid

Buyer A paid the invoice through Bottle B. Seller C received 1 CKB; Bottle B earned a 0.001 CKB routing fee. Before paying, Buyer A's app validated the invoice's network, amount and payment hash against the offer. The payment hash matches the invoice from Step 2.

![Payment completed with payment hash](<screenshots/Screenshot (298).png>)

| Channel | Balance change |
|---|---|
| A–B | Buyer A 398.999 CKB (−1.001) · Bottle B 151.001 CKB (+1.001) |
| B–C | Bottle B 154 CKB (−1) · Seller C 396 CKB (+1) |

---

## 5. After decryption: original text recovered

Seller C revealed the preimage to claim the payment, Fiber returned it to Buyer A, and Buyer A used it as the AES key. The decrypted result matches my original sample text from Step 2 exactly.

![Decrypted result matching the original](<screenshots/Screenshot (300).png>)

---

## Bonus: independent key verification

The tutorial states that the payment hash is the SHA-256 fingerprint of the preimage. I checked this myself in the browser console using the Web Crypto API:

```js
const hex = "9030b367810e7c4c395f56460f508ea2a5f1e56617abb35ffeef12c896f6feb5";
const bytes = new Uint8Array(hex.match(/../g).map(h => parseInt(h, 16)));
const hash = await crypto.subtle.digest("SHA-256", bytes);
console.log([...new Uint8Array(hash)].map(b => b.toString(16).padStart(2, "0")).join(""));
```

**Output:** `1115a70202218b56ef0d80ed2bb04c4c2f1cd9b909e94304699efd26f24b4709`, an exact match with the invoice payment hash.

![SHA-256 of the preimage equals the payment hash](<screenshots/Screenshot (303).png>)

The node logs in the console also showed the route built for `100000000` shannons (1 CKB) with a maximum fee of `500000` shannons (0.005 CKB).

---

## Personal reflection

### How did it feel to have the explanations alongside each interactive step?

I have participated in all the previous Build on CKB campaigns, so I've used both formats, and I prefer having the explanation beside each step. Having the explanation right next to each action meant I understood what I was doing as I did it.

The explanation that the AES key is also the payment preimage was the moment it clicked for me. I'd just watched my own text turn into ciphertext, and right beside it the page explained that the same secret locks the invoice. So paying and receiving the key are one action, with no server in between. Reading that separately from the demo, I don't think I'd have connected the two.

The one place an explanation was missing was when my node failed. The raw script error explained nothing, and the cause was just my connection.

### What surprised you or stood out?

It surprised me that Seller C had to send 5 CKB to Bottle B to be able to receive more. In normal payments, receiving costs nothing, but in a channel, room to receive is something you create.

I also checked the key myself. After decrypting, I ran SHA-256 on the revealed preimage in the browser console, and it matched the invoice payment hash exactly (…f24b4709). So the key I received was the exact one the invoice was locked to.

### Was there a step where you were unsure what to do next?

At Step 3 (Transfer), the offer was delivered to Buyer A automatically, and I wasn't sure if I'd missed a step, so I had to ask. It felt like I'd skipped something. A clear "Offer delivered to Buyer A ✓" message would help.

I was also unsure while opening the channels. I didn't know if reloading would break the channel or lose funds, and I was worried about losing the 500 CKB.

### If you ran into a problem, what did you expect, what happened, and how did you resolve it?

I expected both nodes to start. Instead, both showed a long "Cannot resolve cell dep" error I couldn't understand, and retrying gave a different contract ID each time. After about 11 minutes, turning my Wi-Fi off and on fixed it instantly. At first I felt confused and frustrated, and when the fix was that simple I felt relieved, but annoyed it took so long. The error message didn't tell me it was just a connection problem.

Network issues also stopped my channels from opening that night. I only finished the next day after restarting the nodes, and then both channels reached CHANNEL_READY.

A smaller issue: if I copied the funding address before clicking Get Testnet CKB, the paste didn't work on the faucet. I had to go back and copy again, twice, once per node. It felt minor but annoying. Pre-filling the address would fix it.

### What would you like to build using this pay-to-unlock flow?

The "What does payment guarantee?" quiz confirmed what I suspected: paying proves the key matches the ciphertext, not that the report is accurate. That shaped my idea: a DevOps delivery flow where a freelancer sells a script or config file, with a preview before payment, a listing commitment so every buyer gets the same advertised file, and a proof-of-purchase receipt in case of disputes, as the tutorial suggests.

### If you could change one thing about the tutorial, what would it be?

Replace the raw node error with a plain message like "Couldn't reach CKB Testnet, check your connection" and a Retry button. It cost me about 11 minutes at the very first step, the point where a new user is most likely to give up.

---

## Appendix: Friction log

| Step | What happened | Resolution |
|---|---|---|
| Setup: start nodes | Both WASM nodes failed with `Cannot resolve cell dep for type id`; each retry failed on a different contract ID | Turned Wi-Fi off and on; both nodes started instantly (~11 min lost) |
| Setup: faucet | Copying the address before clicking **Get Testnet CKB** meant the paste failed on the faucet | Clicked the button first, then copied; 10,000 Testnet CKB per node |
| Setup: channels | Unsure whether reloading during channel opening was safe; network issues stopped the channels that night | Restarted the nodes Tuesday; both reached `CHANNEL_READY` at ~1:19 pm |
| Transfer | Offer delivered to Buyer A automatically; unsure whether I'd missed a step | Confirmed and continued |
| Steps 2–5 | Encrypt, invoice, transfer, pay and decrypt completed with no issues | ~14 minutes on a stable connection |

![Node error at startup](<screenshots/Screenshot (260).png>)

### Suggested improvements
1. **Plain-language node error with a Retry button** when the node can't reach CKB Testnet.
2. **Pre-fill the funding address** when **Get Testnet CKB** opens the faucet.
3. **A clear "Offer delivered to Buyer A ✓" confirmation** at the Transfer step.
4. **A note on whether reloading is safe** while channels are opening.
5. **Relabel "Seller's result"** on the decrypt step as "Buyer A's decrypted report."
