# Overlapping Modular Centrality — Caltech Facebook Network

## 1. Project Overview

This notebook replicates the Overlapping Modular Centrality (OMC) framework on the Caltech Facebook network, as described in:

> Ghalmane, Z., Cherifi, C., Cherifi, H. & El Hassouni, M.
> **"Centrality in Complex Networks with Overlapping Community Structure"**
> *Scientific Reports*, 9, 10133 (2019)
> DOI: https://doi.org/10.1038/s41598-019-46507-y

The Caltech network belongs to the **medium community strength** group (μ ≈ 0.27) alongside Princeton (μ ≈ 0.25) and Georgetown (μ ≈ 0.31). Results for this group are reported in Figure 17 of the paper.

The OMC framework decomposes each node's influence into two components:

- **β_L** (Local) — influence within the node's own community via intra-community links
- **β_G** (Global) — influence across communities via inter-community links
- Combined measures (Modulus, Weighted OMC) give the strongest epidemic spreading performance

> **File Format Note:** The Caltech dataset comes as a `.mtx` (Matrix Market) file — different from Netscience (`edges.csv`) and ego-Facebook (`.txt`). The load cell uses `scipy.io.mmread()`. Make sure `scipy` is added to the install cell.

---

## 2. Caltech Facebook Dataset

### 2.1 Description

The Caltech network is a Facebook friendship network among students and faculty at the California Institute of Technology, collected by Traud, Mucha & Porter (2012).

- Nodes represent individual Caltech Facebook users (anonymised integer IDs)
- Edges represent mutual Facebook friendships
- Undirected and unweighted
- Smallest of the seven real-world networks tested in the paper (~620 nodes)
- Only the largest connected component is used

### 2.2 Network Statistics

| Property | Value | Paper Reference |
|---|---|---|
| Nodes (N) | 620 | Table 5 |
| Edges (E) | 7,255 | Table 5 |
| Average degree ⟨k⟩ | 43.31 | Table 5 |
| Max degree k_max | 248 | Table 5 |
| Clustering coefficient C | 0.443 | Table 5 |
| Epidemic threshold λ_th | ~0.012 | Table 5 |
| Mixing parameter μ | ~0.27 | Table 1 |
| Overlapping nodes | ~4.55% | Table 1 |
| Avg memberships m | ~1.066 | Table 1 |
| Community strength | **MEDIUM** | 0.15 < μ < 0.35 |

### 2.3 Download & File Format

**Source:** networkrepository.com
**Download:** https://networkrepository.com/socfb-Caltech36.php

The downloaded zip contains:
- `socfb-Caltech36.mtx` — adjacency matrix in Matrix Market format ← **upload this**
- `readme.html` — dataset description (not needed)

**Matrix Market (.mtx) format:**

```
%%MatrixMarket matrix coordinate pattern symmetric
620 620 7255
2 1
3 1
3 2
...
```

Line 1: format header | Line 2: rows cols nnz | Remaining: row/col pairs

`scipy.io.mmread()` handles parsing automatically.

---

## 3. Code Structure

The notebook `caltech_OMC.ipynb` is structured as follows:

| Section | Description | Key Output |
|---|---|---|
| 1. Install dependencies | `pip install` including scipy | Packages ready |
| 2. Load Network | `scipy.io.mmread()` + COO to NetworkX | Graph G |
| 3. Network Visualisation | Spring layout of 200-node sample | Network plot |
| 4. Community Detection (SLPA) | SLPA with t=100, r=0.1 | `communities_list` |
| 5. Community Visualisation | Colour-coded community plot | Community plot |
| 6. Network Decomposition | Build G_l and G_g | Gl, Gg graphs |
| 7. Compute OMC | Local + global centrality for 4 measures | `omc_all` dict |
| 8. Ranking Measures | Modulus and Weighted OMC, Top-10 table | Ranked nodes |
| 9. SIR Simulation | Epidemic spreading λ=0.1, γ=0.1, 30 runs | `res_*` dicts |
| 10. Δr vs f0 Plot | Relative gain over standard degree centrality | Main result plot |
| 11. Summary Table | Top node per measure, all 4 centralities | Summary table |
| 12. Centrality Distribution | Histogram of local vs global scores | Distribution plot |
| 13. Top-10 Subgraph | Visualise most influential nodes | Subgraph plot |

---

## 4. How to Run

### 4.1 Requirements

- Google Colab (recommended) or Jupyter Notebook with Python 3.8+
- Internet connection for `pip install` in the first cell
- `socfb-Caltech36.mtx` uploaded to the Colab session storage
- `scipy` library (added to install cell — see below)

### 4.2 Step-by-Step

1. Go to https://networkrepository.com/socfb-Caltech36.php and download the zip
2. Extract — you will get `socfb-Caltech36.mtx` and `readme.html`
3. Open `caltech_OMC.ipynb` in Google Colab
4. Upload `socfb-Caltech36.mtx` via the Colab Files panel
5. Run all cells: **Runtime → Run All**

### 4.3 Install Cell (includes scipy)

```python
import subprocess, sys
subprocess.check_call([sys.executable, '-m', 'pip', 'install', '-q',
                       'networkx', 'matplotlib', 'cdlib', 'numpy', 'scipy'])
```

### 4.4 Load Cell (MTX format)

```python
import scipy.io as sio

def load_mtx(filepath):
    """Load .mtx (Matrix Market) file as a NetworkX graph."""
    mat = sio.mmread(filepath)
    mat = mat.tocoo()
    G = nx.Graph()
    for u, v in zip(mat.row, mat.col):
        if u != v:      # skip self-loops
            G.add_edge(int(u), int(v))
    return G

FILENAME = 'socfb-Caltech36.mtx'

G = load_mtx(FILENAME)
G = G.subgraph(max(nx.connected_components(G), key=len)).copy()
print(f'Nodes : {G.number_of_nodes():,}')   # expected: 620
print(f'Edges : {G.number_of_edges():,}')   # expected: 7,255

degrees = [d for _, d in G.degree()]
k1 = np.mean(degrees)
k2 = np.mean([d**2 for d in degrees])
lam_th = k1 / (k2 - k1)
print(f'Epidemic threshold λ_th = {lam_th:.4f}  (paper: ~0.012)')
```

All remaining cells are identical to the other dataset notebooks.

---

## 5. Methods Summary

### 5.1 Community Detection — SLPA

- `t = 100` (iterations)
- `r = 0.1` (memory threshold — labels appearing < 10% of the time are dropped)
- Implementation: `cdlib.algorithms.slpa(G, t=100, r=0.1)`
- Expected result: ~4.55% overlapping nodes, avg membership ~1.066

### 5.2 Network Decomposition

| Network | Description |
|---|---|
| **G_l (Local)** | Intra-community edges only. Overlapping nodes replicated per community. Node labels are `(node_id, community_id)` tuples. |
| **G_g (Global)** | Inter-community edges only. Isolated nodes removed. Nodes absent from G_g receive global score 0. |

### 5.3 OMC Components & Ranking

| Measure | Formula | Uses community topology? |
|---|---|---|
| Local | β_L only | No |
| Global | β_G only | No |
| Modulus | sqrt(β_L² + β_G²) | No |
| Weighted OMC | (1-ρ)(S_My/N)β_L + ρ(α/n_c)β_G | Yes — size, density, neighbours |

### 5.4 SIR Epidemic Model

- Infection rate λ = 0.1 (above epidemic threshold λ_th ≈ 0.012)
- Recovery rate γ = 0.1
- Seeds: top f0 × N nodes ranked by the centrality measure under test
- Averaged over 30 independent SIR runs per (method, f0) combination
- Metric: Δr = (R_c - R_s) / R_s × 100% vs standard degree centrality baseline

---

## 6. Expected Results

Caltech has **medium community structure** (μ ≈ 0.27). From Figure 17:

```
Local measure  → Δr > 0  (~+10% average gain over standard degree)
Global measure → Δr > 0  (~+17% average gain over standard degree)
Global > Local in performance (more inter-community links than strong networks)
Modulus        → Δr > 0  (~+30–35% gain)
Weighted OMC   → Δr > 0  (~+40–45% gain, highest)
```

> **Interpretation:** Caltech has a mix of intra- and inter-community links. Both local hubs and cross-community bridges are important spreaders. This is the key difference from ego-Facebook (strong structure) where the global component performed *worse* than standard degree.

### 6.1 Comparison with Princeton & Georgetown

All three are Facebook university networks from Traud et al. (2012) with medium community structure (Figure 17):

| Network | Nodes | Edges | μ | Overlapping % | Global Betweenness gain |
|---|---|---|---|---|---|
| Princeton | 5,112 | 28,684 | 0.25 | 7.47% | +14% over standard |
| Caltech | 620 | 7,255 | 0.27 | 4.55% | +17% over standard |
| Georgetown | 7,651 | 163,225 | 0.31 | 15.25% | +18% over standard |

As μ increases from Princeton to Georgetown, the global component becomes progressively more important.

---

## 7. File Structure

| File | Description |
|---|---|
| `caltech_OMC.ipynb` | Main notebook — all code |
| `data/caltech/socfb-Caltech36.mtx` | Adjacency matrix in Matrix Market format |

---

## 8. Dependencies

| Package | Version | Purpose |
|---|---|---|
| networkx | >=2.6 | Graph construction, centrality measures, SIR simulation |
| cdlib | >=0.2 | SLPA overlapping community detection |
| numpy | >=1.20 | Numerical averaging and degree calculations |
| matplotlib | >=3.3 | All visualisations |
| scipy | >=1.7 | Loading `.mtx` Matrix Market file (`mmread`) ← **extra vs other notebooks** |

`scipy` is only needed for this notebook (and potentially Princeton/Georgetown if they also use `.mtx` format).
