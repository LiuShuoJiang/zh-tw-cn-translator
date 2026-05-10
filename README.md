# zh-tw-cn-translator

A [Claude Code](https://claude.com/claude-code) skill (also usable as a standalone reference) for translating large technical documents between **Traditional Chinese (Taiwan)** and **Simplified Chinese (Mainland China)** — with correct domain-specific terminology mapping, not just character-level conversion.

> **The problem this skill solves:** A naive character converter would turn `位元` into `位元` (both characters are valid in simplified Chinese!) and miss the semantic mapping to `位` — Mainland China's term for "bit". This skill handles **both layers**: character conversion AND terminology mapping.

---

## Why This Exists

Translating between Traditional Chinese (Taiwan) and Simplified Chinese (Mainland China) is **not just a character-level conversion**. In technical domains, Taiwan and Mainland China use fundamentally different terminology for the same concepts:

| English        | Taiwan (Traditional) | Mainland (Simplified) |
| -------------- | -------------------- | --------------------- |
| bit            | 位元                  | 位                    |
| memory         | 記憶體                | 内存                  |
| implementation | 實作                  | 实现                  |
| probability    | 機率                  | 概率                  |
| variance       | 變異數                | 方差                  |
| measurement    | 量測                  | 测量                  |

Off-the-shelf converters (OpenCC, browser extensions) operate at the character level and miss these semantic gaps. This skill provides:

- A curated **~220-entry terminology table** spanning 12 technical domains
- A **workflow specification** for chunked, parallelizable translation of large documents
- **Markdown cleanup rules** for HackMD/CodiMD-flavored source documents
- **Context disambiguation** rules (e.g., 進位 as numeral system vs. as arithmetic carry)

---

## Features

- **Bidirectional**: Traditional → Simplified and Simplified → Traditional
- **Domain coverage**: CS, networking, operating systems, security/crypto, probability/statistics, physics/thermodynamics, measurement/engineering, general academic writing
- **Chunking for large documents**: documents over ~150 lines are split at natural section boundaries (markdown headings) and translated in parallel
- **Structural verification**: validates section count, code blocks, image references, and LaTeX block preservation between source and output
- **Markdown normalization**: converts non-standard syntax like `:::info`, `==highlight==`, `^superscript^`, `~subscript~` to portable markdown
- **Preservation rules**: code blocks, LaTeX math, URLs, and image references are kept verbatim

---

## Installation

### For Claude Code

Clone (or copy) this repo into your Claude Code skills directory:

```bash
git clone https://github.com/<your-username>/zh-tw-cn-translator.git \
    ~/.claude/skills/zh-tw-cn-translator
```

Restart Claude Code (or run `/reload-plugins`). The skill will be auto-discovered and triggered on Chinese-translation requests.

### For Codex (or other agent frameworks)

Place the directory at the appropriate skills location for your platform (e.g., `~/.agents/skills/` for Codex). The `SKILL.md` file is platform-agnostic.

### Standalone use (no agent)

`references/terminology.md` is a self-contained markdown reference table. You can:

1. Open it as a lookup table while doing manual translation, or
2. Feed it to any LLM along with `SKILL.md` as context

---

## Usage in Claude Code

Ask Claude:

```text
Translate /path/to/document.md from Traditional Chinese (Taiwan) to Simplified Chinese.
```

or

```text
Convert this article to traditional Chinese, targeting Taiwan readers.
```

Claude will:

1. Assess document size and domain
2. Load `references/terminology.md` for term mapping
3. Chunk large documents at section boundaries
4. Apply terminology + character conversion in parallel
5. (Optionally) clean up non-standard markdown
6. Verify structural integrity and report

---

## Terminology Coverage

`references/terminology.md` is organized into 12 sections:

| Section                              | Sample entries                                                |
| ------------------------------------ | ------------------------------------------------------------- |
| Computer Architecture & Hardware     | bit, byte, register, memory, firmware                         |
| Programming & Software Engineering   | function, variable, class, polymorphism, dependency           |
| Data Structures & Algorithms         | hash table, linked list, binary tree, quicksort               |
| Networking & Internet                | protocol, server, port, gateway, socket                       |
| Operating Systems                    | kernel, process, thread, daemon, signal                       |
| Number Representation                | two's complement, overflow, bitmask, binary                   |
| User Interface & General Computing   | window, menu, font, cursor, terminal                          |
| Security & Cryptography              | key, certificate, vulnerability, timing attack, side channel  |
| Probability & Statistics             | probability, variance, distribution, confidence interval      |
| Physics & Thermodynamics             | entropy, phase space, dissipation, coarse-graining            |
| Measurement & Engineering            | latency, jitter, throughput, performance, threshold           |
| General Academic Writing             | through/via, loop                                             |

Total: ~220 entries.

---

## Example

**Input** (Traditional Chinese, with HackMD syntax):

````markdown
:::info
在 32 位元系統中，整數使用**二補數**來表示有號數。
迴圈每次執行都會檢查==溢位==的條件。
:::

```c
int x = 0x7FFFFFFF;
```
````

**Output** (Simplified Chinese, normalized markdown):

````markdown
> **提示**
>
> 在 32 位系统中，整数使用**补码**来表示有符号数。
> 循环每次执行都会检查**溢出**的条件。

```c
int x = 0x7FFFFFFF;
```
````

Notice what was changed:

- `位元` → `位` (terminology, not character conversion)
- `二補數` → `补码` (terminology)
- `迴圈` → `循环` (terminology)
- `執行` → `执行` (character conversion)
- `溢位` → `溢出` (terminology)
- `:::info`...`:::` → blockquote with bold header
- `==text==` → `**text**` (bold)
- **Code block preserved verbatim** (no translation inside)

A naive character converter would have produced `在 32 位元系统中，整数使用二补数来表示有号数。迴圈每次执行都会检查溢位的条件。` — leaving the Taiwan terminology untranslated and the HackMD markup intact.

---

## Repository Layout

```
zh-tw-cn-translator/
├── SKILL.md                    # the workflow specification (agent-readable)
├── README.md                   # this file
├── LICENSE                     # MIT
└── references/
    └── terminology.md          # ~220-entry terminology table
```

---

## Contributing

Contributions to the terminology table are very welcome. Please open a PR following this format:

```markdown
| English | Taiwan (Traditional) | Mainland (Simplified) | Notes |
|---------|---------------------|----------------------|-------|
| <term> | <繁體> | <简体> | <context, disambiguation> |
```

**Guidelines:**

- Place the new entry in the **most specific applicable section** (e.g., a statistics term goes under *Probability & Statistics*, not *General Academic Writing*).
- Maintain **alphabetical order by English term** within each section.
- If a term is **context-sensitive** (e.g., 進位 means "numeral system" in some contexts and "arithmetic carry" in others), add a note column and consider documenting it in `SKILL.md`'s "Important Principles" section.
- If the term is **identical in both variants**, still include it with a `Same in both` note — this prevents downstream tools from incorrectly "translating" it.

---

## License

[MIT](LICENSE) — free to use, modify, and redistribute with attribution.

---

## Acknowledgements

- The CS/IT terminology table draws heavily from the [L10N TW Community](https://github.com/l10n-tw) (Taiwan Free/Open Source Software Localization Community).
- Probability, statistics, and physics terminology curated from technical translation projects in those domains.
- Inspired by the limitations of character-level converters like OpenCC when applied to technical writing.
