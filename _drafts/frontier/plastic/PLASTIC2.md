The GPT-generated code, which no human invested to understand, now exists in the repository with no one aware of it. It is not brought into discussions, people do not talk about it, and if someone has to work on it again in the future, there will be no help available
The plastic code ?



Ah, I see exactly what you’re pointing at. You’re describing a phenomenon in software development that’s often referred to as **“plastic code”** (sometimes also called “ghost code” or “orphaned code” in other contexts). Let me unpack it carefully.

**Plastic code** is code that:

1. **Exists in a repository but is not actively understood** – it was generated (maybe by AI, scaffolding tools, or quick-and-dirty scripting) rather than designed carefully by humans.
2. **Is not discussed or maintained** – nobody reviews it, documents it, or brings it up in team discussions.
3. **Becomes effectively unowned** – if someone needs to modify or debug it in the future, there’s no historical knowledge, no guidance, and no one “feels responsible” for it.

The metaphor “plastic” is used because the code is **malleable yet fragile**: it exists physically in the repo and can be changed, but its purpose and inner workings are soft, undefined, and easily broken. Unlike well-understood code, it doesn’t have a “rigid structure” of shared knowledge supporting it.

**Risks of plastic code**:

* High chance of introducing bugs when someone modifies it.
* Hard to refactor because no one understands the intent.
* Can accumulate, creating a maintenance burden (a kind of “technical debt”) without anyone noticing.

**Example scenario**:

* An AI generates a utility function for data parsing.
* The code works and passes tests.
* Developers commit it, but no one reviews it or documents the logic.
* Months later, a new developer needs to extend that functionality—they’re flying blind.

Some teams mitigate plastic code by:

* Code reviews, even for AI-generated code.
* Documentation of intent and assumptions.
* Periodic audits to remove or refactor unused/orphaned code.

If you want, I can also suggest a **clever metaphorical term for AI-generated plastic code** that captures its ghostly, slippery nature—something teams could actually name in discussion. It’s fun and might stick better than just “plastic code.”

Do you want me to do that?
