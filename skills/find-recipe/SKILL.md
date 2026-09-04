---
name: find-recipe
description: Find the best local AI inference model and recipe for a specific NVIDIA hardware, operating system, and backend, using catalog.json and recommended_catalog.json (local copies, or fetched fresh from https://raw.githubusercontent.com/NVIDIA/local-ai-inference-recipes/main/catalog.json and https://raw.githubusercontent.com/NVIDIA/local-ai-inference-recipes/main/recommended_catalog.json). Use when the user asks "which model/recipe should I run", "what can I run on my GPU", or names hardware (e.g. a GeForce RTX SKU, DGX Spark, RTX PRO) and wants a runnable recipe.
---

# Find a recipe

Match the user's hardware + OS + backend against `catalog.json` and
`recommended_catalog.json`, and hand back a runnable recipe — not a list to
sift through.

## 1. Locate the two files, refreshing at most once a day

`catalog.json` and `recommended_catalog.json` ship at the bundle root,
next to this skill, but the server copy can move ahead of it over time —
recipes get added, promoted, or withdrawn independently of any given
bundle. The catalog doesn't change more than once a day in practice, so
don't fetch on every invocation — check the local copy's age first:

1. If a local copy exists next to this skill and its last-modified time is
   under 24 hours old, use it as-is — skip fetching.
2. Otherwise (no local copy, local copy is 24+ hours old, or the user
   explicitly asked for the latest/current catalog), fetch fresh from the
   public repository, if network access is available:
   - https://raw.githubusercontent.com/NVIDIA/local-ai-inference-recipes/main/catalog.json
   - https://raw.githubusercontent.com/NVIDIA/local-ai-inference-recipes/main/recommended_catalog.json

   Use whatever web-fetch capability is available (a web-fetch tool, or
   `curl`/`Invoke-WebRequest` if shell access and network use are both
   permitted). If you can write files, overwrite the local copy with what
   you fetched (and its last-modified time updates naturally), so the next
   invocation can skip fetching again for up to another 24 hours.
3. If fetching fails (no network, blocked, request error) but a local copy
   exists next to this skill, fall back to it even if stale, and say
   you're using a possibly-outdated bundled copy.
4. If neither a usable local copy nor a successful fetch is available, say
   so explicitly and stop — do not answer from memory of what the catalog
   might contain.

Both are plain JSON — read them directly. Do not read raw
`models/**/recipe.yaml` source or `catalog/hardware/*.yaml`/`catalog/backends/*.yaml`
for this task: the JSON files are already the fully-resolved, filtered
output (no deprecated, withheld, or ambiguous-default recipes), and every
field needed for matching and launching is already flat on each recipe
object.

## 2. Establish hardware, OS, and backend

Ask only for what's missing; don't ask about anything the user already
stated or that's obvious from context (e.g. a Dockerfile targeting Linux).

- **Hardware** — the user will usually name a product or catalog hardware
  group ("GeForce RTX", "RTX PRO", "my DGX Spark"). `catalog.json`'s
  `hardware[]` array may represent a broader group rather than an exact SKU,
  so match on `display_name` (case-insensitive substring/token match) first,
  then fall back to `id`. Do not infer a catalog group from a specific SKU
  that the JSON does not name (for example, choose neither `geforce-rtx` nor
  `rtx-pro` for an otherwise ambiguous "RTX 5090"); explain the available
  groups and ask the user to choose. If more than one entry plausibly matches
  (e.g. a single-GPU SKU vs. a `kind: "configuration"` multi-GPU grouping of
  it, linked via `base_hardware`), prefer the single-GPU `kind: "sku"` entry
  unless the user mentioned multiple GPUs or a specific count — then match
  the `gpu_count`-suffixed `configuration` entry instead. If nothing
  plausibly matches, say so explicitly and stop — do not guess a substitute
  SKU.
- **OS** — `linux` or `windows`. Infer from context if the user hasn't said
  (their shell, package manager, or explicit statement); ask if genuinely
  ambiguous.
- **Backend** — `llama-cpp` (native, GGUF checkpoints) or `vllm` (containerized,
  OpenAI-compatible). If the user has a preference (e.g. "I want to use
  vLLM"), honor it. If not, prefer whichever backend has a
  `recommended_catalog.json` entry for this hardware+OS; if both do, or
  neither does, ask which they want, unless one is clearly a poor fit for
  their stated OS (e.g. `vllm`'s container recipes are Linux-first —
  flag this if they're on Windows and only vLLM recipes exist).

## 3. Pick the recipe

Check `recommended_catalog.json` first, then fall back to `catalog.json`:

1. **`recommended_catalog.json`** — look up
   `recommended_recipes[<backend>][<os>]`, and find the entry whose
   `hardware` equals the matched hardware `id`. If found, resolve
   `recipe` (an id) against `models[]` in the same file to get the full
   recipe object. This is the maintainer-curated pick — prefer it over
   anything found in step 2 below, and say so ("this is the recommended
   configuration for your hardware").
2. **`catalog.json`** (no recommended entry for this exact combination) —
   find the model(s) whose `recipes[]` contains an entry matching hardware
   `id` + `platform.os` + `backend`. Every model+hardware+OS+backend
   combination in this file already resolves to at most one
   `status: "optimized"` recipe (ambiguous optimized-vs-optimized
   conflicts are pre-resolved before this file is generated) — so:
   - If exactly one recipe matches, use it.
   - If multiple recipes match (different models, or the same model at
     different precisions/variants), prefer `status: "optimized"` over
     `"experimental"`. Among remaining ties, list the candidates for the
     user (model, precision, variant, `min_vidmem_gb`) rather than
     guessing — precision/variant is a real quality/memory tradeoff the
     user should choose, not an accident of file order.
   - If nothing matches this exact hardware+OS+backend at all, say so
     plainly. Do not substitute a different hardware/OS/backend without
     asking first.

## 4. Hand back a runnable recipe

Once one recipe is chosen, give the user, in order:

1. **What it is**: model display name + `precision` (+ `variant` if set) +
   hardware + OS + backend, and its `status` (call out `"experimental"`
   explicitly — it hasn't been through the same validation as
   `"optimized"`).
2. **Prerequisites** (`prerequisites[]`) and **limitations** (`limitations[]`),
   verbatim — do not paraphrase away a constraint.
3. **Security**, if `security` is set — surface it before any setup or launch
   command. It is often a real operational constraint (e.g. "host networking
   exposes the server; do not run this on an untrusted network").
4. **Setup**, if `runtime.setup_command` is present — run it first, exactly as
   written. This can apply to a container recipe or to generated native
   Windows llama.cpp setup.
5. **Launch**: `serve.command` (the primary command), followed by
   `serve.additional_commands[]` if present (e.g. a multi-node vLLM
   deployment's worker command) — preserve every flag, quote, and line
   continuation exactly as shipped. Never edit, reorder, or "clean up" a
   command.
6. **Alternatives**, if `launch.alternatives[]` is present — for each
   alternative, identify its `id` and give its complete `runtime.setup_command`
   (if present), `serve.command`, and `serve.additional_commands[]` (if
   present). Keep an alternative's setup and commands together; never combine
   commands or flags from the primary launch and an alternative.
7. **Test it**: `clients.shell` and/or `clients.python`, if present.

Never fabricate a field that's `null` or missing (e.g. don't invent a
`backend_version` pin if the recipe has none) — state that it's
unspecified instead.
