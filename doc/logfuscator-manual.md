# LOGFUSCATE(1) User Commands

## NAME

logfuscate - replace personal identifiable data in text files using bidirectional translations

## SYNOPSIS

**logfuscate** [*OPTIONS*] *FILE*...

## DESCRIPTION

**logfuscate** is a command-line tool designed for anonymizing source code and text files by replacing company information, developer usernames, emails, and other identifiable data with similar-looking placeholders. The tool uses bidirectional translation dictionaries, allowing files to be anonymized and later restored to their original content using the same dictionary.

The tool is particularly oriented towards source code anonymization, supporting various identifier conventions including camelCase, snake_case, kebab-case, and space-separated words.

## OPTIONS

### Basic Options

**-h**, **--help**
: Display help information and exit.

**-v**, **--version**
: Display version information and exit.

**-V**, **--verbose**
: Enable verbose output. Shows dictionaries being loaded and lists all replacements made in the format: `filename:line:column: "original" -> "replacement"`.

### Operation Modes

**-r**, **--reverse**
: Reverse mode. Restore files to their original content by swapping the find and replacement values in the dictionary. All other behaviors (priority, terminal, mode flags) remain the same.

**-n**, **--dry-run**
: Preview mode. List the translations that would be applied without modifying any files. Output format: `filename:line:column: "original" -> "replacement"`. Each occurrence is listed separately.

### Output Options

**-i**, **--in-place**
: Modify input files in place. Cannot be combined with **--output**.

**-o**, **--output** *FILE*
: Write output to the specified file. Cannot be used with multiple input files or with **--in-place**.

**-d**, **--output-dir** *DIR*
: Write output files to the specified directory, preserving filenames. Creates the directory if it doesn't exist. When used with **--recursive**, recreates the full directory structure.

Default behavior (no output option): Write concatenated output to standard output.

### Dictionary Options

**--dict** *FILE*
: Specify an additional dictionary file. Can be used multiple times. Merges with default dictionaries unless **--no-defaults** is specified.

**--no-defaults**
: Skip the default dictionary search path. Only use dictionaries specified with **--dict**. Error if **--dict** is not provided.

### Recursive Processing Options

**-R**, **--recursive**
: Process directories recursively. Input arguments can be directories, and all files within will be processed.

**-a**, **--all**
: Include hidden files (files starting with `.`). Default is to skip hidden files.

**--exclude** *PATTERN*
: Exclude files matching the glob pattern. Can be specified multiple times. If the pattern ends with `/`, it applies to directory names; otherwise, it applies to filenames.

**-L**, **--follow-symlinks**
: Follow symbolic links when processing recursively. Default is to skip symbolic links.

### Error Handling

**--abort-on-error**
: Abort immediately if any file cannot be processed. Default behavior is to skip failed files, display an error message, and continue with remaining files.

## DICTIONARY SEARCH PATH

Unless **--no-defaults** is specified, **logfuscate** searches for dictionary files in the following order (higher priority first):

1. Dictionary files specified with **--dict** (in order specified)
2. `./logfuscate.conf` (current working directory)
3. `~/.config/logfuscate/logfuscate.conf` (user config directory)
4. `~/logfuscate.conf` (user home directory)

When multiple dictionaries are found, they are merged together. If the same original value appears in multiple dictionaries with different replacements, the higher priority dictionary wins.

## DICTIONARY FILE FORMAT

Dictionary files are JSON documents with the following structure:

```json
{
  "version": "1.0",
  "description": "Optional description of this dictionary",
  "translations": [
    ...translation entries...
  ]
}
```

### Metadata Fields

**version** (required)
: Format version string. Currently "1.0". The tool will attempt to process unknown versions but will show a warning.

**description** (optional)
: Free-form text description of the dictionary's purpose.

**translations** (required)
: Array of translation entries (see below).

### Translation Entry Formats

Translation entries can use either a shorthand or full format.

#### Shorthand Format

For entries using all default values:

```json
{"original text": "replacement text"}
```

#### Full Format

```json
{
  "find": "original text",
  "repl": "replacement text",
  "mode": "cnksi",
  "pri": 100,
  "term": true
}
```

### Translation Entry Fields

**find** (required)
: The text to find and replace. In shorthand format, this is the object key.

**repl** (required)
: The replacement text. In shorthand format, this is the object value.

**mode** (optional)
: String of character flags controlling matching behavior. See MODE FLAGS section below.

**pri** (optional)
: Numeric priority value. Higher numbers are applied first. Entries without priority have equal default priority and are applied in array order.

**term** (optional)
: Boolean. If true, stops further replacements at this specific match location (but continues processing the rest of the file). Default is false (allows chaining replacements).

### Mode Flags

The **mode** field controls how text matching is performed. It is a string of character flags:

**c**
: camelCase aware. Matches camelCase variations like `AcmeCorp`, `acmeCorp`.

**n**
: snake_case aware. Matches snake_case variations like `acme_corp`.

**k**
: kebab-case aware. Matches kebab-case variations like `acme-corp`.

**s**
: Space-separated aware. Matches space-separated variations like `acme corp`.

**i**
: Case insensitive. Matches any case variation.

**L**
: Literal. Exact match only (case sensitive). Can also be specified as empty string `""`.

#### Mode Behavior

**If mode is omitted:**
: Default depends on the format of the **find** text:
  - If **find** contains underscores, hyphens, or uses camelCase: default is `"L"` (literal)
  - If **find** contains only space-separated words: default is `"cnksi"` (fully flexible)

**If mode is specified:**
: Start with literal matching as the base, then enable specific flexibilities based on the flags provided.

**Escaped spaces:**
: Use backslash to specify a literal space: `"acme\ corp"`. When the **find** text contains escaped spaces:
  - Default mode is `"L"` (literal)
  - Only `i` (case insensitive) and `c` (camelCase) flags make sense
  - With `c` flag: only the first letter of each word can vary in case; remaining letters must match

#### Style Preservation

When replacements are made, the tool preserves the style (casing and separator) found in the original text:

- `AcmeCorp` → `CompanyX`
- `acme_corp` → `company_x`
- `acme-corp` → `company-x`
- `Acme Corp` → `Company X`

### Example Dictionary

```json
{
  "version": "1.0",
  "description": "Anonymization for Acme Corp source code",
  "translations": [
    {
      "find": "Acme Corp",
      "repl": "CompanyX",
      "pri": 100,
      "term": true
    },
    {"SECRET_API_KEY": "CONFIG_VALUE"},
    {
      "find": "john.doe@acme.com",
      "repl": "dev1@example.com",
      "mode": "L"
    },
    {
      "find": "internal server",
      "repl": "external host",
      "mode": "i"
    }
  ]
}
```

## INPUT

Input files are specified as command-line arguments. Special handling:

**-**
: Read from standard input. Can be mixed with regular files and is processed in the order specified.

**Multiple files**
: When multiple files are specified, they are processed in order. Output behavior depends on flags:
  - No output flags: concatenated to stdout
  - **--in-place**: each file modified in place
  - **--output-dir**: each file written to output directory with same name
  - **--output**: error (cannot specify single output file with multiple inputs)

**Directories** (with **--recursive**)
: Process all files within directories recursively, respecting **--exclude**, **--all**, and **--follow-symlinks** flags.

## PROCESSING BEHAVIOR

### Translation Application Order

1. Dictionaries are loaded and merged according to priority
2. Translations are sorted by priority (higher first), then by array order
3. For each location in the input text, translations are applied in order
4. If a translation has `term: true`, no further translations are applied to that specific match location
5. If a replacement creates text matching another entry, that entry is applied (chaining) unless stopped by a terminal flag

### Reverse Mode

When **--reverse** is specified:

- The **find** and **repl** values are swapped
- Priority and terminal semantics remain unchanged
- Mode flags work the same way
- This allows restoring the original content from anonymized files

### File Handling

- **Character encoding**: UTF-8 is assumed for all files
- **File permissions**: Preserved when creating output files
- **Directory structure**: Preserved when using **--output-dir** with **--recursive**
- **Symlinks**: Skipped by default unless **--follow-symlinks** is specified

## EXIT STATUS

**0**
: Success

**Non-zero**
: Error occurred. Exit codes follow standard Unix conventions for different error types.

## EXAMPLES

### Basic anonymization

```bash
logfuscate source.py
```

Anonymizes `source.py` and writes to stdout using default dictionaries.

### In-place modification

```bash
logfuscate --in-place config.py utils.py
```

Anonymizes both files in place.

### Using custom dictionary

```bash
logfuscate --dict custom.conf --output-dir output/ src/
```

Recursively processes `src/` directory, merges custom dictionary with defaults, and writes to `output/` directory.

### Reverse operation

```bash
logfuscate --reverse --in-place anonymized.py
```

Restores `anonymized.py` to its original content.

### Preview changes

```bash
logfuscate --dry-run --verbose project/
```

Shows which dictionaries would be loaded and which replacements would be made without modifying files.

### Process with stdin

```bash
cat log.txt | logfuscate - > anonymized.log
```

Anonymizes content from stdin.

### Complex recursive operation

```bash
logfuscate --recursive --all --exclude "*.log" --exclude "test_*" \
  --exclude "build/" --follow-symlinks --output-dir anonymized/ src/
```

Recursively processes `src/`, including hidden files, excluding log files and test files and build directory, following symlinks, and writing to `anonymized/` directory.

### Use only custom dictionary

```bash
logfuscate --no-defaults --dict project.conf --in-place file.txt
```

Anonymizes using only the specified dictionary, ignoring defaults.

## NOTES

- The tool processes text files and preserves line endings
- Binary files are not supported and will cause errors
- Very large files are processed line by line to manage memory usage
- The tool is designed for source code but can be used with any text files
- Dictionary files are parsed once at startup; syntax errors cause immediate abort

## BUGS

Report bugs to: <project-issues-url>

## AUTHOR

Written as specified by user requirements.

## COPYRIGHT

Copyright information here.

## SEE ALSO

**sed**(1), **awk**(1), **grep**(1)
