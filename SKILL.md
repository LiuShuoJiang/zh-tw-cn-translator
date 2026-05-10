---
name: zh-tw-cn-translator
description: >
  Use when the user asks to translate technical Chinese content between Traditional Chinese (Taiwan) and
  Simplified Chinese (Mainland China), in either direction. Triggers include explicit requests like
  "translate this to simplified", "convert to traditional Chinese", "繁转简", "简转繁", "繁体翻译", "简体翻译",
  "台湾用语转大陆用语", or any phrasing about Chinese variant conversion for technical writing. Also triggers
  when the user has a markdown document, blog post, article, or technical documentation in one Chinese variant
  and wants it in the other (any size, from a single paragraph to 1000+ line article), or when they reference
  terminology gaps between Taiwan and Mainland usage (e.g., 位元 vs 位, 記憶體 vs 内存, 實作 vs 实现, 機率 vs
  概率, 量測 vs 测量). Applies to content in computer science, networking, operating systems, cryptography,
  physics, statistics, mathematics, and other technical domains.
---

# Chinese Traditional-Simplified Technical Document Translator

## Why This Skill Exists

Translating between Traditional Chinese (Taiwan) and Simplified Chinese (Mainland China) is NOT just a character-level conversion. In technical domains (especially computer science, engineering, and IT), Taiwan and Mainland China use fundamentally different terminology for the same concepts. For example:

| English | Taiwan (Traditional) | Mainland (Simplified) |
|---------|---------------------|----------------------|
| bit | 位元 | 位 |
| byte | 位元組 | 字节 |
| memory | 記憶體 | 内存 |
| two's complement | 二補數 | 补码 |
| implementation | 實作 | 实现 |
| operating system | 作業系統 | 操作系统 |

A naive character converter would turn "位元" into "位元" (no change — both are valid simplified characters!) and miss the semantic mapping to "位". This skill handles both layers: character conversion AND terminology mapping.

## Translation Workflow

### Step 1: Assess the Document

Read the source document and determine:
- **Direction**: Traditional → Simplified, or Simplified → Traditional?
- **Size**: How many lines? (determines chunking strategy)
- **Domain**: What technical field? (CS, electronics, networking, etc.)
- **Non-standard markup**: Does it use platform-specific syntax (HackMD `:::success`, `==highlight==`, etc.)?

### Step 2: Load the Terminology Reference

Read the terminology reference table from `references/terminology.md` in this skill's directory. The table contains ~220 entries across 12 domain sections — computer architecture, programming, data structures, networking, OS, number representation, UI/general computing, security/cryptography, probability/statistics, physics/thermodynamics, measurement/engineering, and general academic writing.

When translating Traditional → Simplified, apply the "Taiwan → Mainland" column.
When translating Simplified → Traditional, apply the reverse mapping.

### Step 3: Chunk Strategy (for documents > 150 lines)

Large documents MUST be split into chunks to maintain translation quality and avoid context window pressure. The chunking strategy:

1. **Identify natural section boundaries** using markdown headings (`##`, `###`)
2. **Target chunk size**: 80-150 lines per chunk (adjust based on content density)
3. **Never split mid-paragraph or mid-code-block**
4. **Aim for 8-12 chunks** for a typical 800-1000 line document
5. **Create a temporary `chunks/` directory** to store intermediate results

For documents under 150 lines, translation can be done in a single pass.

### Step 4: Translate Each Chunk

For each chunk, apply these transformations in order:

#### 4a. Terminology Mapping (MOST IMPORTANT)

Apply domain-specific terminology conversions from the reference table. This is the step that distinguishes professional translation from naive conversion. Key categories:

**Computer Architecture**: 位元→位, 位元組→字节, 暫存器→寄存器, 記憶體→内存

**Programming**: 函式→函数, 程式→程序, 變數→变量, 陣列→数组, 指標→指针

**Networking**: 網際網路→互联网, 網路→网络, 協定→协议, 封包→数据包

**OS/Systems**: 作業系統→操作系统, 核心→内核, 執行緒→线程, 行程→进程

**Number Representation**: 一補數→反码, 二補數→补码, 溢位→溢出, 進位(numeral)→进制

**Software Engineering**: 實作→实现, 最佳化→优化, 相容→兼容, 演算法→算法

**Probability & Statistics**: 機率→概率, 變異數→方差, 分佈→分布, 信賴區間→置信区间, 動差→矩

**Physics**: 熱力學→热力学, 相空間→相空间, 微觀→微观, 巨觀/宏觀→宏观

**Measurement & Engineering**: 量測→测量, 效能→性能, 延遲→延迟, 抖動→抖动, 追蹤→跟踪

**Security**: 金鑰→密钥, 憑證→证书, 資安→信息安全

**Academic writing**: 透過→通过, 迴圈→循环

#### 4b. Character Conversion

Apply standard Traditional ↔ Simplified character conversion for all remaining text:

這→这, 個→个, 們→们, 對→对, 關→关, 與→与, 從→从, 發→发, 開→开, 為→为, etc.

#### 4c. Markdown Cleanup (if requested)

If the source uses non-standard markdown (common in HackMD/CodiMD documents):
- `:::success` ... `:::` → blockquote with bold header
- `:::info` ... `:::` → blockquote with bold header
- `==text==` → **text** (bold)
- `^superscript^` → `<sup>superscript</sup>`
- `~subscript~` → `<sub>subscript</sub>`
- Image sizing syntax like `=70%x` → remove
- Emoji shortcodes like `:notes:` → remove or replace

#### 4d. Preservation Rules

- Keep ALL code blocks exactly as-is (do not translate code)
- Keep ALL LaTeX math expressions as-is
- Keep ALL URLs and image references as-is
- Keep English terms, proper nouns, and technical terms in English
- Keep the original document structure (headings, lists, tables)

### Step 5: Use Parallel Agents for Large Documents

When the document has 8+ chunks, dispatch translation agents in parallel (batches of 4-5) for efficiency:

```
For each chunk:
1. Agent reads assigned line range from source
2. Agent applies terminology + character conversion
3. Agent writes result to chunks/chunk_XX.md
```

Provide each agent with:
- The specific line range to translate
- The key terminology table (can be abbreviated to relevant terms)
- Markdown cleanup rules (if applicable)
- Clear instruction: "FAITHFUL translation only — do not add, remove, or modify content"

### Step 6: Combine and Verify

1. **Concatenate** all chunk files in order
2. **Verify structure**: Compare section headers between source and translation (count must match)
3. **Verify completeness**: Compare structural element counts:
   - Code block markers (should be identical)
   - Image references (should be identical)
   - LaTeX lines (should be identical or close)
4. **Fix residual issues**: Search for common missed conversions:
   - Grep for Traditional characters that should have been converted
   - Check for terminology that was missed (e.g., "二进位" should be "二进制")
   - Check for tautologies created by term merging (e.g., "反码也称为「反码」")
5. **Clean up** temporary chunk files

### Step 7: Final Quality Report

Report to the user:
- Source lines vs translated lines (should be close)
- Number of sections (should match exactly)
- Structural elements preserved (code blocks, images, LaTeX)
- Any issues found and fixed during verification

## Important Principles

1. **Faithfulness**: The translation must preserve the original meaning exactly. Do not add explanations, remove content, or rewrite sentences.

2. **Context sensitivity for 進位/进位**: The word 進位 has two meanings:
   - As a numeral system (二進位, 十進位) → 进制 (二进制, 十进制)
   - As arithmetic carry (進位) → 进位 (stays the same)
   Always determine from context which meaning applies.

3. **Preserve intentional distinctions**: Some documents deliberately distinguish between related terms. For example, an article might use both 計算機 (abstract computing machine) and 電腦 (physical computer) — preserve this as 计算机 vs 电脑.

4. **When in doubt, be conservative**: If unsure whether a term should be converted, leave it in the original form and flag it for human review.

## Direction: Simplified → Traditional

When translating in the reverse direction (Mainland → Taiwan), apply the terminology table in reverse. Additional considerations:

- 内存 → 記憶體 (not just 記憶体)
- 补码 → 二補數 (not 補碼, unless targeting Hong Kong readers)
- 算法 → 演算法
- 实现 → 實作 (not 實現, which means "to realize" in Taiwan usage)
