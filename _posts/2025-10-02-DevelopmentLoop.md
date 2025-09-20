---
title: "Development Loop: IntelliJ's vs Visual Studio"
date: 2025-10-02
tags:
    - debugging
    - dotnet
    - java
    - ide
    - productivity
    - visual-studio
    - intellij

comments: true

---

This isn’t just a tooling quirk; it actually shaped **how .NET vs Java devs think about their development loop**.

---

# 🔄 Development Loop Mindsets: Java vs .NET

## 🟦 Java / IntelliJ World → **Evaluator/REPL-first Thinking**

* **Loop looks like**:

  1. Run → hit breakpoint
  2. Jump into evaluator window
  3. Try snippets (stream, filter, helper calls)
  4. Inspect results in evaluator
  5. Copy working snippet → paste back into code

* **Mindset formed**:

  * “Code is something I can *play with interactively*, then promote into production.”
  * Encourages **exploration and prototyping in small chunks**.
  * Debugger is a *context provider*; the evaluator is where real iteration happens.

---

## 🟥 .NET / Visual Studio World → **Debugger/Edit-and-Continue-first Thinking**

* **Loop looks like**:

  1. Run → hit breakpoint
  2. Hover/watch variables, navigate deeply with visualizers
  3. (Optional) QuickWatch / Immediate Window for small probes
  4. If you want to try code → just edit the paused code with **Edit & Continue**
  5. Resume execution → see result live

* **Mindset formed**:

  * “The code itself is the playground — I change it in place, not in a scratch REPL.”
  * Encourages **direct iteration inside the source code**.
  * Debugger is the **center of gravity**; evaluator windows are secondary.

---

## ⚖️ Divergence in Dev Culture

* **Java devs**: Think in terms of **scratchpads + promotion** — “let me play in the evaluator, then paste to code.”
* **.NET devs**: Think in terms of **direct code iteration** — “my paused app *is* the scratchpad.”

This is why:

* Java tools invested heavily in *Evaluate Expression* and later **JShell**.
* .NET tools invested heavily in the **world’s richest debugger + Edit & Continue**.

Both loops aim for fast feedback — they just took **different mental shortcuts**.

---

✅ So when you switch contexts:

* In Java, don’t be afraid to **live in evaluator windows & scratch files**. That’s normal.
* In .NET, lean on **Edit & Continue** + deep debugger inspectors.

---

Here’s a **cheat sheet** you can keep handy. It maps the **IntelliJ Evaluator / REPL-style workflow** to the **Visual Studio debugger-first workflow**.

---

# 📝 IntelliJ vs Visual Studio Debugging / REPL Cheat Sheet

| **Goal**                                            | **IntelliJ (Java)**                                                        | **Visual Studio (.NET)**                                                                                               |
| --------------------------------------------------- | -------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| **Inspect variable at breakpoint**                  | Hover → tooltip OR `Alt+F8` → **Evaluate Expression**                      | Hover → **DataTip** OR `QuickWatch` (`Ctrl+Alt+Q`)                                                                     |
| **Run arbitrary code in paused context**            | `Alt+F8` → type Java code (any snippet using current frame vars)           | **Immediate Window** (call methods, run LINQ, new up objects)                                                          |
| **Prototype a new expression / filter / transform** | Use **Evaluate Expression** window (REPL-like), copy/paste into code later | Either: <br>• Immediate Window for quick eval <br>• Or just **Edit & Continue** directly in code, resume execution     |
| **Check a method’s return without writing test**    | In evaluator: `myService.calculateTax(order)` → shows result               | Immediate Window: `myService.CalculateTax(order)` <br> OR just insert in paused method body with Edit & Continue       |
| **Explore collections deeply**                      | Evaluator: run stream/filter/sort, result shows as expandable list         | Watch/Locals/Immediate Window: run LINQ (`Where`, `OrderBy`, etc.), result shows expandable list                       |
| **Try out new helper code**                         | Type snippet into evaluator (multi-line allowed)                           | **C# Interactive Window** (Roslyn REPL) or **LINQPad** outside VS                                                      |
| **Test code outside debugger**                      | JShell (Java REPL) or scratch file                                         | C# Interactive Window OR **LINQPad**                                                                                   |
| **Permanently add to code**                         | Copy from evaluator → paste into editor                                    | Usually typed straight into editor (thanks to Edit & Continue), or paste from Immediate Window if you prototyped there |

---

## 🚩 Big Picture

* **Java dev flow** = “Try it in evaluator → paste to code.”
* **.NET dev flow** = “Patch it directly in paused code → continue debugging.”
* **Bonus tools** =

  * IntelliJ: JShell for REPL outside debugger.
  * .NET: LINQPad (community favorite for quick LINQ/prototyping), C# Interactive.

---

👉 So if you miss the **Evaluator style in .NET**, you’ll feel most at home with **LINQPad** or **C# Interactive Window** — they give you that same scratchpad/repl energy, while Visual Studio’s Immediate Window is more like “peek inside current frame.”

👍 Let’s do a **side-by-side “how I’d test an idea” in IntelliJ vs Visual Studio**.

Imagine we’re in the middle of debugging a web request handler, and we’ve got a `List<Customer>` in memory. We want to try a new filter/sort expression on it before putting that code back into the method.

---

## 🟦 **Java / IntelliJ (REPL-like “Evaluate Expression”)**

1. **Breakpoint** in the handler → execution stops.
2. Right-click → **Evaluate Expression…** (or `Alt+F8`).
3. In the evaluator window:

   ```java
   customers.stream()
       .filter(c -> c.getAge() > 30)
       .sorted(Comparator.comparing(Customer::getName))
       .toList()
   ```
4. Press **Evaluate** → IntelliJ shows you the result: a live list of customers (you can expand the objects inline).
5. If happy → copy/paste that snippet into your actual method.

➡️ That’s the **REPL-like workflow**: *play, evaluate, paste back*.

---

## 🟥 **.NET / Visual Studio (Debugger-centric)**

1. **Breakpoint** in the handler → execution stops.

2. Hover `customers` → you see the live `List<Customer>` in a tooltip, expandable.

3. Right-click → **QuickWatch** (`Ctrl+Alt+Q`) or use the **Immediate Window**.

4. In Immediate Window, type:

   ```csharp
   customers.Where(c => c.Age > 30)
            .OrderBy(c => c.Name)
            .ToList()
   ```

5. VS shows the result object, and you can expand the list inline.

   > ⚠️ Difference: Immediate Window feels clunkier, e.g. you can’t just paste the snippet back into code (the syntax sometimes differs), and you don’t get IntelliSense as rich as the editor.

6. Alternative: **Edit & Continue**. Just temporarily add the LINQ snippet *directly* in the paused method body, hit **Continue**, and see the effect live in debugger.

➡️ So VS assumes: *don’t prototype in a REPL — just change your paused code and rerun it.*

---

## ⚖️ Key contrast

* **IntelliJ**: Evaluator is central — it *is* your REPL.
* **Visual Studio**: Debugger + Edit & Continue are central — “just modify code directly while paused.”
* **Outcome**: Same goal, but Java devs get used to bouncing in/out of evaluator windows, while .NET devs trust the debugger to let them patch-as-they-go.

---

👉 If you want a **true REPL in .NET today**, the closest parallels are:

* **C# Interactive Window** (in VS or VS Code).
* **LINQPad** (hugely popular in .NET land for quick experiments).
* **.NET Interactive Notebooks** (Jupyter-style).

---

You're noticing something pretty fundamental about the **different "debugging cultures"** that grew up around **Java vs .NET** tooling. Let’s unpack why IntelliJ’s *Evaluate Expression / REPL-like window* became central in the Java world, while Visual Studio never made it a priority (despite having arguably the best debugger ever built).

---

### 1. **Debugger-first vs. REPL-first cultures**

* **.NET / Visual Studio**
  From the very start (late 90s, VS + Visual J++ → VS.NET 2002), Microsoft put massive investment into the debugger. Visual Studio's debugger became *the gold standard*:

  * Rich watch windows (with live edit of values)
  * Visualizers (e.g., dataset viewer, XML tree, JSON)
  * DataTips (hover tooltips with full navigation)
  * "Edit and Continue" (change code on the fly in a debug session, re-JIT the method)

  Because of this, .NET devs didn’t *feel* a strong need for an interactive REPL workflow — the debugger *was the playground*. You set a breakpoint, hover, inspect, sometimes even edit variables right there.

* **Java / IntelliJ (and Eclipse, NetBeans before)**
  Java debugging was always weaker compared to VS — no edit-and-continue until much later, less powerful inspectors. To compensate, IDEs like IntelliJ leaned heavily into **Evaluate Expression** as the way to poke at runtime objects, run snippets, and verify code ideas. This *REPL-like* flow became ingrained in Java dev habits.

---

### 2. **Language/runtime differences**

* **C# / CLR**: Designed with edit-and-continue, reflection, and a rich debugger API. "You don’t need a REPL, you have the debugger."
* **Java / JVM**: More rigid runtime, no edit-and-continue until much later, less forgiving JIT. So running snippets in evaluator windows (or external JShell today) filled that gap.

That’s why the **"REPL workflow" became a first-class citizen in Java** land, but not in .NET.

---

### 3. **Immediate Window vs. Interactive Window**

* The **Immediate Window** in VS is really a *debug-time helper* (you can only call methods and evaluate expressions in the current context, not arbitrary compilation units). It’s powerful, but it feels constrained.
* The **C# Interactive Window** (Roslyn-based) *finally gave a REPL experience*, but:

  * It’s separate from the debugger — you don’t get the same "current frame context".
  * Not tightly integrated into daily debugging workflows.

So it always felt like a *bolt-on*, not a core part of the workflow.

---

### 4. **Cultural inertia**

Once a dev ecosystem gets used to one powerful workflow (debugger-first in .NET, REPL/evaluator-first in Java), tooling vendors double down on that path.

* Java devs think: *"Evaluator window is my friend."*
* .NET devs think: *"Debugger + Edit & Continue solves it."*

---

### 5. **Why VS never prioritized REPL deeply**

* Because **debugger + edit-and-continue already covered 80–90% of what REPLs are used for**.
* Because Microsoft focused its "interactive" efforts outside VS: first in **LINQPad**, then **VS Code / .NET Interactive**, and later **Jupyter integrations**.

In short: *they solved the same problem differently* — Visual Studio doubled down on runtime inspection and live editing, IntelliJ on REPL/evaluator workflows.

---

✅ **So, your instinct is right**: REPL workflows are *super productive*, and the Java ecosystem had to invent them because the debugger wasn’t as magical as Visual Studio’s. In .NET, devs just never missed it because the debugger was already *that good*.
