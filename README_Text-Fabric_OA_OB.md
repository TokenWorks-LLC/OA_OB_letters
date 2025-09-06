# Old Assyrian / Old Babylonian with Text‑Fabric — Colab Notebook (README)

**Audience:** Assyriology PhD students (new to programming).  
**Notebook:** `Text_Fabric_OA_OB_Colab.ipynb` (Colab‑ready) / `Text_Fabric_OA_OB_notebook.ipynb` (local Jupyter).

## Purpose
This notebook introduces the **Text‑Fabric (TF)** data model and APIs for exploring Old Assyrian (OA) and Old Babylonian (OB) corpora. You will learn how to load a TF “app,” inspect node *types* and *features*, traverse text structure (documents → lines → words), run searches, and export results (e.g., node IDs and line text) to CSV for downstream analysis.

## Learning goals
- Understand TF’s core objects **A, F, L, T, TF** and the idea of annotated text as a **graph** (nodes + features).
- Navigate hierarchy: e.g., `L.u(node, otype="line")` (up to line), `L.d(line, otype="w")` (down to words).
- Interpret node **types** used in OA/OB: `doc`/`text` (document level), `line`, `w` (word), and `wdiv` (word divider).
- Use **features** such as `docnumber`, `pnumber`, `lnno`, `ARK`, and render text with `T.text(...)` / display with `A.plain(...)`.
- Export structured data to CSV (e.g., one ID per line, including document and line references).

## Quick start
### 1) Environment
- **Colab**: open the Colab notebook and run the first cell to install packages (internet required).  
- **Local Jupyter** (Python ≥ 3.10):  
  ```bash
  pip install text-fabric
  ```

### 2) Load an app
```python
from tf.app import use
A = use("oldassyrian", hoist=globals())  # or "oldbabylonian"
```
`hoist=globals()` gives you the standard handles: **A** (app), **F** (features), **L** (locality/up/down), **T** (text), **TF** (metadata).

### 3) Inspect corpus structure
```python
F.otype.all             # available node types
list(TF.features)[:10]  # some feature names
```

### 4) Basic traversal & export (example: wdiv → line)
```python
import csv

ids = sorted(F.type.s("wdiv"))          # all word-divider nodes
rows = []
for n in ids:
    ln = L.u(n, otype="line")[0]        # parent line
    doc = (L.u(ln, otype="doc") or L.u(ln, otype="text") or [None])[0]
    rows.append({
        "wdiv_node_id": n,
        "line_node_id": ln,
        "docnumber": F.docnumber.v(doc) if doc else None,
        "lnno": F.lnno.v(ln),
        "ARK": F.ARK.v(doc) if doc else None,
    })

with open("wdiv_index.csv", "w", newline="", encoding="utf-8") as f:
    w = csv.DictWriter(f, fieldnames=rows[0].keys())
    w.writeheader(); w.writerows(rows)
```

## Notes for newcomers to Python
- Run a cell with **Shift+Enter**. If variables “disappear,” **restart** the kernel and run cells from the top.
- Many TF queries return **sets**; sort them for reproducibility: `sorted(...)`.
- CSV writing requires `import csv`. Pandas alternative:  
  ```python
  import pandas as pd
  pd.DataFrame(rows).to_csv("wdiv_index.csv", index=False)
  ```

## Common pitfalls & quick fixes
- **`NameError: name 'csv' is not defined`** → add `import csv` (or use Pandas).
- **`%load_ext autoreload` fails on Python 3.12** (missing `imp`) → either upgrade IPython (>= 8.25) or inject a one‑cell shim:
  ```python
  import types, importlib, sys
  imp = types.ModuleType("imp"); imp.reload = importlib.reload; sys.modules["imp"] = imp
  %load_ext autoreload
  %autoreload 2
  ```

## Citing & attribution
Please cite **Text‑Fabric** and the **Old Assyrian TF app** when you publish results based on this notebook.

- Dirk Roorda, *Text‑Fabric v8.3.4: Text‑Fabric with a new display algorithm*, Zenodo (2020). DOI: 10.5281/zenodo.3909582.  
- *Nino‑cunei/oldassyrian: First public version*, Zenodo (2020).  
- Text‑Fabric documentation: https://annotation.github.io/text-fabric/tf/  
- Tutorial foundation: **Text‑Fabric Old Assyrian tutorial** (nbviewer link in the notebook comments).

**Additional attribution**: This notebook was adapted and extended by the UC Berkeley team — **Micaela Montes** and **Adam Anderson** — building on the Text‑Fabric OA/OB environment by Dirk Roorda and collaborators.

## Acknowledgments
We gratefully acknowledge the TF ecosystem and the OA/OB corpora providers that made this computational exploration possible. This teaching material is intended as an on‑ramp for Assyriologists to engage directly with annotated cuneiform corpora in Python.

## License
This repository is dedicated to the public domain under
[CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/).

To the extent possible under law, TokenWorks LLC has waived all copyright and
related or neighboring rights to this work. No rights reserved.

