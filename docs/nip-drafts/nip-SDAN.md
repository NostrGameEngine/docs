## NOSTR Simple Decentralized Advertising Network (NOSTR‑SDAN)

`draft` `optional`  `author:riccardobl` 

---

## Rationale

Most digital products rely on data‑driven advertising. NOSTR‑SDAN brings this model to NOSTR apps, enabling decentralized ad auctions and payouts. By standardizing bidding, offers, and payments via NOSTR events, applications can:

* Monetize through ad space auctions
* Distribute ads across multiple offerers
* Maintain transparency and decentralization

---

# High Level Overview

1. **Advertiser** publishes a **Bidding Event** (`kind:30100`) with ad description, bid parameters, and targeting.
2. **Offerers** filter these events, then respond with an **Offer ActionEvent** (`kind:30101`, `type=offer`) containing a payment invoice.
3. **Advertiser** reviews offers and sends **Accept** or **Reject** **ActionEvents** (`type=accept_offer` or `type=reject_offer`).
4. Upon acceptance, **Offerer** displays the ad; once the user action completes, sends a **Payment Request** (`type=payment_request`).
5. **Advertiser** pays and emits either a **Payout** (`type=payout`) or **Payout Error** (`type=payout_error`).
6. Parties may impose NIP‑13 PoW challenges (`difficulty` field) to deter cheating.

---

## Nomenclature

* **Advertiser**: Entity promoting a product or service, paying per user action.
* **Offerer**: Entity offering ad space in their application.
* **Bid**: Advertiser’s proposed msat amount per action.
* **Action**: User interaction that triggers payment (e.g., view, click).
* **PoW Difficulty**: Optional proof‑of‑work requirement to penalize misbehavior.

---

## Bidding Event

A replaceable event (`kind:30100`) where an advertiser bids for ad placement.

```json
{
  "kind": 30100,
  "content": json({
    "description": "<product description>", 
    "context": "<product context in natural language for semantic search>",
    "payload": "<link or text for the ad>",
    "size": [<width>, <height>],
    "link": "<link to open when the ad is interacted with>",
    "call_to_action": "<text for the call to action>",
    "bid": "<amount in msats to pay per action>",
    "hold_time": "<time in seconds>",
  }),
  "tags": [
    ["k", "<action type>"],
    ["m", "<mime/type>"],
    ["h", "<tier1 category>"],
    ["H", "<tier2 category>"],
    ["T", "<tier3 category>"],
    ["t", "<tier4 category>"],
    ["l", "<language>"],
    ["p", "<target>", "<target>", ...],
    ["d", "<ad id>"]
    ["f", "<slot bid>"],
    ["s", "<slot size>"],
    ["expiration", "<timestamp>"]
  ]
}
```

### Content Structure

* **description** (`required`): User‑facing product/service description.
* **context** (`optional`): Natural‑language context for optional semantic matching (not displayed).
* **payload** (`required`): Ad payload (text or image URL).
* **size** (`required`): `[width, height]` in pixels.
* **link** (`optional`): URL opened on interaction.
* **call\_to\_action** (`optional`): Button/text prompt (e.g., “Buy now”). The offerer may choose to display this prompt as part of the ad, or omit it entirely. If not set, the offerer may apply a default call-to-action of their own choosing.
* **bid** (`required`): The amount in msats the advertiser is willing to pay for a successful action.
* **hold\_time** (`optional`): Seconds the advertiser will reserve funds after the offer has been accepted, while waiting for the action to be completed.

### Tags

#### Action Type (`k`) `required`

Specifies the user action that triggers payment. Available types:

* `link`: Paid when a user clicks the ad and successfully opens the target URL.
* `view`: Paid when the ad is rendered and visible to the user.
* `conversion`: Paid when a user completes a defined goal (e.g., newsletter signup, purchase) after interacting with the ad.
* `attention`: Paid when the user demonstrates focused engagement (e.g., hovering, dwell time) with the ad content.

#### Mime Type (`m`) `required`

Specifies the content format of the ad payload. Supported types:

* `image/png`: PNG image URL
* `image/jpeg`: JPEG image URL
* `image/gif`: GIF image URL
* `text/plain`: Plain text content
* `text/markdown`: Markdown-formatted text content

#### Categories (`h`,`H`,`T`,`t`) `optional` `recommended`

Organize ads into a hierarchical taxonomy for targeted filtering. Tiers:

* `h`: Tier 1 category
* `H`: Tier 2 category
* `T`: Tier 3 category
* `t`: Tier 4 category

Refer to the [Nostr Content Taxonomy](../nostr-content-taxonomy) [\[csv\]](nostr-content-taxonomy.csv) (adapted from IAB Content Taxonomy) for full list of categories.


#### Language (`l`) `optional`

Specifies the language of the ad content. Offerers may choose to display ads only to users matching this two‑letter ISO 639‑1 code (e.g., `en`, `es`).

#### Targets (`p`) `optional`

Specifies one or more offerer pubkeys that are authorized to offer to this bid. If an offer originates from a pubkey not listed here, the advertiser may automatically reject it.

This can be used to target specific apps.

#### Ad ID (`d`) `required`

* Unique identifier in advertiser’s namespace.

#### Slots `required`  

Bids can fill predefined slots that the offerer can use to filter out ads they cannot or do not wish to display. This filtering occurs at the relay level before more detailed ranking takes place.

**f tag**

Format: `BTC<amount>`
Indicates the minimum millisatoshi (payment) the advertiser is willing to pay per action.
Accepted values are:

  * `BTC10_000`    (10 satoshis)
  * `BTC100_000`   (100 satoshis)
  * `BTC1_000_000`  (1 000 satoshis)
  * `BTC2_000_000`  (2 000 satoshis)
  * `BTC5_000_000`  (5 000 satoshis)
  * `BTC10_000_000` (10 000 satoshis)
  * `BTC50_000_000` (50 000 satoshis)

Offerers can filter out bids below their minimum acceptable satoshi amount.

**s tag**

Format: `<width>x<height>`
Specifies the maximum container size in pixels that can contain the ad
Accepted values are:

- 120x90
- 180x150
- 728x90
- 300x250
- 720x300
- 240x400
- 250x250
- 300x600
- 160x600
- 336x280

The ad does not have to fill the entire container, but it must fit within the specified dimensions. 
This allows offerers to pre-filter bids based on their rough ad space capacity.

#### expiration tag `optional`

* Unix timestamp (seconds) when bid expires.

---

## Cancellation Event

Advertiser cancels a bid by publishing a NOSTR **Deletion** (`kind:5`) referencing the bidding event ID.

---

## Action Events

Replaceable NIP‑44 encrypted events (`kind:30101`) used for offer, acceptance, payment requests, and payouts.

```yaml
{
  "kind": 30101,
  "content": nip44(json({
    "type": "<action type>",
    "message": "<status message>",
    "difficulty": <optional pow difficulty to respond to the action>,
    // ... other fields depending on the action type ...
  })),
  "tags": [
    ["e","<bidding_event_id>"],
    ["p","<counterparty_pubkey>"]
  ]
}
```

### Common Fields

* **type** (`required`): Subtype (`offer`, `accept_offer`, etc.).
* **message** (`required`): Human‑readable status or error.
* **difficulty** (`optional`): A number defining the number of leading zero bits required in the event ID of the response ActionEvent, as defined in NIP‑13. This proof-of-work mechanism is used to penalize a counterparty that fails to follow the protocol rules. See the  [Punishments and Due Diligence](#punishments-and-due-diligence) section for more details.

### Common Tags
* **e** (`required`): The ID of the original bidding event this ActionEvent is related to.
* **p** (`required`): The pubkey of the counterparty (offerer or advertiser) this ActionEvent is directed to.


### Offer ActionEvent (offerer → advertiser)

This Action Event is used by the offerer to propose its ad space for a given bid. The `content` field (encrypted via NIP‑44) must include:

* **type**: `offer`
* **invoice** (`required`): A BOLT11 invoice, BOLT12 offer, or LNURL address for payment.

The tags must include:

* **e**: The ID of the original bidding event this offer is responding to.
* **p**: The pubkey of the advertiser who made the bid.

Upon receiving this event, the advertiser may accept or reject using the corresponding ActionEvents.

### Accept Offer ActionEvent (advertiser → offerer)

This Action Event is used by the advertiser to accept a specific offer. The encrypted `content` field (NIP‑44) must include:

* **type**: `accept_offer`

The tags must include:

* **e**: The ID of the original bidding event this offer is responding to.
* **p**: The pubkey of the offerer who made the offer.

Upon sending this event, the advertiser must reserve (hold) the full `bid` amount from their funds for up to the specified `hold_time`. This reservation guarantees payment to the offerer once the user action completes and the offerer submits a Payment Request.

### Reject Offer ActionEvent (advertiser → offerer)

This Action Event is used by the advertiser to decline a specific offer. The encrypted `content` field (NIP‑44) must include:

* **type**: `reject_offer`
* **message** (`required`): A string explaining the reason for rejection.

The tags must include:

* **e**: The ID of the original bidding event this offer is responding to.
* **p**: The pubkey of the offerer who made the offer.

Advertisers may reject offers for reasons such as:

* **out\_of\_budget**: No remaining budget for ads.
* **payment\_method\_not\_supported**: Unsupported invoice or payment method.
* **invoice\_expired\_or\_invalid**: Invoice does not match the bid or has expired.
* **expired**: The bid or ad has expired and can no longer be accepted.

Implementers can define additional custom reason strings as needed.

### Payment Request ActionEvent (offerer → advertiser)

Upon receiving an **Accept Offer** ActionEvent, the offerer’s application must begin delivering the ad or user interaction flow. Some actions (e.g., `conversion`) may require user input or external processes and could fail or timeout.

When the offerer has reason to believe the requested action has been successfully completed, it MUST publish a **Payment Request** ActionEvent (`type=payment_request`) with the following encrypted NIP‑44 content:

* **type**: `payment_request`
* **message** (`required`): A human‑readable explanation of why the action is considered complete (e.g., “Ad displayed to user”, “User clicked link”, “Purchase confirmed”). This explanation can be evaluated by a human or an automated reviewer to determine whether the action was successfully completed.

The tags must include:

* **e**: The ID of the original bidding event this payment request is related to.
* **p**: The pubkey of the advertiser who made the bid.

This event serves as a trigger for the advertiser to execute the payout.

### Payout ActionEvent (advertiser → offerer)

Triggered when the advertiser processes a **Payment Request**. Upon receiving a `payment_request` ActionEvent, the advertiser may (optionally) verify the requested user action. The advertiser can choose to skip verification and proceed directly to payment if desired.

* If the action is (or is assumed) successful, the advertiser must pay the agreed `bid` amount to the invoice provided in the original **Offer** event, then publish a **Payout** ActionEvent:

  * **type**: `payout`
  * **message** (`required`): A natural-language description of the payment execution (e.g., “Paid to BOLT11 invoice”, “Paid via LNURL address”). This message can be evaluated by a human or automated reviewer.

* If verification fails or payment cannot be executed, the advertiser should instead publish a **Payout Error** ActionEvent.

The tags must include:

* **e**: The ID of the original bidding event this payout is related to.
* **p**: The pubkey of the offerer who made the offer.

### Payout Error (advertiser → offerer)

Sent when the advertiser is unable to complete the payout. The encrypted `content` must include:

* **type**: `payout_error`
* **message** (`required`): An error message explaining why the payout could not be completed. This can be any string, but the following default values are recommended:

  * `no_route`: The advertiser cannot find a payment route to the offerer's node.
  * `invoice_expired_or_invalid`: The invoice provided is expired or invalid.
  * `expired`: The bid time has expired or the ad is no longer valid.
  * `action_incomplete`: The advertiser did not detect successful completion of the user action on their end.


The tags must include:

* **e**: The ID of the original bidding event this payout error is related to.
* **p**: The pubkey of the offerer who made the offer.

Upon receiving a `payout_error`, or if no payout is ever initiated, the offerer may, at their discretion, impose penalties or blacklist the advertiser (see [Punishments and Due Diligence](#punishments-and-due-diligence)).

An advertiser may choose to proceed with payment even after the original bid or ad expiration, ensuring flexibility for late fulfillment.


## Punishments and Due Diligence

### When the app cheats

An application may falsely claim completion of user actions (e.g., reporting an ad view that never occurred). Advertisers should perform due diligence by:

* Restricting bids to trusted offerers via the `p` tag.
* Applying heuristic checks or third‑party audits to detect inconsistencies.

If an advertiser detects cheating, it may:

* Ignore all future offers from the offending pubkey.
* Impose a PoW challenge by raising the `difficulty` in outgoing ActionEvents, requiring the offerer to commit computational work to continue interactions.

### When the advertiser cheats

An advertiser may fail to honor payments, claim `hold_time` has expired prematurely, or otherwise violate the protocol.

If an offerer detects misbehavior, it may:

* Reject bids from that advertiser pubkey permanently.
* Impose a PoW challenge by raising the `difficulty` in outgoing ActionEvents, requiring the advertiser to commit computational work to continue interactions.

---

Implementations should maintain local blacklists of repeat offenders and may share reputational data off‑protocol to enhance ecosystem trustworthiness.

THe Pow penalty PoW deters further misconduct while allowing honest parties to redeem themselves for issues that may have been caused by bugs or misconfigurations.