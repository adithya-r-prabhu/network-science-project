# Overlapping Modular Centrality — Replication Study

Replication of the paper:

> Ghalmane, Z., Cherifi, C., Cherifi, H. & El Hassouni, M.
> **Centrality in Complex Networks with Overlapping Community Structure**
> *Scientific Reports*, 9, 10133 (2019)
> DOI: [10.1038/s41598-019-46507-y](https://doi.org/10.1038/s41598-019-46507-y)

---

## Overview

The paper introduces **Overlapping Modular Centrality (OMC)**, a framework that extends classical centrality measures (Degree, Betweenness, Closeness, Eigenvector) to networks with overlapping community structure. Each node's influence is represented as a two-dimensional vector:

- **Local component (beta_L)** — influence within the node's own community via intra-community links
- **Global component (beta_G)** — influence across communities via inter-community links

Two combined ranking measures are also evaluated:
- **Modulus** — Euclidean magnitude of the OMC vector
- **Weighted OMC** — a linear combination weighted by community topology (community size, inter-community link fraction, number of foreign communities)

Performance is evaluated using the **SIR epidemic spreading model**: top-ranked nodes are used as initial spreaders, and the relative improvement in outbreak size (Delta r) over standard degree centrality is measured.

---

## Repository Structure

```
.
├── notebooks/
│   ├── omc_complete_replication.ipynb  # ⭐ Complete replication with LFR networks
│   ├── ego_facebook_OMC.ipynb          # ego-Facebook (STRONG, μ ~ 0.075)
│   ├── caltech_OMC.ipynb               # Caltech (MEDIUM, μ ~ 0.27)
│   ├── netscience_OMC.ipynb            # Netscience (STRONG, μ ~ 0.094)
│   └── extended.ipynb                  # ⭐ CA-GrQc extended dataset (5,242 nodes)
├── figures/                            # ⭐ All generated plots stored here
├── data/
│   ├── ego-facebook/
│   │   └── facebook_combined.txt
│   ├── caltech/
│   │   └── socfb-Caltech36.mtx
│   ├── netscience/
│   │   └── edges.csv
│   └── extended/
│       └── Extended_dataset.txt        # CA-GrQc collaboration network
├── docs/
└── paper/
    └── paper.pdf
```

---

## Extended Replication Work

This repository has been enhanced with comprehensive replication code for all experiments from Ghalmane et al. (2019):

### Available Notebooks

| Notebook | Description | Runtime |
|----------|-------------|---------|
| **`omc_complete_replication.ipynb`** | Complete replication with synthetic LFR networks (Figures 3-6) | ~30 min |
| **`ego_facebook_OMC.ipynb`** | Enhanced ego-Facebook analysis with full visualizations | ~25 min |
| **`caltech_OMC.ipynb`** | Caltech Facebook network analysis | ~10 min |
| **`netscience_OMC.ipynb`** | Netscience co-authorship network analysis | ~10 min |
| **`extended.ipynb`** | CA-GrQc collaboration network analysis (5,242 nodes) | ~50 min |

### Key Enhancements

- ✅ **LFR Network Generator** — Synthetic networks with controlled μ, oₙ, oₘ parameters
- ✅ **Complete OMC Framework** — Local, Global, Modulus, and Weighted centrality
- ✅ **SIR Simulation Engine** — Epidemic spreading evaluation with proper averaging
- ✅ **30+ Visualizations** — Network samples, degree distributions, community analysis, centrality distributions, OMC performance plots
- ✅ **Replicated Figures** — Figures 3-6 (synthetic) and Figures 16-17 (real-world)

### Quick Start

```bash
cd notebooks

# Main replication (synthetic LFR experiments)
jupyter lab omc_complete_replication.ipynb

# Real-world datasets
jupyter lab ego_facebook_OMC.ipynb
jupyter lab caltech_OMC.ipynb
jupyter lab netscience_OMC.ipynb
jupyter lab extended.ipynb
```

---

## Generated Figures

All figures are automatically saved to the **`figures/`** directory:

### Synthetic Experiments (from `omc_complete_replication.ipynb`)
| Figure | Description |
|--------|-------------|
| `figure3_community_strength.png` | Community strength effect (μ = 0.1, 0.4, 0.6) |
| `figure4_local.png` | Local centrality vs overlapping nodes |
| `figure5_global.png` | Global centrality vs overlapping nodes |
| `figure6_modulus.png` | Modulus vs overlapping nodes |
| `figure16_17_realworld.png` | Real-world network comparisons |

### Real-World Datasets (from individual notebooks)
| Network | Figures Generated |
|---------|-------------------|
| ego-Facebook | `facebook_network_sample.png`, `facebook_degree_distribution.png`, `facebook_community_analysis.png`, `facebook_centrality_distribution.png`, `facebook_omc_results.png` |
| Caltech | `caltech_network_sample.png`, `caltech_degree_distribution.png`, `caltech_community_analysis.png`, `caltech_centrality_distribution.png`, `caltech_omc_results.png` |
| Netscience | `netscience_network_full.png`, `netscience_degree_distribution.png`, `netscience_community_analysis.png`, `netscience_centrality_distribution.png`, `netscience_omc_results.png` |
| CA-GrQc (extended) | `extended_network_sample.png`, `extended_degree_distribution.png`, `extended_community_analysis.png`, `extended_centrality_distribution.png`, `extended_omc_results.png` |

**Total: 30+ figures** covering network structure, community analysis, centrality measures, and OMC performance evaluation.

---

## Datasets

| Network | Nodes | Edges | mu | Community Strength | Source |
|---|---|---|---|---|---|
| ego-Facebook | 4,039 | 88,234 | 0.075 | Strong | [SNAP](https://snap.stanford.edu/data/ego-Facebook.html) |
| Caltech | 620 | 7,255 | 0.27 | Medium | [networkrepository.com](https://networkrepository.com/socfb-Caltech36.php) |
| Netscience | 1,589 | 2,742 | 0.094 | Strong | [Netzschleuder](https://networks.skewed.de/net/netscience) |

The `data/` directory contains all dataset files except `facebook_combined.txt`, which must be downloaded separately from SNAP (see below).

---

## Setup

**Requirements:** Python 3.8+

```bash
# Create and activate virtual environment
python -m venv .venv
source .venv/bin/activate        # Linux/macOS
.venv\Scripts\activate           # Windows

# Install dependencies
pip install networkx matplotlib cdlib numpy scipy pandas jupyter
```

---

## Running the Notebooks

```bash
jupyter lab
```

Open any notebook from the `notebooks/` directory and run all cells.

**Expected runtimes** (approximate, on a standard laptop):

| Notebook | Runtime |
|---|---|
| caltech_OMC.ipynb | ~2 min |
| netscience_OMC.ipynb | ~5 min |
| ego_facebook_OMC.ipynb | ~20–40 min (betweenness on large graph) |

---

## Getting the ego-Facebook Data

The `facebook_combined.txt` file is not included due to size. Download it from SNAP:

1. Go to https://snap.stanford.edu/data/ego-Facebook.html
2. Download `facebook_combined.txt.gz`
3. Extract and place `facebook_combined.txt` in `data/ego-facebook/`

---

## Methods

Each notebook follows this pipeline:

1. Load network and extract the largest connected component
2. Detect overlapping communities using **SLPA** (Speaker-Listener Label Propagation, t=100, r=0.1) via [cdlib](https://cdlib.readthedocs.io/)
3. Build two derived networks:
   - **G_l** (local network) — intra-community edges; overlapping nodes replicated per community
   - **G_g** (global network) — inter-community edges only
4. Compute local centrality on G_l and global centrality on G_g for all four measures
5. Combine into OMC vectors; compute Modulus and Weighted OMC rankings
6. Run SIR simulations (lambda=0.1, gamma=0.1, 30 runs per seed set) over seed fractions f0 in [2%, 14%]
7. Plot Delta r vs f0 and summary tables

---

## Key Results (from the paper)

**Strong community structure (ego-Facebook, Netscience, mu < 0.15):**
Local component outperforms standard centrality (~+20–37%). Global component performs worse than standard. Modulus and Weighted OMC give the best results (~+40–52%).

**Medium community structure (Caltech, mu ~ 0.27):**
Both local and global components outperform standard centrality. Global component gains more importance as mu increases. Combined measures remain optimal.

---

## Dependencies

| Package | Purpose |
|---|---|
| networkx | Graph construction, centrality computation, SIR simulation |
| cdlib | SLPA overlapping community detection |
| numpy | Numerical operations |
| matplotlib | Visualisation |
| scipy | Loading `.mtx` files (Caltech only) |
| pandas | Loading Netzschleuder CSV files (Netscience only) |
