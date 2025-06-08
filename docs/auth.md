User authentication is a core feature of Nostr Game Engine, provided by the [nge-auth](https://javadoc.ngengine.org/org/ngengine/auth/package-summary.html) module. 

It is fully managed by the engine. You only need to provide an [AuthStrategy](https://javadoc.ngengine.org/org/ngengine/auth/AuthStrategy.html) as configuration and then [start the Auth Flow](#starting-the-auth-flow). Once authenticated, the engine gives you a `NostrSigner` instance, from which you can access the user’s public key (that can be used as an unique identifier) and handle signing or encryption.



## Auth Methods

Multiple authentication mechanisms are supported, each of which can be enabled, disabled, or configured based on your application’s requirements. All mechanisms support persistent storage where applicable.

!!! note
    In the Nostr Game Engine, users are uniquely identified by their Nostr public key, with the option to use a non-unique gamertag for display and discovery purposes, see [Player Abstraction](./network/player.md) for more details.


### Local Identity

This is the simplest form of authentication. The user provides a private key, either as a NIP-49-encrypted (`ncryptsec`) or cleartext (`nsec`) string. The key is optionally stored locally encrypted in a data store.

Users can also generate a new private key on the fly using this mechanism, making it suitable for quick "guest account" creation.

### NIP-46 Remote Signer

This method allows users to authenticate using a NIP-46-compatible remote signer. It offers increased security by keeping the private key on a separate, trusted device or app. However, it requires the user to set up and connect a NIP-46 signer application.

### NIP-07 Browser Extension

*Planned*: Support for NIP-07 browser extensions is planned for the webgl porting of the engine.
This will enable authentication through compatible browser-based wallets or extensions.
 

## Auth Flow

The authentication flow is fully handled by NGE, down to the UI and user prompts. This allows you to focus on your game logic without worrying about the underlying authentication mechanics.

### Auth Strategy
!!! abstract "See [AuthStrategy Javadoc](https://javadoc.ngengine.org/org/ngengine/auth/AuthStrategy.html)"

The only thing you need to configure is the authentication strategy. With this simple class you can set which authentication methods you want, their metadata (if applicable), if you want the authentication to be persistent, and more.

!!! note
    The `AuthStrategy` by default has `Local Identity` enabled and automatic persistent storage using the data store obtained from whatever `DataStoreProvier` is available to its consumer


```java
Nip46AppMetadata nip46 = new Nip46AppMetadata().setName("ngengine.org - Unnamed App"); 
AuthStrategy strategy = new AuthStrategy(
    (signer)->{
        // This callback is called when the user successfully authenticates.
        // The signer is important, as it is used to identify the user and interact with the Nostr protocol in many different ways
        // Once you have the signer, you should store it somewhere or pass it to a component.

    }
)
.enableNip46RemoteIdentity(new Nip46AuthStrategy().setMetadata(NIP46_METADATA)) // the nip-46 strategy comes with sane defaults, so we often only need to set the metadata
.enableLocalIdentity();
```

optionally, you can pass also an instance of the `PlayerManagerComponent`, this is not required, but it will allow the auth flow UI to display some better user information, like the avatar and display name, if available.

```java
PlayerManagerComponent playerManager = mng.getComponent(PlayerManagerComponent.class);
strategy = strategy.setPlayerManager(playerManager);
```

### Starting the Auth Flow

Once you have the auth stategy defined, you can start the authentication flow by simply calling:

```java
ngeapp.requestAuth(strategy);
```

if you have access to the `NGEApplication` instance.

Or if you are inside a component, and you prefer not linking to the `NGEApplication` class, you can open the auth window directly:

```java
// get window manager, used to open windows
NWindowManagerComponent windowManager = mng.getComponent(NWindowManagerComponent.class);
windowManager.showWindow(AuthSelectionWindow.class, strategy);
```
