# luau-mc — Luau for Minecraft

[Luau](https://luau.org) is a [simple](https://luau.org/syntax), [performant](https://luau.org/performance), [expressive](https://luau.org/typecheck) language perfect for modding.
Today, Luau powers millions of games on ROBLOX, many of which push the boundaries of what is often described as a very limited platform.
Amongst the world of video game modding, Minecraft may very well be the hardest to get into. For a platform as successful as Minecraft — one whom is still yet to realize much of its potential — this **must** change.

Unfortunately, `luau-mc` is [currently] nothing more than a plaything of documentation for non-existant/theoretical Luau APIs.
Today, `luau-mc` is not a team, but is instead just the [pipe] dream of a single person.
To let you down even further, I am the furthest thing from the type of Java developer needed to make this dream a reality.
Try as I might, I simply do not [yet] have the time nor resources to work towards bringing this project to fruition. This is to say: without your support, `luau-mc` will not exist.

I've made this public with the hope that it may somehow inspire those capable of making the necessary developments to make this project possible, and to help those in the modding/plugin community see just how much better the development experience could be.

# How can I help?

A **LOT** of work is needed to make `luau-mc` possible:
- The [luau-java](https://github.com/hollow-cube/luau-java) project *does* exist, but uses Java's FFM API, which is only available from Java 22 onward
- [Paper](https://github.com/PaperMC/Paper) currently revolves around Java 21, making it outright **incompatible** with `luau-java`
- [Fabric]() too revolves around Java 21 *but* users can elect to run Minecraft on newer versions of Java, meaning mods using `luau-java` *should* be possible today

☕ **If you're a Java developer:**
- Help `Paper` bring support to Java 25 **(necessity)**
- Help `Fabric` bring support to Java 25 (encourages more users to update to an luau-java compatible version)
- Contribute to `luau-java `to maintain support for Java 25, or establish a fork compatible with older versions of Java [to bring `luau-mc` to older versions of Minecraft]
- Suggest features for `luau-mc`'s Luau-facing `java` library

🐳 **If you're an Luau developer:**
- Suggest features and changes you'd like to see in the user-facing Luau APIs

‼️ **Other ways to help:**
- Bring [luau-lsp](https://github.com/JohnnyMorganz/luau-lsp) support to more IDEs
- Help fill out our luau-lsp type definition files
- Bring attention to the cause

# Disclaimers
`luau-mc` is **not** in any way affiliated with Microsoft, Mojang, ROBLOX, the team behind Luau, or any other user projects mentioned here.
