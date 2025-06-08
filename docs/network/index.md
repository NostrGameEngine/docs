Networking is one of the core features of NGE. It is implemented entirely on top of WebRTC and uses Nostr relays for peer discovery and signaling. 

This design means that, given one or more public relays, your application is immediately fully connected, there is no need to set up a dedicated server or subscribe to a third‐party service. You can also allow users to choose their own relays (that could even be self‐hosted). As a result, your game will never stop working due to “master servers going offline” or being discontinued.

!!! note "For jME3 Developers"
    The networking system in NGE is built on top of SpiderMonkey, so most of your SpiderMonkey‐based code should be portable. However, unlike SpiderMonkey, where you have one server instance and multiple client instances, NGE uses one instance of `P2PChannel` (that is a SpiderMonkey server) on each "Client". In other words, every client is technically a "Server" because the protocol is peer‐to‐peer. Additionally, the serialization protocol differs slightly:
    
    - Dynamic class registrations still exist (as in recent SpiderMonkey releases), but registration IDs are connection‐specific. Different peers may register the same packet classes under different IDs, but this should not cause any issues in the common use cases.
    - The NGE serializer uses a growing byte buffer, is stricter about types, and ignores any `@Serializable.serializer` field. In most cases, you can still use SpiderMonkey serializers alongside NGE’s own `DynamicSerializer`, though they may be less efficient.

    If you have performance‐critical code, that needs a custom serializer, consider writing a NGE `DynamicSerializers` for best results.

## Starting a Connection Manually

!!! abstract "See [P2PChannel Javadoc](https://javadoc.ngengine.org/org/ngengine/network/P2PChannel.html)"

Although there is a simpler, automated way to connect to peers (as we will see next in [Lobbies & Discovery section](./lobby)), you can still start a connection manually when you already know a peer’s details:

```java
P2PChannel conn = new P2PChannel(
    localNostrSigner,
    "My Awesome Game Name",
    "v1.0",
    sharedPrivateKey,
    null,
    dispatcher
);
```

* **localNostrSigner**:   The local player’s `NostrSigner`. You can create one yourself or obtain it from the [Authentication](../auth.md) flow.
* **"My Awesome Game Name"**:   The name of your game (choose any string).
* **"v1.0"**:  The version of your game (choose any string).
* **sharedPrivateKey**:  A `NostrPrivateKey` pre‐shared between the peers that want to connect. It is used for both authentication and encryption. You can distribute this key however you like, as long as both parties have it.
* **null**: A relay to use as a TURN server. Passing `null` disables TURN fallback; this is acceptable for most modern, unrestricted networks.
* **dispatcher**: A `Runner` that will dispatch network packets. You control the threading model here; however, to avoid concurrency complexity, you can simply pass the runner provided in the `onAttached()` or `onEnable()` methods of your components. This approach “just works” without worrying about threading issues.

!!! tip 
    You can share the `sharedPrivateKey` with multiple peers, allowing them to connect to each other, creating a multi-peer connection.

!!! note
    Communication between peers is encrypted twice: once with the `sharedPrivateKey` and once with each peer’s public key. This means you can share the same `sharedPrivateKey` among multiple peers without them being able to read each other’s signaling messages.

Once the `P2PChannel` is configured, start the connection with:

```java
conn.start();
```

This call initiates signaling and NAT traversal. 

Depending on network conditions, ICE negotiation may take a few seconds. After the WebRTC connection is established, communication will be low‐latency and reliable.

