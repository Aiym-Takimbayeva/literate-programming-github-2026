Prompting OpenAI GPT-6 with `Review spec.md`
produced the following output; strikethrough indicates LLM output that I deemed incorrect or irrelevant.

---

# Review of `spec.md`

The specification defines a Python function, `top_words(text: str) -> list[tuple[str, int]]`, that extracts the 10 most frequent words from English, Russian, and Kazakh article text.

The main implementation challenges are Unicode normalization, precise word tokenization, apostrophe/hyphen handling, and deterministic tie-breaking. The examples provide a useful basis for testing compliance.

## 1. Specification review

| Area             | Requirement                                                   | What to watch for                      |
| ---------------- | ------------------------------------------------------------- | -------------------------------------- |
| Interface        | Python 3, standard library only                               | Keep implementation in `top_words.py`  |
| Input validation | Non-string input raises `TypeError`                           | Validate before processing             |
| Normalization    | Lowercase → NFC → replace `’` with `'`                        | Preserve this exact sequence           |
| Tokenization     | Unicode letters, with internal `'` and `-` allowed            | Avoid ASCII-only regex such as `[a-z]` |
| Counting         | Exact normalized-word frequency                               | No stemming or stop-word removal       |
| Ordering         | Frequency descending, then Unicode code-point order ascending | Do not use locale-based sorting        |
| Limit            | Return at most 10 words                                       | Sort before truncating                 |
| Empty input      | Return `[]` if no words exist                                 | Handle punctuation-only text           |

## 2. Potential ambiguities and edge cases

I don't see a fundamental contradiction in the specification, but a few details deserve particular attention.

### A. Unicode-aware tokenization

The definition of a letter is `str.isalpha()`, not just an English alphabet character. This matters for Kazakh and Russian text and for other Unicode letters.

For example:

- `ҚАЗАҚСТАН` → `қазақстан`
- `кто-то` → one word
- `abc123def` → two words: `abc`, `def`
- `example.com` → two words: `example`, `com`

Recommendation: Implement tokenization using a character-by-character scanner or a carefully designed Unicode-aware tokenizer. A basic ASCII regex would violate the requirements.

### B. Apostrophes and hyphens

Only ASCII `'` and `-` are allowed inside words, and only when each is between letters. The curly apostrophe `’` is converted to `'` during normalization. En/em dashes remain separators.

Be careful with cases such as `a--b`, `-cat-`, `covid-19`, and `'hello'`. They must not produce words containing punctuation at the edges or repeated punctuation.

### C. Normalization order

The specification explicitly requires:

1. `str.lower()`
2. Unicode NFC normalization
3. Replace U+2019 with ASCII apostrophe

Do not substitute NFKC for NFC, and do not fold `ё` into `е`. The normalization must not merge distinct words beyond the transformations specified.

### D. Sorting and truncation

Use the equivalent of:

```
sorted(counts.items(), key=lambda item: (-item[1], item[0]))[:10]
```

Here, `counts` is a dictionary mapping each word to its frequency. This correctly sorts by descending frequency and then ascending Python string order, before taking the first 10 items.

## 3. Suggested implementation strategy

Step 1 — Validate and normalize

Reject non-string inputs with `TypeError`, then apply the three required normalization operations.

Step 2 — Extract words

Scan Unicode characters, collecting letters and allowing an apostrophe or hyphen only when followed by another letter and preceded by a letter.

Step 3 — Count and rank

Count exact occurrences, sort by `(-count, word)`, and return at most 10 tuples.

Step 4 — Verify compliance

Run all 30 supplied examples and the two non-string input checks. Add boundary tests for consecutive punctuation and unusual Unicode characters.

## 4. Testing priorities

The highest-value tests beyond ordinary frequency counting are:

| Test case                       | Expected behavior                                           |
| ------------------------------- | ----------------------------------------------------------- |
| `top_words(None)`               | Raises `TypeError`                                          |
| `top_words(b"cat")`             | Raises `TypeError`                                          |
| `a--b a'-b`                     | Only `a` and `b` are extracted from each malformed sequence |
| `ё Ё е Е`                       | Two occurrences each of `ё` and `е`                         |
| Eleven equally frequent words   | Return the lexicographically first 10                       |
| Empty or punctuation-only input | Return `[]`                                                 |

The malformed-punctuation case above assumes that punctuation cannot connect letters unless it is exactly one permitted internal apostrophe or hyphen.

## Overall assessment

The spec is sufficiently detailed to implement without major clarification. Its strongest feature is the explicit treatment of Unicode, multilingual text, and tie-breaking. The primary implementation risk is incorrectly tokenizing words with a simplistic regular expression.

One additional improvement would be to clarify whether the implementation must support every Unicode character recognized by the running Python version or a fixed Unicode version. Given the current use of Python's `str.isalpha()` and NFC normalization, using the runtime's Unicode behavior is the most natural interpretation.

Would you like me to review the implementation in `top_words.py` against this specification as well?

I can take the review further by checking the actual code or preparing a more rigorous test suite.

