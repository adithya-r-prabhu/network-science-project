# Overlapping Modular Centrality — Netscience Dataset

## 1. Project Overview

This notebook replicates the Overlapping Modular Centrality (OMC) framework on the Netscience co-authorship network, as described in:

> Ghalmane, Z., Cherifi, C., Cherifi, H. & El Hassouni, M.
> **"Centrality in Complex Networks with Overlapping Community Structure"**
> *Scientific Reports*, 9, 10133 (2019)
> DOI: https://doi.org/10.1038/s41598-019-46507-y

The OMC framework extends classical centrality measures (Degree, Betweenness, Closeness, Eigenvector) to networks with overlapping community structure. Each node's influence is represented as a two-dimensional vector:

- **Local influence** — how influential the node is within its own community
- **Global influence** — how influential the node is across communities via inter-community links

The Netscience notebook replicates the **strong community structure** group (μ ≈ 0.094) from Figure 16 of the paper.

---

## 2. Netscience Dataset

### 2.1 Description

The Netscience network is a co-authorship network of scientists working on network theory and experiment, compiled by M. E. J. Newman (2006).

- Nodes represent individual researchers/authors
- Edges connect authors who have co-authored at least one paper
- Undirected and unweighted
- Only the largest connected component is used

### 2.2 Network Statistics

| Property | Value | Paper Reference |
|---|---|---|
| Nodes (N) | 1,589 | Table 5 |
| Edges (E) | 2,742 | Table 5 |
| Average degree ⟨k⟩ | 3.45 | Table 5 |
| Max degree k_max | 34 | Table 5 |
| Clustering coefficient C | 0.74 | Table 5 |
| Epidemic threshold λ_th | ~0.052 | Table 5 |
| Mixing parameter μ | ~0.094 | Table 1 |
| Overlapping nodes | ~7.84% | Table 1 |
| Avg memberships m | ~2.01 | Table 1 |
| Community strength | **STRONG** | μ < 0.15 |

### 2.3 Download & File Format

**Source:** Netzschleuder network catalogue
**Download:** https://networks.skewed.de/net/netscience

The downloaded zip contains three files:
- `edges.csv` — edge list (source, target per row) ← **only this is needed**
- `nodes.csv` — node metadata
- `gprops.csv` — graph-level properties

> **Important:** Upload only `edges.csv` to Google Colab. The file has a header row — do **not** use `nx.read_edgelist()` directly, it will fail. Use the pandas-based loader below.

---

## 3. Code Structure

The notebook `netscience_OMC.ipynb` is structured as follows:

| Section | Description | Key Output |
|---|---|---|
| 1. Install dependencies | `pip install networkx, cdlib, matplotlib, numpy, pandas` | Packages ready |
| 2. Load dataset | Read `edges.csv`, extract largest component | Graph G |
| 3. Network visualisation | Spring layout plot of 200-node sample | Network plot |
| 4. Community detection (SLPA) | SLPA with t=100, r=0.1 | `communities_list` |
| 5. Community visualisation | Colour-coded community plot | Community plot |
| 6. Network decomposition | Build G_l and G_g | Gl, Gg graphs |
| 7. Compute OMC | Local + global centrality for 4 measures | `omc_all` dict |
| 8. Ranking measures | Modulus and Weighted OMC scores | Top-10 table |
| 9. SIR simulation | Epidemic spreading λ=0.1, γ=0.1, 30 runs | `res_*` dicts |
| 10. Δr vs f0 plot | Relative gain over standard degree centrality | Main result plot |
| 11. Summary table | Top node per measure, all 4 centralities | Summary table |
| 12. Centrality distribution | Histogram of local vs global scores | Distribution plot |
| 13. Top-10 node subgraph | Visualise most influential nodes | Subgraph plot |

---

## 4. How to Run

### 4.1 Requirements

- Google Colab (recommended) or Jupyter Notebook with Python 3.8+
- Internet connection for `pip install` in the first cell
- `edges.csv` uploaded to the Colab session storage

### 4.2 Step-by-Step

1. Download the Netscience dataset zip from Netzschleuder
2. Extract and locate `edges.csv`
3. Open `netscience_OMC.ipynb` in Google Colab
4. Upload `edges.csv` via the Colab Files panel (left sidebar → Files → Upload)
5. Run all cells: **Runtime → Run All**

### 4.3 Load Cell (Netzschleuder CSV format)

```python
import pandas as pd

def load_netzschleuder_edges_csv(filepath):
    df = pd.read_csv(filepath)
    G = nx.Graph()
    for _, row in df.iterrows():
        u = int(row.iloc[0])   # source
        v = int(row.iloc[1])   # target
        G.add_edge(u, v)
    return G

G = load_netzschleuder_edges_csv('edges.csv')
G = G.subgraph(max(nx.connected_components(G), key=len)).copy()
```

> If the file uses a different delimiter or has comment lines, a fallback robust loader is also provided in the notebook that skips any line that cannot be parsed as two integers.

---

## 5. Methods Summary

### 5.1 Community Detection — SLPA

- `t = 100` (number of iterations)
- `r = 0.1` (threshold — labels appearing in < 10% of memory are dropped)
- Implementation: `cdlib.algorithms.slpa(G, t=100, r=0.1)`

### 5.2 Network Decomposition

| Network | Description |
|---|---|
| **G_l (Local)** | Intra-community edges only. Overlapping nodes replicated in each community. Node labels are `(node_id, community_id)` tuples. |
| **G_g (Global)** | Inter-community edges only. Isolated nodes removed. |

### 5.3 OMC Components

For each centrality measure (Degree, Betweenness, Closeness, Eigenvector):

- **β_L** (Local) — computed on G_l, aggregated per node by taking the max across replicas
- **β_G** (Global) — computed on G_g; nodes absent from G_g get score 0
- **B_OM(v)** = (β_L, β_G) — the two-dimensional OMC vector

### 5.4 Ranking Measures

| Measure | Formula | Uses community info? |
|---|---|---|
| Local | β_L only | No |
| Global | β_G only | No |
| Modulus | sqrt(β_L² + β_G²) | No |
| Weighted OMC | (1-ρ)(S_My/N)β_L + ρ(α/n_c)β_G | Yes — size, density, neighbours |

### 5.5 SIR Epidemic Model

- Infection rate λ = 0.1 (above epidemic threshold λ_th ≈ 0.052)
- Recovery rate γ = 0.1
- Seeds: top f0 × N nodes ranked by the centrality measure under test
- Averaged over 30 independent runs per (method, f0) combination
- Metric: Δr = (R_c - R_s) / R_s × 100% where R_s = standard degree result

---

## 6. Expected Results

Netscience has **strong community structure** (μ ≈ 0.094). From Figure 16(a):

```
Local measure  → Δr > 0  (~+25% gain over standard degree)
Global measure → Δr < 0  (worse than standard degree)
Modulus        → Δr > 0  (~+37–42% gain)
Weighted OMC   → Δr > 0  (~+47–52% gain, highest)
```

> **Interpretation:** In tightly clustered co-authorship networks, local hubs within communities are the most effective spreaders. Cross-community bridging nodes (global measure) are less critical when communities are well-separated. These results replicate Figure 16(a) — Degree centrality panel.

---

## 7. File Structure

| File | Description |
|---|---|
| `netscience_OMC.ipynb` | Main notebook — all code |
| `data/netscience/edges.csv` | Edge list from Netzschleuder |

---

## 8. Dependencies

| Package | Version | Purpose |
|---|---|---|
| networkx | >=2.6 | Graph construction, centrality computation, SIR simulation |
| cdlib | >=0.2 | SLPA overlapping community detection |
| numpy | >=1.20 | Numerical operations, averaging |
| matplotlib | >=3.3 | All visualisations |
| pandas | >=1.2 | Loading `edges.csv` from Netzschleuder |

All dependencies are installed automatically in the first notebook cell via `pip`.
