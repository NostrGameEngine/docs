
!!! abstract "See [LobbyManager Javadoc](https://javadoc.ngengine.org/org/ngengine/network/LobbyManager.html)"

Instead of manually sharing private keys, you can let NGE’s [LobbyManager](https://javadoc.ngengine.org/org/ngengine/network/LobbyManager.html) handle key distribution and peer discovery for you. This is the recommended way to start a connection.

First, initialize a `LobbyManager`:

```java
LobbyManager lobbyManager = new LobbyManager(
    localNostrSigner,
    "My Awesome Game Name",
    "v1.0",
    List.of("wss://relay.ngengine.org"),
    null
);
```

* **localNostrSigner**:  The local player’s `NostrSigner` (from [Authentication](../auth.md)).
* **"My Awesome Game Name"**:  Your game’s name.
* **"v1.0"**:  Your game’s version.
* **List.of("wss://relay.ngengine.org")**:  A list of one or more Nostr relays for peer discovery and signaling. At least one relay is required; you can supply additional relays for redundancy.
* **null**:  A `NostrRelay` to use as a TURN server (pass `null` to disable TURN, which is acceptable for most modern unrestricted networks).

Once the `LobbyManager` is created, you can perform the following operations:

## Create a Lobby

```java
String passphrase = "";
Map<String, String> data = Map.of("map", "ancient ruins");
Duration duration = Duration.ofDays(1);
lobbyManager.createLobby(
    passphrase,
    data,
    duration,
    (lobby, error) -> {
        if (error != null) {
            System.err.println("Failed to create lobby: " + error.getMessage());
        } else {
            System.out.println("Lobby created successfully: " + lobby);
        }
    }
);
```

* **passphrase**:  A string to protect a private lobby. Leave empty (`""`) for a public lobby.
* **data**:  A `Map<String, String>` containing custom metadata (e.g., `"map": "ancient ruins"`). You can use these key‐value pairs to filter lobbies later and to store additional public information about the game session.
* **duration**:  A `Duration` after which the lobby will be automatically deleted.
* **Callback**: Receives either a `LocalLobby` object (if successful) or an `Exception` (if failed).

The returned `LocalLobby` object can be updated in (near) real‐time by calling:

!!! abstract "See [LocalLobby Javadoc](https://javadoc.ngengine.org/org/ngengine/network/LocalLobby.html) and [Lobby Javadoc](https://javadoc.ngengine.org/org/ngengine/network/Lobby.html)"

```java
lobby.setData(key, value);
```

This change may take a moment to propagate.



!!! tip
    You can retrieve the lobby’s `sharedPrivateKey` via:

    ```java
    NostrPrivateKey key = lobby.getKey(passphrase);
    ```
    However, as we will see below, you rarely need to do this manually, `LobbyManager` will handle it for you.


!!! warning
    Creating a lobby does **not** automatically connect you to it. To connect, see [Connect to a Lobby](#connect-to-a-lobby) below.

## Find Lobbies

Query the relay(s) for all available lobbies matching your criteria:

```java
String search = "";
int limit = 100;
Map<String, String> filters = Map.of("map", "ancient ruins");
lobbyManager.findLobbies(
    search,
    limit,
    filters,
    (lobbies, error) -> {
        if (error != null) {
            System.err.println("Failed to find lobbies: " + error.getMessage());
        } else {
            System.out.println("Found lobbies: " + lobbies);
        }
    }
);
```

* **search**:  A string to search for in lobby names or metadata. Leave empty (`""`) to match all lobbies. The search is performed on‐relay if the relay supports it; otherwise, it happens client‐side.
* **limit**:  Maximum number of lobbies to return.
* **filters**:  A `Map<String, String>` to filter lobbies by metadata keys/values. 1 letter keys will be filtered on‐relay (per the Nostr protocol); longer keys will be filtered client‐side.
* **Callback**:  Returns a `List<Lobby>` (if successful) or an `Exception` (if failed).

## Connect to a Lobby

After creating or discovering a `Lobby`, connect to it with:

```java
P2PChannel chan = lobbyManager.connectToLobby(lobby, passphrase);
```

* **lobby**:  The `Lobby` or `LocalLobby` object you created or discovered.
* **passphrase**:  The passphrase used when creating the lobby (use `""` if it was public).

This method returns a `P2PChannel` immediately. Internally, it handles key retrieval, signaling, and ICE negotiation. Once the channel is connected, you can send and receive packets as usual.

!!! note "For jME3 Developers"
    From this point onward, the flow is very similar to a SpiderMonkey server


## Listening for Peers

Once you have a `P2PChannel`, you can register listeners to detect incoming connections. 

!!! note
    Peer connections may take a few seconds to complete, due to NAT traversal and signaling delays.


```java
chan.addConnectionListener(new com.jme3.network.ConnectionListener() {
    @Override
    public void connectionAdded(Server server, HostedConnection conn) {
        // A peer has connected. You can now send packets to 'conn'.
    }

    @Override
    public void connectionRemoved(Server server, HostedConnection conn) {
        // A peer has disconnected. Clean up any resources here.
    }
});
```

Optionally, you can listen for peers that have been discovered but have not yet fully connected:

```java
chan.addDiscoveryListener(
    new org.ngengine.nostr4j.rtc.listeners.NostrRTCRoomPeerDiscoveredListener() {
        @Override
        public void onRoomPeerDiscovered(
            NostrPublicKey peerKey,
            NostrRTCAnnounce announce,
            NostrRTCRoomPeerDiscoveredState state
        ) {
            // Handle a newly discovered peer (before ICE is complete)
        }
    }
);
```