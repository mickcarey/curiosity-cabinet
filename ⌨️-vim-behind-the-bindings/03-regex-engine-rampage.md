# The Regex Engine Rampage 🔥
## Inside Vim's Pattern-Matching Monster

They say regular expressions are write-only languages. Well, Vim took that as a challenge and built not one, but TWO regex engines, then made them race each other for your entertainment!

## The Tale of Two Engines 🎭

### Meet the Competitors

In 2013, Vim did something bonkers: it implemented a **second regex engine** alongside the original. Now every search pattern goes through a beauty pageant:

```
Your Pattern: /\v(foo|bar)+baz/
                │
                ├──> Old Engine (Backtracking NFA)
                │     "I'm reliable!"
                │
                └──> New Engine (Hybrid NFA/DFA)
                      "I'm fast!"

Vim: "Fight! Winner handles this search!"
```

### The Old Guard: Backtracking NFA 🏰

The original engine is a **recursive descent parser** with backtracking:

```c
// Simplified pseudo-code of what's happening
function match_pattern(pattern, text, pos) {
    if (pattern is empty) return SUCCESS;

    if (pattern[0] == '*') {
        // Try matching 0 times
        if (match_pattern(pattern[1:], text, pos)) return SUCCESS;
        // Try matching 1+ times
        if (match_char(prev_pattern, text[pos]) &&
            match_pattern(pattern, text, pos+1)) return SUCCESS;
    }
    // ... more pattern types ...

    return FAILURE;  // Backtrack!
}
```

**Pros**: Handles ALL the weird Vim regex features
**Cons**: Can explode exponentially (hello, catastrophic backtracking!)

### The New Hotness: The Hybrid DFA 🚀

The new engine converts patterns to a **deterministic finite automaton** when possible:

```
Pattern: /hello/

State Machine:
START --h--> S1 --e--> S2 --l--> S3 --l--> S4 --o--> ACCEPT
  └─[other]──┴─[other]─┴─[other]─┴─[other]─┴─[other]─┘
                            │
                            v
                          REJECT
```

But here's the twist: when it encounters Vim-specific features (like `\%23l` for line 23), it seamlessly **falls back to NFA mode**!

## The Pattern Compiler Wizardry 🧙‍♂️

### Stage 1: The Tokenizer

Your innocent pattern `/\v(\w+)@<!foo/` first gets tokenized:

```
Tokens:
1. VERY_MAGIC flag
2. GROUP_OPEN
3. WORD_CHAR with PLUS quantifier
4. GROUP_CLOSE
5. NEGATIVE_LOOKBEHIND operator
6. LITERAL "foo"
```

### Stage 2: The AST Builder

Those tokens become an **Abstract Syntax Tree**:

```
        CONCAT
       /      \
   LOOKBEHIND  LITERAL
      │         "foo"
   PLUS(+)
      │
  WORD_CHAR(\w)
```

### Stage 3: The Bytecode Generation

Here's where it gets spicy! The AST compiles to **custom bytecode**:

```
00: LOOKBEHIND_START
01: WORD_CHAR
02: BRANCH_IF_MATCH 01  ; Loop for +
03: LOOKBEHIND_END_NEG
04: LITERAL 'f'
05: LITERAL 'o'
06: LITERAL 'o'
07: MATCH_SUCCESS
```

This bytecode runs on Vim's own **regex virtual machine**! Yes, Vim has a VM just for regex. Your editor is basically running programs within programs. Inception! 🎬

## The Magic Modes Madness 🎩

Vim has FOUR different regex flavors, because why not:

### 1. Very Nomagic (`\V`)
"What you type is what you get" (mostly)

### 2. Nomagic (`\M`)
"Some things are special, but not many"

### 3. Magic (`\m`) - The Default
"Lots of special characters, hope you like backslashes!"

### 4. Very Magic (`\v`)
"Everything is special! Perl programmers, welcome home!"

The engineering challenge? The parser has to handle all four modes **simultaneously** because patterns can switch modes mid-stream:

```
/\v(very magic)\M(suddenly nomagic)\m(back to magic)/
```

The mode switcher is implemented as a **state flag** in the tokenizer that changes the interpretation of subsequent characters. It's like having four different languages in one parser!

## The Optimization Olympics 🏅

### The Boyer-Moore Substring Trick

For simple literal searches, Vim bypasses the regex engine entirely and uses **Boyer-Moore** string searching:

```
Text:    "the quick brown fox jumps"
Pattern: "jumps"

Boyer-Moore builds a "bad character" table:
j: skip 4
u: skip 3
m: skip 2
p: skip 1
s: skip 0
*: skip 5

Search from right to left, skip intelligently!
```

This is why searching for plain text in Vim is lightning fast even in huge files!

### The Fixed-String Optimizer

When Vim detects a pattern like `/foo.*bar.*baz/`, it extracts the fixed strings and searches for them first:

```
Step 1: Find "foo" using Boyer-Moore
Step 2: From there, find "bar"
Step 3: From there, find "baz"
Step 4: Only NOW run the full regex to verify
```

Genius? The **bloom filter** pre-check that eliminates 99% of non-matches before the regex engine even fires up!

### The Memoization Magic

The new engine implements **memoization** for recursive patterns:

```c
typedef struct {
    int position;
    int pattern_offset;
    int result;  // MATCH, NO_MATCH, or UNKNOWN
} memo_entry;

// Before trying to match, check the memo table!
if (memo_table[pos][pattern_offset] != UNKNOWN) {
    return memo_table[pos][pattern_offset];
}
```

This prevents the same sub-pattern from being evaluated multiple times at the same position. It's dynamic programming for text!

## The Vim-Specific Regex Superpowers 🦸

### Positional Assertions

Vim lets you assert positions in ways that would make Perl jealous:

```
\%23l   - Line 23
\%>5c   - After column 5
\%<10v  - Before virtual column 10
\%#     - At cursor position
```

The engine maintains a **position context structure**:

```c
typedef struct {
    linenr_T line;
    colnr_T  col;
    colnr_T  vcol;  // Virtual column (tabs count differently!)
    int      cursor_offset;
} pos_context;
```

Every position assertion updates this context as the pattern matches!

### The Collection Operator

`\%[optional]` matches "o", "op", "opt", "opti", "optio", "option", "optiona", or "optional".

This compiles to a **trie structure** in the bytecode:

```
     o ─> p ─> t ─> i ─> o ─> n ─> a ─> l
     ↓    ↓    ↓    ↓    ↓    ↓    ↓    ↓
   MATCH MATCH MATCH MATCH MATCH MATCH MATCH MATCH
```

The engine traverses this trie, accepting at any node marked as a valid endpoint!

### The Recursive Descent into Madness

Vim supports **recursive patterns** for matching nested structures:

```vim
/\v\{%(\{%(\{%(\{[^}]*\}|[^{}])*\}|[^{}])*\}|[^{}])*\}/
```

This matches nested braces! The engine maintains a **recursion stack** with depth limits to prevent stack overflow. It's essentially running a context-free grammar parser inside a regex engine!

## The Syntax Highlighting Symphony 🎨

### The Incremental Parser

Syntax highlighting doesn't just use regex—it uses **synchronized regex regions**:

```vim
syntax region String start=/"/ end=/"/ skip=/\\"/
```

This creates a state machine:
```
NORMAL --"─-> STRING --"─-> NORMAL
         ↑       │      ↓
         └───\\──┘      └─> (stay in STRING)
```

The clever bit? **Synchronization points** every N lines:

```
Line 100: sync point (full reparse)
Line 101-199: incremental parsing
Line 200: sync point (full reparse)
```

Edit line 150? Vim only reparses from line 100, not line 1!

### The Priority Queue

Multiple syntax patterns can match the same text. Vim uses a **priority queue**:

```c
typedef struct {
    int priority;
    syntax_pattern *pattern;
    int start_col;
    int end_col;
} syntax_match;

// Max-heap of matches, highest priority wins!
```

## The Performance Profiler 📊

Vim includes a **regex profiler** for debugging slow patterns:

```vim
:profile start regex.log
:profile func /.*/
" Do your search
:profile stop
```

It tracks:
- Number of backtrack steps
- Time spent in each engine
- Cache hit/miss ratios
- Memory allocations

## The Future is Parallel? 🔮

Fun fact: There's experimental code in Vim's source for **parallel regex matching** using thread pools:

```c
// From the future (currently ifdef'd out)
typedef struct {
    thread_T    thread_id;
    int         start_pos;
    int         end_pos;
    int         result;
} parallel_match_job;
```

The idea: Split the text into chunks, search in parallel, merge results. The challenge? Patterns that span chunk boundaries!

## Why This Insanity Matters 🌟

This dual-engine, multi-mode, bytecode-compiling, position-aware, recursion-supporting regex monster is what enables:

- Lightning-fast syntax highlighting
- The magic of `:global` commands
- Search and replace across millions of lines
- Plugin ecosystems that can extend the pattern language

Every `/` search you do in Vim triggers this beautiful chaos. It's not just pattern matching—it's a computational orchestra playing at the speed of thought!

And that's why Vim's regex engine isn't just a feature—it's a rampage through the very concept of what text searching can be! 🚀💥