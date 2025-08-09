## NOSTR Ads Network

`draft` `optional`  `author:riccardobl` 


---

# High Level Overview

1. **Advertiser** publishes a **Bidding** event (`kind:30100`) with ad description, bid parameters, and targeting.
2. **Offerers** filter these events, then respond with an **Offer** negotiation event (`kind:30101`, `type=offer`) containing a payout invoice.
3. **Advertiser** reviews offers and sends an **Accept offer** negotiation event (`kind:30101`, `type=accept_offer`).
4. Upon acceptance, **Offerer** displays the ad; once the user action completes, sends a **Payment Request** negotiation event (`kind:30101`, `type=payment_request`).
5. **Advertiser** pays and emits a **Payout** (`type=payout`), optionally including a refund invoice and the preimage as proof of payment.
6. Either party can **Bail** (`kind:30101`, `type=bail`) at any point with an explanation, optionally providing the preimage of the refund payment.
7. Parties may impose NIP-13 PoW challenges (`difficulty` field) to deter cheating.
8. For conversion tracking, advertisers can include a `$OFFER_ID` placeholder in destination links.


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
    "link": "<link to open when the ad is interacted with>",
    "call_to_action": "<text for the call to action>",
    "bid": <amount in msats to pay per action>,
    "hold_time": <time in seconds>,
    "max_payouts": <maximum number of paid actions>,
    "payout_reset_interval": <interval in seconds to reset the payout count>, 
  }),
  "tags": [
    ["k", "<action type>"],
    ["m", "<mime/type>"],
    ["t", "<category>"],
    // ["t", "<category2>"],
    ["l", "<language>"],
    // ["l", "<language2>"],
    // ["p", "<offerer_pubkey>"],
    // ["p", "<offerer2_pubkey>"],
    // ["p", "<offerer3_pubkey...>"],
    ["y", "<app_pubkey>"],
    // ["y", "<app2_pubkey...>"],
    ["d", "<ad id>"]
    ["f", "<price slot>"],
    ["s", "<size>"],
    ["S", "<aspect ratio>"],
    ["D", "<delegate pubkey>", "<payload>"],
    ["expiration", "<timestamp>"]
  ]
}
```

### Content Structure

* **description** (`required`): User‑facing product/service description.
* **context** (`optional`): Natural‑language context for optional semantic matching (not displayed).
* **payload** (`required`): Ad payload (text or image URL).
* **link** (`required`): URL opened on interaction.
* **call\_to\_action** (`optional`): Button/text prompt (e.g., “Buy now”). The offerer may choose to display this prompt as part of the ad, or omit it entirely. If not set, the offerer may apply a default call-to-action of their own choosing.
* **bid** (`required`): The amount in msats the advertiser is willing to pay for a successful action (number).
* **hold\_time** (`required`): Seconds the advertiser will reserve funds after the offer has been accepted, while waiting for the action to be completed (number).
* **max\_payouts** (`required`):  The maximum number of times an offerer can be paid for a successful action per ad. Payouts pause once this limit is reached and resume after the payout_reset_interval.
* **payout_reset_interval** (`required`):  Time in seconds after which the payouts counter resets, allowing payouts to resume.


### Tags

#### Action Type 
(`k`) `required`

Specifies the user action that triggers payment. Available types:

* `action`: Paid when a user clicks or performs a similar interaction with the ad to open the target URL.
* `view`: Paid when the ad is rendered and visible to the user.
* `attention`: Paid when the user shows focused engagement with the ad (e.g., hover, dwell time, or when the ad enters the viewport).

#### Mime Type 
(`m`) `required`

Specifies the content format of the ad payload. Supported types:

* `image/png`: PNG image URL
* `image/jpeg`: JPEG image URL
* `image/gif`: GIF image URL
* `text/plain`: Plain text content

#### Category 
(`t`) `optional` `recommended`

One or more `t` tags specify the categories of the ad content used for filtering and targeting.
Offerers may choose to display ads only in specific categories. 

Advertisers should select relevant categories from the [Nostr Content Taxonomy](../nostr-content-taxonomy) [\[csv\]](nostr-content-taxonomy.csv) (adapted from IAB Content Taxonomy).


#### Language 
(`l`) `optional`

One or more `l` tags specify the language of the ad content as a two‑letter ISO 639‑1 code (e.g., `en`, `es`).
Offerers may choose to display ads only in specific languages.

#### Offerer Whitelist
(`p`) `optional`

One or more `p` tags specify the pubkeys that are authorized to offer to this bid. 
If an offer originates from a pubkey not listed in a `p` tag, the advertiser may automatically reject it.

The advertiser can omit the `p` tag to allow any pubkey to respond to the bid.


#### App Whitelist
(`y`) `optional`

One or more `y` tags specify the pubkeys of the applications that are authorized to offer to this bid. 
If an offer originates from an app not listed in a `y` tag, the advertiser may automatically reject it.

The advertiser can omit the `y` tag to allow any application to respond to the bid.

#### Ad ID 
(`d`) `required`

* Unique identifier in advertiser’s namespace.

#### Price Slot 
(`f`) `required`

Indicates the minimum amount of millisatoshi the advertiser is willing to pay per action.
Accepted values are:

  * `BTC1_000`    (1 satoshis)
  * `BTC2_000`    (2 satoshis)
  * `BTC10_000`    (10 satoshis)
  * `BTC100_000`   (100 satoshis)
  * `BTC1_000_000`  (1 000 satoshis)
  * `BTC2_000_000`  (2 000 satoshis)
  * `BTC5_000_000`  (5 000 satoshis)
  * `BTC10_000_000` (10 000 satoshis)
  * `BTC50_000_000` (50 000 satoshis)

Offerers should filter out bids below their minimum acceptable satoshi amount.

#### Ad size and Aspect ratio 
(`s`) `required`
(`S`) `required`
 
Specifies the original dimensions of the ad in pixels and its aspect ratio.
This information can be used by offerers to filter out bids for ads that are not intended to be displayed in their available ad space.

Offerers can choose to display the ad anyway and they might perform scaling if needed, but they should use these values as a guide to make sure the ad is properly rendered in the application.

Accepted values are:


|`s` tag | `S` tag | description|
|-------|--------------|------------|
|`480x60`|`8:1`|banner|
|`720x90`|`8:1`|leaderboard|
|-------|--------------|------------|
|`512x128`|`4:1`|horizontal banner|
|`512x256`|`2:1`|horizontal narrow banner|
|-------|--------------|------------|
|`256x512`|`1:2`|vertical banner|
|`128x512`|`1:4`|vertical narrow banner|
|-------|--------------|------------|
|`256x256`|`1:1`|square|



----

To target different ad sizes, advertisers can publish multiple bidding events with different `s` and `S` tags. 
 

#### Delegate  
(`D`) `required`

The `D` is used to define the pubkey that is delegated to manage the ad campaign on behalf of the advertiser. 
This allows advertisers to trust a specialized third-party to stay always online and handle the negotiations and payments for them.

An advertiser may choose to delegate the management of their ads to a delegate they run themselves.

The tag must contain two values:

1. A hex-encoded public key identifying the delegate (i.e., the bot or service that will handle negotiations and payouts).
2. `(optional)` A payload containing additional information needed by the delegate to fulfill its role. This is typically an **encrypted JSON object**,  NIP-44 with the delegate’s public key as the recipient.

A typical decrypted payload might look like:

```yaml
{
  "dailyBudget": 10000, // hint the delegate to limit the total amount of msats spent per day
  "nwc": "nostr+walletconnect://..." // Budgeted NWC URL used to authorize payouts
}
```

If a `D` tag is included, **all negotiation events must be sent to the delegate's pubkey** (from the tag), **not the advertiser’s**. This ensures the delegate handles the lifecycle of offers and payments.

Delegates might automatically listen for biddings that have a matching `D` tag.

> [!IMPORTANT]  
> The dailyBudget should be considered a recommendation to help the delegate pace spending; it’s not a hard cap. To prevent overspend, a strict budget limit should be configured at the NWC wallet level.
 
 
#### Expiration 
(`expiration`) `optional`

Unix timestamp (seconds) when bid expires.

---

## Cancellation Event

Advertiser cancels a bid by publishing a NOSTR **Deletion** (`kind:5`) referencing the bidding event ID.

---

## Negotiation Events

Replaceable NIP‑44 encrypted events (`kind:30101`) used for negotiating offers and payouts. These events are used to communicate actions between the advertiser and offerer, such as making offers, accepting offers, requesting payments, and processing payouts.

```yaml
{
  "kind": 30101,
  "content": nip44(json({
    "type": "<type>",
    "message": "<status message>",
    // ... other fields depending on the type ...
  })),
  "tags": [
    ["d","<event_id>"],
    ["p","<counterparty_pubkey>"],
    ["expiration","<timestamp>"]
  ]
}
```

### Common Fields

* **type** (`required`): Subtype (`offer`, `accept_offer`, etc.).
* **message** (`optional`): Human‑readable status or error.


### Common Tags
* **d** (`required`): The ID of the event this negotiation is related to.
* **p** (`required`): The pubkey of the counterparty (offerer or advertiser) this event is directed to.
* **expiration** (`optional`) Unix timestamp (seconds) when negotiation expires.



### Making an offer 
`(offerer → advertiser)`

This event is used by the offerer to propose its ad space for a given bid. The `content` field (encrypted via NIP‑44) must include:

* **type**: `offer`
* **difficulty** (`optional`): The pow required to accept this offer, as detailed in NIP‑13.  0 means no PoW is required.

The tags must include:

* **d** (`required`): The ID of the original bidding event this offer is responding to.
* **p** (`required`): The pubkey of the advertiser who made the bid.
* **y** (`required`): The pubkey of the application this offer is originated from, payments will be sent to the value in the  `lud16` or `lud06` nip-01 metadata associated with this pubkey


Upon receiving this event, the advertiser may accept or reject the offer using the corresponding event.


### Accepting an Offer 
`(advertiser → offerer)`

This event is used by the advertiser to accept a specific offer. The encrypted `content` field (NIP‑44) must include:

* **type**: `accept_offer`
* **difficulty** (`optional`): The pow required to request a payment for this offer, as detailed in NIP‑13.  0 means no PoW is required.

The tags must include:

* **d**: The event id of the offer to accept
* **p**: The pubkey of the offerer who made the offer.

Upon sending this event, the advertiser must reserve (hold) the full `bid` amount from their funds for up to the specified `hold_time`. This reservation guarantees payment to the offerer once the user action completes and the offerer submits a Payment Request.


### Requesting a Payment 
`(offerer → advertiser)`

Upon receiving an `accept_offer` negotiation event, the offerer’s application must begin delivering the ad or user interaction flow. Some actions (e.g., `conversion`) may require user input or external processes and could fail or timeout.

When the offerer has reason to believe the requested action has been successfully completed, it MUST publish a `payment_request` negotiation event with the following encrypted NIP‑44 content:

* **type**: `payment_request`
* **message** (`required`): A human‑readable explanation of why the action is considered complete (e.g., “Ad displayed to user”, “User clicked link”, “Purchase confirmed”). This explanation can be evaluated by a human or an automated reviewer to determine whether the action was successfully completed.

The tags must include:

* **d**: The event id of the offer 
* **p**: The pubkey of the advertiser who made the bid.

This event serves as a trigger for the advertiser to execute the payout.

### Payout event 
`(advertiser → offerer)`

Triggered when the advertiser processes a **Payment Request**. Upon receiving a `payment_request` event, the advertiser may (optionally) verify the requested user action. The advertiser can choose to skip verification and proceed directly to payment if desired.

If the action is (or is assumed) successful, the advertiser must pay the agreed `bid` amount to the value in the `lud16` or `lud06` nip-01 metadata associated with the pubkey in the `y` tag of the **Offer** event, then publish a **Payout** event with the following encrypted NIP‑44 content:

* **type**: `payout`
* **message** (`required`): A natural-language description of the payment execution (e.g., “Paid to BOLT11 invoice”, “Paid via LNURL address”). This message can be evaluated by a human or automated reviewer.
* **preimage** (`optional`): The preimage of the paid invoice, if applicable. This is used to prove that the payment was executed successfully.


The tags must include:

* **d**: The event id of the offer 
* **p**: The pubkey of the offerer who made the offer.

If verification fails or payment cannot be executed, the advertiser should instead publish a [Bailing Event](#bailing-event) with a reason for rejection.


### Bailing or cancelling an offer 
`(advertiser → offerer || offerer → advertiser)`

Both the advertiser and the offerer can inform the other party that they intend to bail out or reject an offer by a negotiation event with the following encrypted NIP‑44 content:

* **type**: `bail`
* **message** (`required`): The reason for bailing out or cancelling the offer.

The tags must include:

* **d**: The event id of the offer to bail or cancel
* **p**: The pubkey of the counterparty (offerer or advertiser) this event is directed to.

The following default messages are recommended for use when bailing out of an offer:

* `out_of_budget`: No remaining budget for ads.
* `expired`: The bid or hold time has expired and the negotiation is no longer valid or not accepted.
* `failed_payment`: The advertiser can't pay the invoice
* `action_incomplete`: The advertiser did not detect successful completion of the user action on their end.
* `cancelled`: The offer was cancelled.
* `unknown` : unknown reason for bailing out.
 

Implementers can define additional custom reason strings as needed.

When an offer that was originally agreed upon is bailed, the counterparty may choose to impose a punitive measure, such as banning the offending pubkey from future interactions or requiring a proof‑of‑work challenge to continue, see [Punishments and Due Diligence](#punishments-and-due-diligence) for more details.


## Punishments and Due Diligence

### When the app cheats

An application may falsely claim completion of user actions (e.g., reporting an ad view that never occurred). Advertisers should perform due diligence by:

* Restricting bids to trusted offerers via the `p` tag.
* Applying heuristic checks or third‑party audits to detect inconsistencies.

If an advertiser detects cheating, it may:

* Ignore all future offers from the offending pubkey.
* Impose a PoW challenge by raising the `difficulty` in outgoing negotiation events, requiring the offerer to commit computational work to continue interactions.

### When the advertiser cheats

An advertiser may fail to honor payments, claim `hold_time` has expired prematurely, or otherwise violate the protocol.

If an offerer detects misbehavior, it may:

* Reject bids from that advertiser pubkey permanently.
* Impose a PoW challenge by raising the `difficulty` in outgoing negotiation events, requiring the advertiser to commit computational work to continue interactions.

---

Implementations should maintain local blacklists of repeat offenders and may share reputational data off‑protocol to enhance ecosystem trustworthiness.

THe Pow penalty PoW deters further misconduct while allowing honest parties to redeem themselves for issues that may have been caused by bugs or misconfigurations.


## Tracking Actions

To track which offer triggers a specific action, the advertiser can include a `$OFFER_ID` placeholder in the destination link. The offerer must replace this placeholder with the ID of the accepted offer event before displaying the ad. This enables the advertiser to correlate user actions with the specific offer that generated them.

eg. 
  `https://example.com/landing-page?track=$OFFER_ID`