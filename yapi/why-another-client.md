# [DRAFT] Why There Are So Many API Clients (and Why I Built Yapi Anyway)

If you search for "Postman alternative," you’ll find dozens of tools that all
claim to fix the same problems. That should have warned me that I was building
something stupid.

However, API clients are not a solved problem. The way we write software keeps
chaging, and with LLMs entering the loop of our workflow, having tools which
synergise the efforts of humans and LLMs is crucial.

This post is an accounting of the current landscape, what each class of tool
does well, where it breaks down, and why I ultimately built **yapi** instead
of adopting yet another alternative.

---

## [Postman](https://www.postman.com/) and [Insomnia](https://insomnia.rest/): Interaction-First
The ol' classic. Everyone's favorite and least favorite at the same time.

**Strengths**

* Good UX for exploration.
* Easy to onboard non-engineers.

**Limitations**

* The GUI is the source of truth.
* State lives in synced workspaces and opaque JSON.
* Automation is bolted on after the fact.

**Why this matters now**
These tools assume a human is always present to click, inspect, and correct.
LLMs cannot do that. They cannot safely read or modify hidden state,
and they cannot reason about a workflow embedded in a GUI.

These tools are great for learning API development in a fun and easy way. But
it's not fast to use for power users.

---

## [curl](https://curl.se/), [HTTPie](https://httpie.io/),
[xh](https://github.com/ducaale/xh): Command-First

**Strengths**

* Ubiquitous and composable.
* Easy to generate.
* Ideal for quick debugging.

**Limitations**

* No structure beyond the shell.
* Chaining becomes fragile piping.
* Assertions, reuse, and state require external glue.

**Why this matters now**
LLMs are good at emitting commands. They are bad at maintaining shell scripts
as systems evolve. Once a workflow spans multiple steps, correctness degrades
quickly.

These tools are excellent primitives. They are not systems.

---

## [Hurl](https://hurl.dev/): Test-First
To be fair, Hurl is really close conceptually to what I wanted.
However, I didn't want a DSL for making requests, I wanted YAML with an LSP so
anyone can onboard super fast and LLMs (and humans) can use the LSP to write
correct code straight away.

**Strengths**

* Declarative, readable, diffable.
* CI-native by design.
* Strong focus on assertions.

**Limitations**

* HTTP-only, no gRPC, TCP, WebSockets.
* 'DS(m)L' weird hand-rolled domain specific markup language for defining
requests. *(Insert @theprimagen pronouncing it 'dismal')*

**Why this matters now**
Hurl is the closest existing tool to treating requests as code.
But modern systems are rarely single-protocol or single-step. Once you need gRPC,
raw sockets, or stateful workflows, you leave its design envelope.

---

## [Bruno](https://www.usebruno.com/)

**Strengths**

* Files are the source of truth.
* Git-friendly and offline.
* A principled response to Postman’s cloud model.
* Excellent ergonomics for manual exploration.

**Limitations**

* Still human-centric.
* Workflows rely on embedded scripting.
* Errors and output are optimized for reading, not parsing.
* Agents can generate files but struggle to repair them.

**Why this matters now**
Bruno proves that “files over GUIs” was the right move. But it still assumes a developer is the one reading failures and deciding what to fix next.

Agents remain second-class users.

---

## TUI Clients ([Posting](https://posting.sh/), [ATAC](https://github.com/Julien-cpsn/ATAC)): Interaction Reimagined

Posting actually serialises requests in yaml very similar to yapi. It wouldn't be too hard to write a Posting adapter for **yapi**.
**yapi**.

**Strengths**

* Fast, keyboard-driven workflows.
* Dramatically better than GUIs.
* Pleasant for humans.

**Limitations**

* The TUI is the product.
* Files support the UI, not the other way around.
* Automation is incidental.

**Why this matters now**
If the interface is the abstraction, automation will always be downstream.

---

## The Gap None of These Tools Close

All of the above tools were designed for a world where:

* humans write the code,
* automation is supplementary,
* and APIs are mostly HTTP.

That world no longer exists.

Today:

* LLMs generate and modify configs.
* Regressions are caught automatically.
* APIs span HTTP, gRPC, and raw protocols.
* Text files are the universal interface.

No existing tool treats **API interaction as a durable, executable artifact owned jointly by humans and agents**.

That is the gap Yapi exists to fill.

---

## Why Yapi Is Different (and Why It Had to Exist)

Yapi makes a different set of tradeoffs, intentionally:

* Files are the source of truth.
* Workflows are declarative, not scripted.
* Assertions are first-class.
* Output is structured and machine-readable.
* Protocols are abstracted, not baked in.

This isn’t about being “better than Postman.”
It’s about optimizing for a different future.

Yapi assumes:

* agents will write and fix tests,
* failures must be self-describing,
* and reproducibility matters more than interactivity.

If you want a better GUI client, Bruno is excellent.
If you want a better curl, HTTPie is great.
If you want a better test runner, Hurl is compelling.

Yapi exists because none of those tools are built for **AI-native ownership**.

That may or may not matter to you yet.
But it will.

