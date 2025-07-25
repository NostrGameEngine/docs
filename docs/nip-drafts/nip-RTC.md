NIP-RTC WebRTC
==============
`draft` `optional` `author:jacany` `author:riccardobl`
based on https://github.com/nostr-protocol/nips/blob/ead1cd6ca6b5b789d70e0d146d17266a2e8e2fba/100.md

This NIP defines how to do WebRTC communication over nostr.
---------------------------------

## Defining Rooms
Rooms are essentially the space that you will be connected to.
Rooms are created by generating a random secret key from wich the derived public key is used as the room id.
Event an one to one connection requires a room.
To comunicate with someone in the room the events must use a `r` tag with the room id and a `p` tag with the public key of the person you are trying to connect to.
Some events will require only the `r` tag and as such are broadcasted to the entire room, others require the `p` tag and are sent to a specific person in the room.

Clients MUST listen for both the `r` and `p` tag containing their pubkey so they know that event is intended for them, for the offer, answer and candidate events, and only to the `r` tag for connect and disconnect events.

## Encryption
The `content` field is encrypted with `NIP-44`, twice, first with the conversation key derives by sender priv + receiver pub and then with the conversation key derived by room priv + receiver pub.

Pseudo code:

```javascript
function encrypt(localPrivKey, content, recipientPubkey, roomSecretKey){
    const serialized = JSON.stringify(content);
    let encrypted = nip44Encrypt(serialized, nip44ConversationKey(localPrivKey, recipientPubkey));
    encrypted = nip44Encrypt(encrypted, nip44ConversationKey(roomSecretKey, recipientPubkey));
    return encrypted;
}

function decrypt(localPrivKey, content, senderPubkey, roomPublicKey){
    content = nip44Decrypt(content, nip44ConversationKey(localPrivKey, roomPublicKey));
    content = nip44Decrypt(content, nip44ConversationKey(localPrivKey, senderPubkey));
    return JSON.parse(decrypted);
}
```

### Broadcasting that you are present
Announces that the client is ready to connect to others.
The connection ID is inferred from the provided pubkey.
```json
{
    "kind": 25050,
    "pubkey": "<sender pubkey>",
    "tags": [
        ["t", "connect"],
        ["r", "room id"]
        ["expiration", "<expiration>"]
    ]
}
```

### Closing
This is used when disconnecting from everybody else. Ex: when browser tab is closed.
```json
{
    "kind": 25050,
    "pubkey": "<sender pubkey>",
    "tags": [
        ["type", "disconnect"],
        ["r", "room id"]
    ]
}
```

### Offering to connect
Used when responding to a `type:connect`. Offering to connect to that peer.
```json
{
    "kind": 25050,
    "pubkey": "<sender pubkey>",
    "tags": [
        ["type", "offer"],
        ["p", "<reciever>"],
        ["r", "<room id>"]
    ],
    "content": encrypt({
        "offer": "<offer>",
        "turn": ["protocol://turnserver:port", "protocol://turnserver2:port"],
        ...misc
    })
}
```

### Answering an Offer
```json
{
    "kind": 25050,
    "pubkey": "<sender pubkey>",
    "tags": [
        ["type", "answer"],
        ["p", "<reciever>"],
        ["r", "<room id>"]
    ],
    "content":  encrypt({
        "sdp": "<sdp>",
        "turn": ["protocol://turnserver:port", "protocol://turnserver2:port"],
        ...misc
    })
}
```

### Broadcasting ICE Candidate
```json
{
    "kind": 25050,
    "pubkey": "<sender pubkey>",
    "tags": [
        ["type", "candidate"],
        ["p", "<reciever>"],
        ["r", "<room id>"]
    ],
    "content": encrypt({
        "candidates": ["<sdp>"],
        ...misc
    })
}
```