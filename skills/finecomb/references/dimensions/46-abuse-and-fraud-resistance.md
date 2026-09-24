# 46 Abuse and fraud resistance

> This dimension looks at features that **work as designed but can be used for harm**: to send spam or phishing, to commit fraud, to harass people, or to get free resources. It applies to any product with users, accounts, messages, uploads or value. Apply your own knowledge of how similar products have been abused to each row.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Messages sent in our name** | Invitations, shares, comments, notifications, password resets and contact forms carry attacker-chosen text, links or recipients, so the product sends spam or phishing from a trusted domain (see [4.40](../specialties/4.40-notifications-and-outbound-messages.md)) |
| **Free resources** | Free tiers, trials, sandboxes, build minutes, storage and compute can be used for mining, file hosting, proxies or attack traffic; limits apply per request rather than per real account |
| Fake and mass accounts | Sign-up and verification do not resist bulk account creation; rewards, credits and referral bonuses can be farmed with fake or linked accounts |
| Fraud in value flows | Refunds, chargebacks, coupons, gift cards, promotions and points can be looped or combined for profit (see [9](9-business-logic-and-flow-integrity.md) and [4.39](../specialties/4.39-payments-accounting-and-billing.md)) |
| Harassment and unwanted contact | People cannot block, mute or report; a blocked user can still reach them through another feature (mentions, invitations, shared spaces); location or contact details are visible to strangers by default |
| Content others will open | Uploaded files, pages and links are served under the product's domain, so it hosts malware or phishing; link previews fetch internal addresses (see [4.21](../specialties/4.21-outbound-requests-and-server-side-request-forgery.md)) |
| **Limits bound to the real actor** | Limits on abuse-prone actions are tied to values the abuser can change at will (address, session, device ID) instead of accounts, payment methods or verified identities (see [20](20-resource-bounds-and-backpressure.md) ("Storage of limit state")) |
| Response to abuse | There is no way to report abuse, or accounts, content, keys and features cannot be disabled quickly; thresholds can only change with a new release |
