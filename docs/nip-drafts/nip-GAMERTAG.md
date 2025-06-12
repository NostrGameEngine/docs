NIP-GAMERTAG Gaming Identity
==============
`draft` `optional` `author:riccardobl`

This NIP defines a nip-39 external identity that defines a searchable display name that a Nostr user can choose to represent themselves in games.
---------------------------------

A gamertag is a user-friendly handle that combines a chosen name with a short discriminator. 
It’s suitable for display or search, but not as a unique identifier (always use the Nostr public key for security).

A valid gamertag must:

- Have exactly two parts, separated by #:
- Name (3–21 characters)
- Discriminator (4 digits)
- Use only alphanumeric characters or underscores ([A-Za-z0-9_]) in the name.
- Match the discriminator to the first four characters of the user’s Bech32 public key payload (characters 5–8 of the Nostr key).


# `i` tag for gamertags

The gamertag is defined with a nip-39 compliant `i` tag that uses `gamertag` as platform followed by the gamertag itself, and no proof.

For example, a valid gamertag would be represented as:

```
["i", "gamertag:Nostrich#1234"]
```

