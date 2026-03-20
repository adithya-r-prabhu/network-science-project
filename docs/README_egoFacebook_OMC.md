# Overlapping Modular Centrality — ego-Facebook Dataset

## 1. Project Overview

This notebook replicates the Overlapping Modular Centrality (OMC) framework on the ego-Facebook dataset, as described in:

> Ghalmane, Z., Cherifi, C., Cherifi, H. & El Hassouni, M.
> **"Centrality in Complex Networks with Overlapping Community Structure"**
> *Scientific Reports*, 9, 10133 (2019)
> DOI: https://doi.org/10.1038/s41598-019-46507-y

The ego-Facebook network is the **primary benchmark dataset** used throughout the paper. It appears in Figure 16 and Figure 19, and is used for the algorithm comparison experiments in Figure 20. It has **strong community structure** (μ ≈ 0.075) — the strongest among all seven real-world networks tested.

The OMC framework represents each node's influence as a two-dimensional vector:

- **β_L** (Local component) — influence within the node's own community
- **β_G** (Global component) — influence across communities via inter-community links
- Combined ranking measures (Modulus, Weighted OMC) give the best epidemic spreading performance

---

## 2. ego-Facebook Dataset

### 2.1 Description

The ego-Facebook network is derived from Facebook app data collected by Leskovec & Mcauley (2012). It captures friendship connections between Facebook users, structured around ego networks (local networks centred on individual users).

- Nodes represent individual Facebook users (anonymised)
- Edges represent mutual friendship connections
- Undirected and unweighted
- Only the largest connected component is used (4,039 nodes)
- Known for very dense, well-separated community clusters (friend circles)

### 2.2 Network Statistics

| Property | Value | Paper Reference |
|---|---|---|
| Nodes (N) | 4,039 | Table 5 |
| Edges (E) | 88,234 | Table 5 |
| Average degree ⟨k⟩ | 43.69 | Table 5 |
| Max degree k_max | 1,045 | Table 5 |
| Clustering coefficient C | 0.605 | Table 5 |
| Epidemic threshold λ_th | ~0.009 | Table 5 |
| Mixing parameter μ | ~0.075 | Table 1 |
| Overlapping nodes | ~8.33% | Table 1 |
| Avg memberships m | ~1.89 | Table 1 |
| Community strength | **STRONG** | Lowest μ of all 7 networks |

### 2.3 Download & File Format

**Source:** SNAP (Stanford Network Analysis Project)
**Download:** https://snap.stanford.edu/data/ego-Facebook.html
**File:** `facebook_combined.txt.gz` — extract to get `facebook_combined.txt`

`facebook_combined.txt` is a plain edge list with one edge per line (no header):

```
0 1
0 2
0 3
...
```

Load with:

```python
G = nx.read_edgelist('facebook_combined.txt', nodetype=int)
```

---

## 3. Code Structure

The notebook `ego_facebook_OMC.ipynb` is structured as follows:

| Section | Description | Key Output |
|---|---|---|
| 1. Install dependencies | `pip install networkx, cdlib, matplotlib, numpy` | Packages ready |
| 2. Load Network | Read `facebook_combined.txt`, extract LCC | Graph G |
| 3. Network Visualisation | Spring layout of 200-node sample | Network plot |
| 4. Community Detection (SLPA) | SLPA with t=100, r=0.1 | `communities_list` |
| 5. Community Visualisation | Colour-coded community plot | Community plot |
| 6. Network Decomposition | Build G_l (local) and G_g (global) | Gl, Gg graphs |
| 7. Compute OMC | Local + global centrality for 4 measures | `omc_all` dict |
| 8. Ranking Measures | Modulus and Weighted OMC, Top-10 table | Ranked nodes |
| 9. SIR Simulation | Epidemic spreading λ=0.1, γ=0.1, 30 runs | `res_*` dicts |
| 10. Δr vs f0 Plot | Relative gain over standard degree centrality | Main result plot |
| 11. Summary Table | Top node per measure, all 4 centralities | Summary table |
| 12. Centrality Distribution | Histogram of local vs global score distributions | Distribution plot |
| 13. Top-10 Subgraph | Visualise most influential nodes | Subgraph plot |

---

## 4. How to Run

### 4.1 Requirements

- Google Colab (recommended) or Jupyter Notebook with Python 3.8+
- Internet connection for `pip install` in the first cell
- `facebook_combined.txt` uploaded to the Colab session storage

### 4.2 Step-by-Step

1. Go to https://snap.stanford.edu/data/ego-Facebook.html
2. Download `facebook_combined.txt.gz` and extract it
3. Open the notebook in Google Colab
4. Upload `facebook_combined.txt` via the Colab Files panel (left sidebar)
5. Run all cells: **Runtime → Run All**

### 4.3 Load Cell

```python
G = nx.read_edgelist('facebook_combined.txt', nodetype=int)
G = G.subgraph(max(nx.connected_components(G), key=len)).copy()
print(f'Nodes : {G.number_of_nodes():,}')   # expected: 4,039
print(f'Edges : {G.number_of_edges():,}')   # expected: 88,234
```

Expected output:

```
Nodes : 4,039
Edges : 88,234
Epidemic threshold λ_th = 0.0095  (paper: 0.009)
```

---

## 5. Methods Summary

### 5.1 Community Detection — SLPA

The Speaker-Listener Label Propagation Algorithm (SLPA) is used to detect overlapping communities.

- `t = 100` (iterations)
- `r = 0.1` (threshold — labels in < 10% of memory are dropped)
- Implementation: `cdlib.algorithms.slpa(G, t=100, r=0.1)`
- Expected result: ~11 communities, ~6 overlapping nodes, avg membership 1.00

### 5.2 Network Decomposition

| Network | Description |
|---|---|
| **G_l (Local)** | Built by replicating each community as an isolated subgraph. Overlapping nodes appear in all their communities. Node labels are `(node_id, community_id)` tuples. Result: 4,045 nodes, 87,273 edges. |
| **G_g (Global)** | Inter-community edges only. Isolated nodes removed. Result: 478 nodes, 962 edges. |

### 5.3 OMC Components

For each of the four centrality measures (Degree, Betweenness, Closeness, Eigenvector):

- **β_L** — computed on G_l, aggregated per node by taking the max across community replicas
- **β_G** — computed on G_g; nodes not in G_g get score 0.0
- **B_OM(v)** = (β_L, β_G) — the two-dimensional OMC vector

### 5.4 Ranking Measures

| Measure | Formula | Paper Eq. |
|---|---|---|
| Local | β_L only | Eq. 1 |
| Global | β_G only | Eq. 1 |
| Modulus | sqrt(β_L² + β_G²) | Eq. 3 |
| Weighted OMC | (1-ρ)(S_My/N)β_L + ρ(α/n_c)β_G | Eq. 4–7 |

### 5.5 Top-10 Nodes by Modulus (Degree Centrality)

```
 Node         Type    Local   Global  Modulus
--------------------------------------------------
  107  Non-overlap   0.2527   0.0482   0.2573
 1684  Non-overlap   0.1921   0.0314   0.1947
 1912  Non-overlap   0.1860   0.0063   0.1861
  483  Non-overlap   0.0391   0.1530   0.1579
  414  Non-overlap   0.0215   0.1509   0.1525
```

Node 107 is the dominant hub (highest local degree centrality). Nodes 483 and 414 are important inter-community bridges (high global scores).

### 5.6 SIR Epidemic Model

- Infection rate λ = 0.1 (well above threshold λ_th ≈ 0.009)
- Recovery rate γ = 0.1
- Seeds: top f0 × N nodes by centrality measure under test
- Averaged over 30 independent runs per (method, f0) pair
- Metric: Δr = (R_c - R_s) / R_s × 100% vs standard degree

---

## 6. Expected Results

ego-Facebook has the **strongest community structure** of all networks tested (μ ≈ 0.075). From Figure 16(a):

```
Local measure  → Δr > 0  (strongly outperforms standard degree, ~+25–37% gain)
Global measure → Δr < 0  (worse than standard degree, ~−18%)
Modulus        → Δr > 0  (best combination, ~+39–47% gain)
Weighted OMC   → Δr > 0  (highest overall, ~+46–52% gain for degree centrality)
```

> **Interpretation:** Communities in ego-Facebook are dense, well-separated friend circles. Targeting high-local nodes (community hubs) is optimal. Global bridges are less critical since inter-community links are sparse.

### 6.1 Summary Table (All Four Centrality Measures)

| Measure | Top Local Node | Top Global Node | Top Modulus Node |
|---|---|---|---|
| Degree | 107 (0.2527) | 483 (0.1530) | 107 |
| Betweenness | 107 (0.0492) | 107 (0.5533) | 107 |
| Closeness | 107 (0.2527) | 107 (0.4091) | 107 |
| Eigenvector | 1912 (0.0954) | 414 (0.3243) | 414 |

---

## 7. Comparison with Other Dataset Notebooks

| Aspect | ego-Facebook | Other notebooks |
|---|---|---|
| File format | Plain `.txt` edge list (SNAP format) | `edges.csv` from Netzschleuder or `.mtx` |
| Load function | `nx.read_edgelist()` directly | Custom loader needed |
| Network size | 4,039 nodes, 88,234 edges (large) | 620–7,651 nodes |
| Community type | Very strong (μ = 0.075) | Medium or weak |
| Expected Δr | Local >> Standard >> Global | Varies by dataset |
| Overlapping nodes | ~8.33%, avg membership 1.89 | Varies (Table 1) |

---

## 8. File Structure

| File | Description |
|---|---|
| `ego_facebook_OMC.ipynb` | Main notebook — full pipeline |
| `data/ego-facebook/facebook_combined.txt` | Edge list (download from SNAP) |

---

## 9. Dependencies

| Package | Version | Purpose |
|---|---|---|
| networkx | >=2.6 | Graph construction, centrality measures, SIR simulation |
| cdlib | >=0.2 | SLPA overlapping community detection |
| numpy | >=1.20 | Numerical averaging, degree moment calculations |
| matplotlib | >=3.3 | All plots |

All installed automatically by the first notebook cell.
