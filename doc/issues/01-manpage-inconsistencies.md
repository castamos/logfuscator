# Inconsistencies Found in logfuscator-manual.md

## 1. **CRITICAL: Tool Name Mismatch**
**Location:** Throughout the manual
**Issue:** The repository is named `logfuscator` but the manual consistently refers to the tool as `logfuscate`.
**Recommendation:** Decide on one name and use it consistently. Either rename the repository to `logfuscate` or update the manual to use `logfuscator`.

---

## 2. **Mode Flags - "L" Flag Logic**
**Location:** Lines 175-176, 184-186
**Issue:** The manual states:
- Line 175-176: `"L": Literal. Exact match only (case sensitive). Can also be specified as empty string "".`
- Line 184-186: `"If mode is specified: Start with literal matching as the base, then enable specific flexibilities based on the flags provided."`

**Problem:** If all modes start with literal matching as the base, the "L" flag becomes redundant and confusing. Additionally:
- How does "L" combine with other flags? Is "Li" (literal + case insensitive) contradictory?
- If literal is the base and you add flags to enable flexibility, what does an explicit "L" flag do?
- Can "L" be combined with other flags, or is it mutually exclusive?

**Recommendation:** Clarify whether:
1. "L" means "literal only" (no other flags allowed), OR
2. All flags start literal and "L" is just documentation (shouldn't be used with other flags)

---

## 3. **Mode Flags - camelCase Flag Contradiction**
**Location:** Lines 160, 187-191
**Issue:**
- Line 160 defines `c` as: "camelCase aware. Matches camelCase variations like `AcmeCorp`, `acmeCorp`"
- Lines 190-191 state: "With `c` flag: only the first letter of each word can vary in case; remaining letters must match"

**Problem:** These definitions contradict each other. In the examples `AcmeCorp` vs `acmeCorp`, more than just the first letter varies - the 'c' in Corp also changes from lowercase to uppercase.

**Recommendation:** Clarify the exact behavior of the `c` flag with escaped spaces vs. without escaped spaces, or revise the definition to be consistent.

---

## 4. **Default Priority Value Undefined**
**Location:** Lines 149-150
**Issue:** "Entries without priority have equal default priority and are applied in array order."

**Problem:** The actual numeric value of the "default priority" is not specified. This creates ambiguity:
- If one entry has `pri: 10` and another has no priority, which is applied first?
- Is the default priority 0, or some other value?
- How do explicitly prioritized entries interact with default priority entries?

**Recommendation:** Specify the default priority value (e.g., "Entries without priority default to priority 0").

---

## 5. **Output Options - Incomplete Mutual Exclusivity**
**Location:** Lines 40-49
**Issue:** The manual states:
- `--output` cannot be combined with `--in-place`
- `--output` cannot be used with multiple input files

**Problem:** It doesn't specify:
- Can `--output-dir` be combined with `--in-place`? (Likely no, but not stated)
- What happens if both `--output` and `--output-dir` are specified?
- What happens if all three are specified?

**Recommendation:** Explicitly state all invalid combinations of output options, or add a general rule like "Only one output option may be specified at a time."

---

## 6. **Recursive Processing Without Flag**
**Location:** Lines 61-62
**Issue:** "`-R, --recursive`: Process directories recursively. Input arguments can be directories, and all files within will be processed."

**Problem:** It doesn't specify what happens when:
- A directory is provided as input WITHOUT the `--recursive` flag
- Should it error? Process only direct files (non-recursive)? Skip the directory?

**Recommendation:** Clarify the behavior when directories are provided without `--recursive`.

---

## 7. **Style Preservation - Unclear Mechanism**
**Location:** Lines 194-200
**Issue:** The examples show:
```
- `AcmeCorp` → `CompanyX`
- `acme_corp` → `company_x`
- `acme-corp` → `company-x`
- `Acme Corp` → `Company X`
```

**Problem:** These examples assume a translation like `"Acme Corp" → "CompanyX"`, but:
- It's not clear from the examples alone what the input find/replacement strings are
- The mechanism for converting "CompanyX" to "company_x" (splitting on case changes, converting to snake_case) is not explicitly documented
- How are multi-word replacements like "Company X" handled differently from "CompanyX"?

**Recommendation:** Add a clear explanation of the style conversion algorithm:
1. How the tool detects the style of the matched text
2. How it transforms the replacement text to match that style
3. Whether the replacement text format affects the output (e.g., does "CompanyX" vs "Company X" produce different results?)

---

## 8. **Translation Chaining - Order Ambiguity**
**Location:** Lines 250-256
**Issue:** "If a replacement creates text matching another entry, that entry is applied (chaining) unless stopped by a terminal flag"

**Problem:** The order of chaining operations is ambiguous:
- Does chaining happen immediately at each location before moving to the next location?
- Or are all first-pass replacements done, then another pass is made for chains?
- Can chains create additional chains (recursive chaining)?
- If multiple translations could chain from a single replacement, which one is chosen?

**Recommendation:** Clarify the exact algorithm:
1. Whether chaining is applied immediately or in subsequent passes
2. Whether chaining is recursive (chains can create more chains)
3. How conflicts are resolved when multiple translations could apply to chained text

---

## 9. **Mode Default Logic - Incomplete Specification**
**Location:** Lines 179-183
**Issue:** Default mode rules state:
- Underscores, hyphens, or camelCase → default `"L"` (literal)
- Only space-separated words → default `"cnksi"` (flexible)

**Problem:** Edge cases not covered:
- What if the text contains both spaces AND underscores? (e.g., "acme_corp name")
- What if the text contains numbers? (e.g., "Acme123Corp")
- What if the text is a single word with no separators? (e.g., "acme")
- Empty string?

**Recommendation:** Provide complete logic for all edge cases or a more precise algorithm for determining the default mode.

---

## 10. **Example Dictionary - Implicit Behavior**
**Location:** Lines 204-228
**Issue:** The example shows:
```json
{
  "find": "Acme Corp",
  "repl": "CompanyX",
  "pri": 100,
  "term": true
}
```

**Problem:** Based on the default mode rules, "Acme Corp" (space-separated words) would default to mode `"cnksi"`, meaning:
- It would match `AcmeCorp`, `acme_corp`, `ACME-CORP`, etc.
- This behavior is not explained in the example

**Recommendation:** Add a comment or explanation to the example showing what text patterns this entry would match, or show the explicit mode field to make the behavior clear.

---

## 11. **Escaped Spaces - Mode Flag Restriction**
**Location:** Lines 187-191
**Issue:** "When the **find** text contains escaped spaces: only `i` (case insensitive) and `c` (camelCase) flags make sense"

**Problem:**
- Why don't `n`, `k`, or `s` flags make sense with escaped spaces?
- What happens if you specify those flags anyway? Error? Ignored?
- The restriction is stated but not enforced or explained

**Recommendation:** Either:
1. Explain why other flags don't make sense and what happens if used, OR
2. Allow all flags and document their behavior with escaped spaces

---

## 12. **Binary Files - Error Handling**
**Location:** Line 352
**Issue:** "Binary files are not supported and will cause errors"

**Problem:** When processing recursively, encountering a binary file could halt progress:
- Does this respect `--abort-on-error` behavior?
- Are binary files automatically skipped, or do they cause errors?
- How does the tool detect binary files?

**Recommendation:** Clarify:
1. How binary files are detected
2. Whether they're automatically skipped or cause errors
3. How `--abort-on-error` affects binary file handling

---

## Summary

**Critical Issues (must fix):**
1. Tool name mismatch (logfuscate vs logfuscator)

**High Priority (significant ambiguity):**
2. Mode flag "L" logic contradiction
3. camelCase flag definition contradiction
4. Default priority value undefined
5. Style preservation mechanism undocumented

**Medium Priority (missing specifications):**
6. Output options mutual exclusivity incomplete
7. Recursive processing without flag undefined
8. Translation chaining order ambiguous
9. Mode default logic incomplete

**Low Priority (documentation improvements):**
10. Example dictionary needs clarification
11. Escaped spaces mode flag restrictions unexplained
12. Binary file error handling unclear
