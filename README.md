# Gift Card Reseller Stock Returns

What happens when a reseller in a **gift card voucher distribution** network wants to send unsold stock back: the rules for which cards can be returned, how credit flows back up the hierarchy, and how to stop returned cards from being sold twice. From the [MoboGage](https://evdsystem.com/about-mobogage/) educational series for [EVD System](https://evdsystem.com/).

---

## Why returns are harder than sales

A sale moves one card down the chain and one payment up. A return reverses both, but by the time it happens the card may have been partly sold, printed on a receipt, or photographed. A distributor may also have already paid commission to the sub-dealer who now wants their money back.

So a return is really three questions. Is this card still untouched? Who gets credited, and how much? What state does the card go to afterwards?

## Which cards are returnable

A sensible default is that a card is returnable only if all of these hold:

- It was never activated, or it was activated and voided inside the void window.
- It has no redemption or balance inquiry recorded against it.
- It is still allocated to the reseller asking for the return.
- It is inside the return window agreed in the reseller contract.

For digital PIN stock (codes delivered as files or via API), add one more: the PIN was never exposed in an outbound message to an end customer. If the reseller cannot prove that, treat the code as compromised and cancel instead of returning it to pool.

## Credit back up the chain

Take a three-tier chain: operator, distributor, sub-dealer. The sub-dealer returns cards they bought from the distributor.

| Step | Ledger effect |
|------|--------------|
| Sub-dealer return accepted | Credit sub-dealer wallet at purchase price paid |
| Commission clawback | Debit sub-dealer the commission earned on those cards, if commission was paid at sale |
| Distributor stock | Cards move back to distributor allocation |
| Distributor to operator (optional) | Same pattern one level up, if the distributor also returns |

Doing the clawback in the same transaction as the credit avoids the month-end argument where the sub-dealer kept the commission on cards they never sold.

## Worked example (figures are illustrative)

A sub-dealer bought 200 cards of face value 50.00 at 47.50 each, so 9,500.00 total, with commission built into the discount. Two weeks later they return 60 untouched cards. The system checks each serial: 58 are clean, 2 had a balance inquiry from a POS terminal and are rejected. Credit is 58 x 47.50 = 2,755.00. The 58 serials move back to the distributor's allocation with state `IN_STOCK`, and the 2 rejected serials stay with the sub-dealer, flagged for review.

## Card state after return

Physical cards that are intact can go back to stock. Cards whose packaging is opened or scratch panel damaged should go to `DESTROY_PENDING`, then `DESTROYED` with a witness record. Digital codes that cannot be proved unexposed go to `CANCELLED` and are never reissued.

Do not reuse a cancelled serial for a new card. Auditors and fraud teams rely on a serial having one life.

## Common problems

**Returns used as a credit facility.** A reseller buys big before a promotion, then returns most of it. Limit return percentage per period, or charge a restocking fee written into the contract.

**Returns after price change.** Credit at the price actually paid, not the current price list.

**Batch returns with mixed states.** Validate per serial and return a line-level result file, so the reseller can see which cards were accepted.

The return validation checklist is in [docs/return-checklist.md](./docs/return-checklist.md).

---

## FAQ

**Can a reseller return unsold gift cards?**
Usually yes, if the cards were never activated or redeemed, are still allocated to that reseller and are within the contract's return window.

**How is the reseller credited for returned gift cards?**
At the price they paid, minus any commission already paid on those cards, posted in the same transaction.

**Can returned digital gift card codes be resold?**
Only if there is proof the code was never sent to a customer. Otherwise cancel it.

**How does this relate to electronic voucher distribution?**
The same hierarchy, credit and serial-state rules apply to telecom airtime PINs in an EVD network.

---

## Further reading

- [Multi-tier reseller voucher distribution](https://evdsystem.com/evd-software/multi-tier-reseller-voucher-distribution/) on evdsystem.com
- [Voucher management system by MoboGage](https://evdsystem.com/voucher-management-system-mobogage/) on evdsystem.com
- Related repos: [voucher-channel-inventory-allocation](https://github.com/iwtglobal/voucher-channel-inventory-allocation), [reseller-commission-settlement](https://github.com/iwtglobal/reseller-commission-settlement)

## License

MIT. See [LICENSE](./LICENSE). Educational material only.
