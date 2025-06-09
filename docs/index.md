# Nostr Game Engine

Nostr Game Engine (NGEngine or NGE) is a 3D game engine built on top of [jMonkeyEngine 3](https://jmonkeyengine.org/) that extends its capabilities with features designed around the Nostr protocol. 

NGE offers:

- **Reliable P2P Networking** using webRTC on top of Nostr based signaling and peer discovery
- **Decentralized Authentication** with Nostr-based identities
- **Asynchronous Asset Management** for non-blocking loading of models, textures, and other assets
- **Component-Based Architecture** with fragment-based composition for flexible game object design

more to come... [check the roadmap](https://ngengine.org/roadmap.html)

NGE’s main goal is to make Nostr integration seamless, so you can handle things like networking and authentication without digging into the details of the protocol. If you ever need more control, you can still use the underlying [nostr4j](https://github.com/NostrGameEngine/nostr4j)  library directly.

---

## Compatibility with jME3

NGE is organized as a set of modules that layer on top of the standard jMonkeyEngine 3 codebase. 

This ensures:

- **Zero Asset Changes**  All jME3 models, materials, and assets work out of the box, no re-export or reformatting required.
- **Reusable jme3 codebase**    You can take existing jME3 code and integrate it into NGE without modifications.
- **Plugin/Extension Support** Most third-party jME3 plugins and extensions remain compatible.
- **Familiar APIs**  Core classes (e.g., `Node`, `Spatial`, `Material`) and workflows (e.g., scene graph, control attachments) behave just as in jME3 with additional NGE features layered on top.

NGE follows many of the same paradigms as jMonkeyEngine 3 (jME3), so if you're already familiar with jME3, you’ll feel right at home.

!!! info "For jME3 developers"

    You can still use the standard jMonkeyEngine 3 practices in NGE, and you can use NGE modules in existing jMonkeyEngine 3 applications. 

    However, this documentation focus on new practices that improve code portability and reusability. 

!!! info "For jME3 developers"
    Throughout this documentation, there will be notes like this to draw parallel to jME3 practices where relevant.

---


### Module Naming Convention

NGE modules use the following prefixes:

- **jME3 Modules**: All original jMonkeyEngine 3 modules are re-published under the `jme3-` prefix (for example, `jme3-core`, `jme3-lwjgl3`, `jme3-networking`).
- **NGE Modules**: NGE-specific modules are published under the `nge-` prefix (for example, `nge-core`, `nge-auth`, `nge-network`).

!!! Tip "You can mix and match `nge-` modules with any compatible jME3 release. However, for the latest bug fixes and cutting-edge features, we recommend using the latest NGE builds."

 


