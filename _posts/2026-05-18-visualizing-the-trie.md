---
layout: post
title: "Visualizing the Trie: insertion and prefix search"
date: 2026-05-18 12:00:00 -0700
---

A companion to [Shipping sm2-flashcards]({% post_url 2026-05-18-shipping-sm2-flashcards %}). The Trie in `app/trie.py` is the project's headline data structure, but how it actually evolves in memory during inserts — and how `search_prefix` walks it — isn't obvious from the code alone.

These diagrams trace the exact sequence of `_TrieNode` allocations and child-pointer updates as three tags get inserted, then visualize the two-phase prefix search.

## Step 1: Initialize the empty Trie

Before any data is inserted, the Trie consists of only a single, blank root node. It has no children, no associated card IDs, and is not a terminal node.

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#e1f5fe', 'edgeLabelBackground':'#ffffff', 'tertiaryColor': '#fff'}}}%%
graph TD
    Root(("Root<br/>(is_terminal: F)"))

    classDef terminal fill:#81d4fa,stroke:#01579b,stroke-width:2px;
    classDef normal fill:#e1f5fe,stroke:#0277bd,stroke-width:1px;

    class Root normal;
```

## Step 2: Insert "py" with Card 10

When `insert("py", 10)` is called, the Trie creates a new path.

1. It creates a node for `'p'`.
2. It creates a node for `'y'`, marks it terminal, and adds Card 10.

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#e1f5fe', 'edgeLabelBackground':'#ffffff', 'tertiaryColor': '#fff'}}}%%
graph TD
    Root(("Root<br/>(is_terminal: F)"))
    P(("p<br/>(is_terminal: F)"))
    Y(("y<br/>(is_terminal: T)<br/>[Cards: 10]"))

    classDef terminal fill:#81d4fa,stroke:#01579b,stroke-width:2px;
    classDef normal fill:#e1f5fe,stroke:#0277bd,stroke-width:1px;

    class Y terminal;
    class Root,P normal;

    Root -- "children['p']" --> P
    P -- "children['y']" --> Y
```

## Step 3: Insert "pax" with Card 20

Next, `insert("pax", 20)`. The Trie follows the existing `'p'` branch and splits to a new `'a'` branch.

1. Follows Root → `p`.
2. Creates a new branch for `'a'`.
3. Creates a node for `'x'`, marks it terminal, and adds Card 20.

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#e1f5fe', 'edgeLabelBackground':'#ffffff', 'tertiaryColor': '#fff'}}}%%
graph TD
    Root(("Root<br/>(is_terminal: F)"))
    P(("p<br/>(is_terminal: F)"))
    Y(("y<br/>(is_terminal: T)<br/>[Cards: 10]"))
    A(("a<br/>(is_terminal: F)"))
    X(("x<br/>(is_terminal: T)<br/>[Cards: 20]"))

    classDef terminal fill:#81d4fa,stroke:#01579b,stroke-width:2px;
    classDef normal fill:#e1f5fe,stroke:#0277bd,stroke-width:1px;

    class Y,X terminal;
    class Root,P,A normal;

    Root -- "children['p']" --> P
    P -- "children['y']" --> Y
    P -- "children['a']" --> A
    A -- "children['x']" --> X
```

## Step 4: Insert "python" with Card 10

Finally, `insert("python", 10)`. This tag reuses the entire existing path of `"py"`.

1. Follows Root → `p` → `y`.
2. Continues the path, creating new nodes for `'t'`, `'h'`, `'o'`, `'n'`.
3. Marks `'n'` terminal and adds Card 10.

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#e1f5fe', 'edgeLabelBackground':'#ffffff', 'tertiaryColor': '#fff'}}}%%
graph TD
    Root(("Root<br/>(is_terminal: F)"))
    P(("p<br/>(is_terminal: F)"))
    A(("a<br/>(is_terminal: F)"))
    X(("x<br/>(is_terminal: T)<br/>[Cards: 20]"))

    Y(("y<br/>(is_terminal: T)<br/>[Cards: 10]"))
    T(("t<br/>(is_terminal: F)"))
    H(("h<br/>(is_terminal: F)"))
    O(("o<br/>(is_terminal: F)"))
    N(("n<br/>(is_terminal: T)<br/>[Cards: 10]"))

    classDef terminal fill:#81d4fa,stroke:#01579b,stroke-width:2px;
    classDef normal fill:#e1f5fe,stroke:#0277bd,stroke-width:1px;

    class Y,X,N terminal;
    class Root,P,A,T,H,O normal;

    Root -- "children['p']" --> P
    P -- "children['a']" --> A
    A -- "children['x']" --> X

    P -- "children['y']" --> Y
    Y -- "children['t']" --> T
    T -- "children['h']" --> H
    H -- "children['o']" --> O
    O -- "children['n']" --> N
```

## Step 5: Visualizing `search_prefix("py")`

This is how autocomplete executes on the final structure above.

The search happens in two phases, highlighted in different colors:

1. **Prefix lookup (`_find`)** — travels down the orange arrows to find the node representing `"py"`. The `'a'` branch (and the entire `"pax"` subtree) is never touched.
2. **Collection walk (`_walk`)** — starting from the `y` node, DFS collects every terminal node in that subtree (the blue path). Returns both `"py"` and `"python"`.

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#e1f5fe', 'edgeLabelBackground':'#ffffff', 'tertiaryColor': '#fff'}}}%%
graph TD
    Root(("Root"))
    P(("p"))
    A(("a<br/>(Ignored)"))
    X(("x<br/>(Ignored)"))
    Y(("y<br/>[MATCH #1: py]"))
    T(("t"))
    H(("h"))
    O(("o"))
    N(("n<br/>[MATCH #2: python]"))

    Root -- "Lookup 'p'" --> P
    P -- "Lookup 'y'" --> Y
    P -. "Ignored Branch" .-> A
    A --> X
    Y -. "DFS Walk" .-> T
    T --> H
    H --> O
    O --> N

    linkStyle 0 stroke:#ff9100,stroke-width:4px
    linkStyle 1 stroke:#ff9100,stroke-width:4px
    linkStyle 4 stroke:#2979ff,stroke-width:3px,stroke-dasharray: 5 5
    linkStyle 5 stroke:#2979ff,stroke-width:3px,stroke-dasharray: 5 5
    linkStyle 6 stroke:#2979ff,stroke-width:3px,stroke-dasharray: 5 5
    linkStyle 7 stroke:#2979ff,stroke-width:3px,stroke-dasharray: 5 5

    classDef searchPath fill:#ffe0b2,stroke:#ff9100,stroke-width:3px
    classDef matchNode fill:#b3e5fc,stroke:#01579b,stroke-width:4px
    classDef normal fill:#e1f5fe,stroke:#0277bd,stroke-width:1px

    class Root,A,X,T,H,O normal
    class P searchPath
    class Y,N matchNode
```

## Two takeaways

1. **Shared prefixes share memory.** Inserting `"python"` reuses the entire `p → y` path from `"py"`. That's why the trie is `O(total characters across all keys)`, not `O(num_keys × avg_key_length)`. Insert a thousand variants of `"graph-*"` and you only pay once for the `"graph-"` prefix.
2. **Search is two phases, not one.** The prefix lookup is a tight `O(k)` walk down the trie. The collection phase is `O(r)` where `r` is the number of matching nodes underneath. Branches that don't share the prefix are pruned entirely from the work — you never even look at them. That's the asymptotic difference vs. a naive `for word in vocab: if word.startswith(prefix)` scan.

This is what `tests/bench_trie.py` measures in the project — Trie prefix search vs. the naive scan on a 5,000-tag corpus. The Trie wins by an order of magnitude on this size and the gap widens as the corpus grows.

<!-- Mermaid renderer: convert fenced ```mermaid blocks into <div class="mermaid"> and initialize. -->
<script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
<script>
  document.querySelectorAll('pre > code.language-mermaid').forEach(function (el) {
    var div = document.createElement('div');
    div.className = 'mermaid';
    div.textContent = el.textContent;
    el.parentElement.replaceWith(div);
  });
  mermaid.initialize({ startOnLoad: true, theme: 'base' });
</script>
