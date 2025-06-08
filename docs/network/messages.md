NGE’s networking layer automatically serializes and deserializes your Java message objects using a secure, built-in protocol. This means you can send and receive fully-typed Java objects between peers with virtually no boilerplate or manual serialization code.

## Listening for Messages

Register a `MessageListener` on your `P2PChannel` to receive incoming `Message` objects:

```java
chan.addMessageListener(new com.jme3.network.MessageListener<HostedConnection>() {
    @Override
    public void messageReceived(HostedConnection source, Message m) {
        // Handle the incoming message 'm' from peer 'source'.
    }
});
```

## Sending a Message

To broadcast a `Message` to all connected peers:

```java
chan.broadcast(message);
```

To send a `Message` to a subset of peers, provide a filter predicate:

```java
chan.broadcast(
    peer -> myPeerList.contains(peer),
    message
);
```

Here, `myPeerList` is a `Collection<HostedConnection>` specifying which peers should receive the message.

### Message Types

NGE includes few built‐in message classes for convenience:

#### TextMessage

!!! abstract "See [TextMessage Javadoc](https://javadoc.ngengine.org/org/ngengine/network/protocol/messages/TextMessage.html)"

Sends a UTF‐8‐encoded `String`. Suitable for chat or lightweight metadata (e.g., JSON payloads). Not recommended for high‐throughput data:

```java
TextMessage msg = new TextMessage("Hello, World!");
chan.broadcast(msg);
```

#### CompressedMessage

!!! abstract "See [CompressedMessage Javadoc](https://javadoc.ngengine.org/org/ngengine/network/protocol/messages/CompressedMessage.html)"

Wraps and compresses another `Message` instance. Ideal for large payloads (images, maps, etc.):

```java
CompressedMessage msg = new CompressedMessage(myUncompressedMessage);
chan.broadcast(msg);
```

#### BinaryMessage

!!! abstract "See [BinaryMessage Javadoc](https://javadoc.ngengine.org/org/ngengine/network/protocol/messages/BinaryMessage.html)"

Sends raw binary data. Use this when you need complete control over serialization.
```java
ByteBuffer buffer = BufferUtils.createByteBuffer(1024);
// Fill 'buffer' with your binary data...
BinaryMessage msg = new BinaryMessage(buffer);
chan.broadcast(msg);
```

!!! tip
    To use a BinaryMessage you must handle (de)serialization manually, as the message only accepts a ByteBuffer.
    So oftentimes using a [Custom Message](#custom-messages) is preferable.


### Custom Messages

For flexibility, you can define your own `Message` classes. Any custom message must:

1. Implement the `com.jme3.network.Message` interface.
2. Have a public no‐argument constructor.
3. Be annotated with `@NetworkSafe` (or `@Serializable`, if you prefer SpiderMonkey’s annotations).
4. Use only safe, serializable fields (see below for details).
5. Have a class name that ends in `Message` (e.g., MyCustom*Message*).


!!! failure "PositionUpdate : wrong"
!!! success "PositionUpdateMessage : correct"

These measures are in place to ensure that only safe and intended classes are sent over the network, preventing potential security issues caused by the deserialization of arbitrary classes and minimizing unintentional mistakes that could lead to such issues.

!!! example
    ```java
    @NetworkSafe
    public class MyMessage implements Message {
        private boolean reliable;
        private int myNumber;

        public MyMessage() {
            // Public no-arg constructor is required.
        }

        public void setNumber(int number) {
            this.myNumber = number;
        }

        public int getNumber() {
            return myNumber;
        }

        @Override
        public Message setReliable(boolean f) {
            this.reliable = f;
            return this;
        }

        @Override
        public boolean isReliable() {
            return reliable;
        }
    }
    ```

Fields in a `Message` can be of another properly declared `Message` type or any of the following serializable types:

```text
Vector.class,
Vector2f.class,
Vector3f.class,
Vector4f.class,
Transform.class,
ColorRGBA.class,
Matrix3f.class,
Matrix4f.class,
Date.class,
Instant.class,
Duration.class,
NostrPublicKey.class,
NostrPrivateKey.class,
NostrKeyPair.class,
UnsignedNostrEvent.class,
SignedNostrEvent.class,
HashMap.class,
WeakHashMap.class,
IdentityHashMap.class,
Hashtable.class,
TreeMap.class,
HashSet.class,
ArrayList.class,
LinkedList.class,
LinkedHashSet.class,
TreeSet.class,
Attributes.class,
Integer.class,
Long.class,
Float.class,
Double.class,
String.class,
Short.class,
Boolean.class,
Byte.class,
Character.class,
int.class,
long.class,
float.class,
double.class,
boolean.class,
byte.class,
char.class,
short.class
```

!!! note
    Collections and Maps are supported, but their elements (for Collections) or keys/values (for Maps) must themselves be one of the supported types listed above. Otherwise, the message will be rejected at serialization time.

 