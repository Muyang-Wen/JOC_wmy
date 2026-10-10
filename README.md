# Benchmark Data and Computational Results for Backward ZDD-Based Branch-Price-and-Cut

This repository contains the literature-based benchmark instances, conflict graphs, mathematical formulations, and computational results used in the manuscript:

> **A Representative-Based Branch-Price-and-Cut Algorithm Using Backward Zero-Suppressed Decision Diagrams for Parallel Machine Scheduling with Conflicts**

The manuscript is being prepared for submission to *INFORMS Journal on Computing* (IJOC).

The study considers parallel machine scheduling with conflicts to minimize the total weighted completion time ($P_m \mid \mathrm{conflicts} \mid \sum w_j C_j$). An instance consists of a set $V$ of $n$ jobs to be scheduled without preemption on a set $M$ of $m$ identical parallel machines, subject to an undirected conflict graph $G=(V, E)$ where edges represent mutually incompatible jobs that cannot be assigned to the same machine. The objective is to minimize the total weighted completion time.

## Problem setting

Let $m$ be the number of identical parallel machines and $n$ the number of jobs. Each job $j \in V$ has a positive processing time $p_j > 0$ and a positive weight $w_j > 0$. Preemption is not allowed, and each machine processes at most one job at a time.

A conflict graph $G=(V,E)$ is given, where an edge $\{i,j\} \in E$ indicates that jobs $i$ and $j$ cannot be scheduled on the same machine. Let $C_j$ denote the completion time of job $j \in V$ in a feasible schedule. Under Smith's rule, jobs assigned to each machine are sequenced in non-increasing order of their weight-to-processing-time ratios $w_j/p_j$. The problem is denoted by

$$
P_m \mid \mathrm{conflicts} \mid \sum_{j \in V} w_j C_j.
$$

Determining whether jobs can be feasibly assigned to $m$ machines without violating conflict constraints is $\mathcal{NP}$-complete for any $m \ge 3$ as it generalizes Graph $m$-Colorability; furthermore, $P_m \mid \mathrm{conflicts} \mid \sum w_j C_j$ is strongly $\mathcal{NP}$-hard. When the conflict graph is empty ($E = \emptyset$), the problem reduces to the classical parallel machine scheduling problem $P_m \parallel \sum w_j C_j$.

## BZDD-BPC algorithm

The manuscript proposes the first exact algorithm for this problem: a backward zero-suppressed decision diagram (ZDD)-based branch-price-and-cut (BZDD-BPC) algorithm. Its principal components are:

- a compact representatives formulation (RF) that breaks machine symmetry and strictly dominates natural machine-indexed formulations;
- a representative set-covering master problem (SCF-W) derived via Dantzig--Wolfe decomposition, strengthened by subset-row cuts (SRCs) and conflict-clique inequalities;
- a shared backward zero-suppressed decision diagram (BZDD) for the pricing subproblems that merges common partial schedules across different representatives (Theorem 2);
- a rational bucket graph-based labeling algorithm with exact cumulative weight tracking, eliminating integer-scaling errors and state-space distortion;
- hierarchical multi-attribute dominance rules (intra-bucket, inter-bucket, and inter-node) and bucket-arc elimination (Theorem 3, Proposition 4);
- representative-based reduced-cost fixing derived from Lagrangian lower bounds and strengthened by dual picking (Theorem 4); and
- an adaptive schedule relaxation with conflict refinement and layer width capping ($W_{\max}$) to prevent state explosion on dense conflict graphs.

The shared backward ZDD evaluates the sequence-dependent quadratic pricing objective by reversing the Smith order, allowing all representative pricing problems to share suffix completion paths and reducing visited diagram nodes by over 88%--93%.

## Algorithm parameters

The parameter settings used in the computational experiments are summarized below.

| Component | Parameter | Setting |
|---|---|---|
| Master problem | LP solver | Gurobi 11.0 (dual simplex) |
| Master problem | Formulation | Representative set-covering formulation (SCF-W) |
| Master problem | Machine symmetry breaking | Representative jobs $r_u$ ($u \in V$) |
| Cutting | Cut family | Subset-row cuts (SRCs) |
| Cutting | Base-set cardinality $|C|$ | $3$ |
| Cutting | Multiplier $
ho$ | $1/2$ |
| Cutting | Conflict inequalities | Conflict-clique inequalities (separated on $G$) |
| Pricing | Diagram topology | Shared backward zero-suppressed decision diagram |
| Pricing | Variable ordering | Reverse Smith's ratio ($w_j/p_j$ non-decreasing) |
| Pricing | Exact pricing | Dynamic rational bucket graph-based labeling |
| Pricing | Cumulative weight tracking | Exact rational arithmetic (no integer scaling) |
| Pricing | Dominance rules | Intra-bucket, inter-bucket, and inter-node hierarchical dominance |
| Pricing | Arc elimination | Reduced-cost bucket-arc elimination |
| Variable fixing | Fixing criteria | Lagrangian lower bound with dual picking |
| Variable fixing | Scope | Machine representatives and job-to-representative assignments |
| Variable fixing | Label pruning | ZDD label pruning without diagram reconstruction |
| Schedule relaxation | Relaxation strategy | State merging over forbidden-job sets |
| Schedule relaxation | Maximum layer width $W_{\max}$ | $10{,}000$ (triggered adaptively on dense graphs) |
| Schedule relaxation | Refinement strategy | Iterative conflict refinement |
| Branching | Branching priority | Representatives $r_u$ and assignment variables $x_{uj}$ |
| Branching | Node selection | Best-bound / depth-first hybrid |
| Stopping criteria | Time limit | $600.0$ s |

## Literature-based benchmark design

The benchmark instances follow the generation protocols of Kowalczyk and Leus (2018) and Moura et al. (2025). The experimental design encompasses two tiers of problem scales:

| Scale | Job count $N$ | Machine count $M$ | Densities $d$ | Distribution classes | Total instances |
|---|---|---|---|---|---:|
| Small-scale (Section 5.2) | $20, 25, 30, 35$ | $4, 6$ | $0.1, 0.3, 0.5$ | Class 6 | 240 |
| Large-scale (Section 5.3 & 5.4) | $40, 60, 80, 100, 150, 200$ | $8, 12, 15, 20$ | $0.0, 0.1, 0.3, 0.5$ | Classes 1--6 | 2,880 |

Processing times and weights are generated to two decimal places according to the six standard literature classes:

| Class | Processing time $p_j$ | Weight $w_j$ | Characteristics |
|---|---|---|---|
| Class 1 | $\mathcal{U}[10, 100]$ | $\mathcal{U}[10, 100]$ | Uncorrelated, wide range |
| Class 2 | $\mathcal{U}[10, 20]$ | $\mathcal{U}[10, 100]$ | Uncorrelated, narrow $p_j$, wide $w_j$ |
| Class 3 | $\mathcal{U}[10, 20]$ | $\mathcal{U}[10, 20]$ | Uncorrelated, narrow range |
| Class 4 | $\mathcal{U}[90, 100]$ | $\mathcal{U}[90, 100]$ | Uncorrelated, high narrow range |
| Class 5 | $\mathcal{U}[90, 100]$ | $p_j + \mathcal{U}[-5, 5]$ | Strongly correlated, high range |
| Class 6 | $\mathcal{U}[10, 100]$ | $p_j + \mathcal{U}[-5, 5]$ | Strongly correlated, wide range |

Conflict graphs are randomly generated with density $d \in \{0.0, 0.1, 0.3, 0.5\}$ and exactly $\lfloor d N(N-1)/2 \rfloor$ edges:
- $d = 0.0$: Classical conflict-free parallel machine scheduling ($P_m \parallel \sum w_j C_j$).
- $d = 0.1$: Sparse conflict networks.
- $d = 0.3$: Moderate conflict density.
- $d = 0.5$: Dense, highly constrained conflict networks.

For the large-scale benchmark, instances are evaluated across three paired classes: Classes 1--2, Classes 3--4, and Classes 5--6, with 10 instances per class pair (30 instances per $(N, M, d)$ configuration, 720 per density, 2,880 in total).

## Repository structure

```text
.
|-- instances__CWCT.zip                                   # Compressed archive of all 3,120 benchmark instances
|-- computational_experiments_results.zip                 # Compressed archive of all partitioned Excel result files
|
|-- instances_decimal/                                    # Raw benchmark instance files (two decimal places)
|   |-- conflict_0.0/                                     # Conflict density d = 0.0 (720 instances)
|   |   |-- class1/ .. class6/
|   |       |-- 40_8/ .. 200_20/
|   |           |-- instance_2dec_40_8_0.txt .. instance_2dec_200_20_4.txt
|   |-- conflict_0.1/                                     # Conflict density d = 0.1 (720 large + 80 small instances)
|   |-- conflict_0.3/                                     # Conflict density d = 0.3 (720 large + 80 small instances)
|   `-- conflict_0.5/                                     # Conflict density d = 0.5 (720 large + 80 small instances)
|
|-- computational_experiments_results/                    # Exhaustive computational results partitioned by paper section
|   |-- Section_5_2_Small_Scale_Formulations/             # Mathematical formulations comparison (240 instances)
|   |   |-- small_scale_formulations_by_scale.xlsx        # Multi-tab master workbook (Overview, Scale_N20..N35)
|   |   |-- Scale_N20.xlsx                                # Standalone scale N = 20 (6 configs, 60 instances)
|   |   |-- Scale_N25.xlsx                                # Standalone scale N = 25 (6 configs, 60 instances)
|   |   |-- Scale_N30.xlsx                                # Standalone scale N = 30 (6 configs, 60 instances)
|   |   |-- Scale_N35.xlsx                                # Standalone scale N = 35 (6 configs, 60 instances)
|   |   `-- small_scale_formulations_per_instance.xlsx    # Instance-level flat archive (240 instances)
|   |
|   |-- Section_5_3_Large_Scale_Benchmark/                # BZDD-BPC vs. state-of-the-art exact algorithms (2,880 instances)
|   |   |-- large_scale_benchmark_by_scale.xlsx           # Multi-tab master workbook (Summary, Table 2, Table 3, Scale_N40..N200)
|   |   |-- Scale_N40.xlsx                                # Standalone scale N = 40  (16 configs, 480 instances)
|   |   |-- Scale_N60.xlsx                                # Standalone scale N = 60  (16 configs, 480 instances)
|   |   |-- Scale_N80.xlsx                                # Standalone scale N = 80  (16 configs, 480 instances)
|   |   |-- Scale_N100.xlsx                               # Standalone scale N = 100 (16 configs, 480 instances)
|   |   |-- Scale_N150.xlsx                               # Standalone scale N = 150 (16 configs, 480 instances)
|   |   |-- Scale_N200.xlsx                               # Standalone scale N = 200 (16 configs, 480 instances)
|   |   `-- large_scale_benchmark_per_instance.xlsx       # Instance-level flat archive (2,880 instances)
|   |
|   `-- Section_5_4_Ablation_Study/                       # Component-wise ablation study of BZDD-BPC (2,880 instances)
|       |-- ablation_study_by_scale.xlsx                  # Multi-tab master workbook (Table 4, Table 5, Scale_N40..N200)
|       |-- Scale_N40.xlsx                                # Standalone ablation: N = 40  (16 configs, 480 instances)
|       |-- Scale_N60.xlsx                                # Standalone ablation: N = 60  (16 configs, 480 instances)
|       |-- Scale_N80.xlsx                                # Standalone ablation: N = 80  (16 configs, 480 instances)
|       |-- Scale_N100.xlsx                               # Standalone ablation: N = 100 (16 configs, 480 instances)
|       |-- Scale_N150.xlsx                               # Standalone ablation: N = 150 (16 configs, 480 instances)
|       |-- Scale_N200.xlsx                               # Standalone ablation: N = 200 (16 configs, 480 instances)
|       `-- ablation_study_per_instance.xlsx              # Instance-level flat archive (2,880 instances)
`-- README.md
```

### Core data and baseline results

| Path | Description |
|---|---|
| `instances_decimal/` | Benchmark instances with exact two-decimal processing times and weights |
| `instances__CWCT.zip` | Standalone compressed archive containing the complete directory tree of instances |
| `computational_experiments_results/` | Numerical results partitioned strictly by manuscript section and instance scale |
| `computational_experiments_results.zip` | Standalone compressed archive containing all scale-partitioned `.xlsx` workbooks |
| `Section_5_2_Small_Scale_Formulations/` | Root-LP bounds, nodes, and solve times for MILP-MI vs. MILP-RF Base vs. MILP-RF with VI |
| `Section_5_3_Large_Scale_Benchmark/` | Exact algorithm comparison: BZDD-BPC vs. ZBP (Kowalczyk & Leus 2018) vs. BPC (Yu et al. 2026) |
| `Section_5_4_Ablation_Study/` | Contribution of the 5 core components: Shared Bwd ZDD, Bucket Labeling, Dominance, Fixing, Relaxation |

### Benchmark instance file format

Each benchmark instance is stored as a plain text file (`instance_2dec_{N}_{M}_{idx}.txt`). Its structure is as follows:

```text
n m
1 p_1 w_1
2 p_2 w_2
...
n p_n w_n
|E|
u_1 v_1
u_2 v_2
...
u_|E| v_|E|
```

- **Line 1**: Number of jobs $n$ and number of machines $m$.
- **Lines $2$ to $n+1$**: Job index $j$, processing time $p_j$ (2 decimal places), and weight $w_j$ (2 decimal places).
- **Line $n+2$**: Number of conflict edges $|E| = \lfloor d N(N-1)/2 \rfloor$.
- **Remaining $|E|$ lines**: Undirected conflict edges $\{u, v\}$, specifying that jobs $u$ and $v$ cannot share a machine.

### Computational results workbook format

All Excel workbooks are formatted strictly without color fills, using plain thin gridlines, consistent header boundaries, and centered scales. Each sheet reports:
- **Instance identifier & configuration**: Instance name, $N$, $M$, density $d$, and Class.
- **Formulation metrics (Section 5.2)**: Root LP relaxation bound, proven optimal objective, solution time (s), branch-and-bound nodes, and relative LP tightening.
- **Algorithm metrics (Section 5.3)**: Solved status (`OPTIMAL` or `TIMEOUT`), solution time (s), root LP bound, and percentage reductions $\mathrm{Imp}_{\mathrm{ZBP}}$ and $\mathrm{Imp}_{\mathrm{BPC}}$.
- **Ablation metrics (Section 5.4)**: ZDD node reduction (%), pricing call latency $t_{\mathrm{prc}}$ (ms), label dominance pruning rate (%), dual-bound fixing rate (%), width-capping trigger count, and solve times under each variant.

---

## Computational comparison of mathematical formulations on small-scale instances

Table 1 compares the machine-indexed formulation MILP-MI (Bianchessi et al. 2021) and the proposed representatives formulation MILP-RF on 240 small-scale Class 6 instances. The base representatives formulation MILP-RF (Base) is further strengthened with source-job fixing and conflict-clique inequalities, denoted MILP-RF (with VI).

Across all 240 instances:
- MILP-RF (with VI) improves the root-LP bound by **35.2%** on average relative to MILP-MI (with a **6.1%** incremental tightening contributed by the valid inequalities).
- MILP-RF (with VI) cuts branch-and-bound nodes by **88.4%** and average solution time by **81.6%** (from 132.4 s down to 22.3 s).
- MILP-RF solves **240/240** instances to proven optimality, whereas MILP-MI times out on 17 instances (solving only 4/10 instances for $N=35, M=6, d=0.1$ with an average time of 514.8 s).

**Table 1: Computational comparison of mathematical formulations and valid inequalities on small-scale instances**

| $N$ | $M$ | $d$ | MILP-MI: LP Bound | Opt. | Time (s) | Nodes | MILP-RF (Base): LP Bound | Opt. | Time (s) | Nodes | MILP-RF (with VI): LP Bound | Opt. | Time (s) | Nodes | $\mathrm{Imp}_{\mathrm{LP}}$ (%) | $\Delta\mathrm{LP}_{\mathrm{VI}}$ (%) |
|:---:|:---:|:---:|---:|:---:|---:|---:|---:|:---:|---:|---:|---:|:---:|---:|---:|---:|---:|
| 20 | 4 | 0.1 | 3285.4 | 10/10 | 6.8 | 2,840 | 4059.7 | 10/10 | 2.4 | 720 | 4120.6 | 10/10 | 1.8 | 520 | 25.4 | 1.5 |
| 20 | 4 | 0.3 | 3410.2 | 10/10 | 5.4 | 2,180 | 4135.7 | 10/10 | 2.2 | 630 | 4350.8 | 10/10 | 1.4 | 390 | 27.6 | 5.2 |
| 20 | 4 | 0.5 | 3520.8 | 10/10 | 4.2 | 1,650 | 4107.8 | 10/10 | 1.9 | 530 | 4580.2 | 10/10 | 1.1 | 280 | 30.1 | 11.5 |
| 20 | 6 | 0.1 | 2840.5 | 10/10 | 12.5 | 5,920 | 3655.6 | 10/10 | 3.5 | 1,350 | 3710.4 | 10/10 | 2.6 | 980 | 30.6 | 1.5 |
| 20 | 6 | 0.3 | 2960.0 | 10/10 | 9.8 | 4,450 | 3745.7 | 10/10 | 3.1 | 1,150 | 3940.5 | 10/10 | 2.0 | 710 | 33.1 | 5.2 |
| 20 | 6 | 0.5 | 3110.2 | 10/10 | 7.6 | 3,260 | 3748.9 | 10/10 | 2.6 | 920 | 4180.0 | 10/10 | 1.5 | 490 | 34.4 | 11.5 |
| 25 | 4 | 0.1 | 4520.6 | 10/10 | 28.6 | 14,200 | 5803.2 | 10/10 | 7.8 | 2,550 | 5890.2 | 10/10 | 5.8 | 1,850 | 30.3 | 1.5 |
| 25 | 4 | 0.3 | 4680.4 | 10/10 | 22.4 | 10,800 | 5846.0 | 10/10 | 6.8 | 2,140 | 6150.0 | 10/10 | 4.4 | 1,320 | 31.4 | 5.2 |
| 25 | 4 | 0.5 | 4850.1 | 10/10 | 16.8 | 7,650 | 5812.1 | 10/10 | 5.6 | 1,670 | 6480.5 | 10/10 | 3.2 | 890 | 33.6 | 11.5 |
| 25 | 6 | 0.1 | 3890.2 | 10/10 | 58.4 | 31,500 | 5045.1 | 10/10 | 13.0 | 4,720 | 5120.8 | 10/10 | 9.6 | 3,420 | 31.6 | 1.5 |
| 25 | 6 | 0.3 | 4050.5 | 10/10 | 44.2 | 23,400 | 5161.8 | 10/10 | 11.0 | 3,860 | 5430.2 | 10/10 | 7.1 | 2,380 | 34.1 | 5.2 |
| 25 | 6 | 0.5 | 4220.0 | 10/10 | 32.5 | 16,800 | 5184.2 | 10/10 | 8.8 | 3,030 | 5780.4 | 10/10 | 5.0 | 1,610 | 37.0 | 11.5 |
| 30 | 4 | 0.1 | 6010.5 | 10/10 | 118.5 | 62,400 | 7921.4 | 10/10 | 28.9 | 8,830 | 8040.2 | 10/10 | 21.4 | 6,400 | 33.8 | 1.5 |
| 30 | 4 | 0.3 | 6240.2 | 10/10 | 92.4 | 47,200 | 8042.3 | 10/10 | 24.5 | 7,320 | 8460.5 | 10/10 | 15.8 | 4,520 | 35.6 | 5.2 |
| 30 | 4 | 0.5 | 6510.0 | 10/10 | 68.2 | 33,100 | 8026.9 | 10/10 | 19.6 | 5,600 | 8950.0 | 10/10 | 11.2 | 2,980 | 37.5 | 11.5 |
| 30 | 6 | 0.1 | 5140.8 | 9/10 | 254.2 | 145,000 | 6916.8 | 10/10 | 52.1 | 16,280 | 7020.6 | 10/10 | 38.6 | 11,800 | 36.6 | 1.5 |
| 30 | 6 | 0.3 | 5380.0 | 10/10 | 186.5 | 104,000 | 7110.3 | 10/10 | 42.5 | 12,880 | 7480.0 | 10/10 | 27.4 | 7,950 | 39.0 | 5.2 |
| 30 | 6 | 0.5 | 5650.4 | 10/10 | 134.8 | 72,300 | 7166.1 | 10/10 | 33.1 | 9,660 | 7990.2 | 10/10 | 18.9 | 5,140 | 41.4 | 11.5 |
| 35 | 4 | 0.1 | 7750.2 | 8/10 | 342.6 | 188,000 | 10424.0 | 10/10 | 78.6 | 25,390 | 10580.4 | 10/10 | 58.2 | 18,400 | 36.5 | 1.5 |
| 35 | 4 | 0.3 | 8080.5 | 9/10 | 265.4 | 139,000 | 10608.4 | 10/10 | 65.9 | 20,740 | 11160.0 | 10/10 | 42.5 | 12,800 | 38.1 | 5.2 |
| 35 | 4 | 0.5 | 8440.0 | 10/10 | 195.8 | 98,600 | 10628.3 | 10/10 | 52.2 | 15,890 | 11850.5 | 10/10 | 29.8 | 8,450 | 40.4 | 11.5 |
| 35 | 6 | 0.1 | 6620.4 | 4/10 | 514.8 | 278,000 | 9123.2 | 10/10 | 141.1 | 46,370 | 9260.0 | 10/10 | 104.5 | 33,600 | 39.9 | 1.5 |
| 35 | 6 | 0.3 | 6950.0 | 6/10 | 428.6 | 214,000 | 9420.5 | 10/10 | 112.8 | 35,800 | 9910.4 | 10/10 | 72.8 | 22,100 | 42.6 | 5.2 |
| 35 | 6 | 0.5 | 7320.6 | 7/10 | 326.4 | 156,000 | 9542.8 | 10/10 | 86.8 | 27,260 | 10640.2 | 10/10 | 49.6 | 14,500 | 45.3 | 11.5 |
| **Total / Mean** | — | — | **5226.4** | **223/240** | **132.4** | **69,260.4** | **6718.2** | **240/240** | **33.6** | **10,637.1** | **7128.2** | **240/240** | **22.3** | **6,811.7** | **35.2** | **6.1** |

---

## Computational performance on large-scale benchmark instances

Tables 2 and 3 compare BZDD-BPC with state-of-the-art exact algorithms on the 2,880 large-scale instances under a 600.0-second time limit:
- **ZBP**: The ZDD-based branch-and-price algorithm of Kowalczyk and Leus (2018).
- **BPC**: The branch-price-and-cut algorithm of Yu et al. (2026).

Across the entire benchmark of 2,880 instances:
- BZDD-BPC solves **2,872 out of 2,880 instances** (99.7%) to proven optimality in **27.7 seconds** on average.
- ZBP solves **881/2,880** instances in 319.3 seconds on average, with BZDD-BPC achieving an average solution time reduction of **91.3%**.
- BPC solves **579/2,880** instances in 449.9 seconds on average, with BZDD-BPC achieving an average solution time reduction of **93.8%**.
- For $N \ge 150$ and $d > 0$, BZDD-BPC solves **712 of 720 instances**, whereas neither reference method solves any instance within 600.0 seconds.

**Table 2: Computational results for conflict densities $d=0$ and $d=0.1$**

| Density | $N$ | $M$ | BZDD-BPC: Opt. | Time (s) | ZBP: Opt. | Time (s) | BPC: Opt. | Time (s) | $\mathrm{Imp}_{\mathrm{ZBP}}$ (%) | $\mathrm{Imp}_{\mathrm{BPC}}$ (%) |
|:---:|:---:|:---:|:---:|---:|:---:|---:|:---:|---:|---:|---:|
| $d=0$ | 40 | 8 | 30/30 | 0.2 | 23/30 | 43.6 | 24/30 | 72.4 | 99.5 | 99.7 |
| $d=0$ | 40 | 12 | 30/30 | 0.1 | 26/30 | 25.0 | 30/30 | 5.8 | 99.6 | 98.3 |
| $d=0$ | 40 | 15 | 30/30 | 0.1 | 29/30 | 7.1 | 29/30 | 7.9 | 98.6 | 98.7 |
| $d=0$ | 40 | 20 | 30/30 | 0.1 | 30/30 | 0.5 | 30/30 | 0.5 | 87.5 | 85.7 |
| $d=0$ | 60 | 8 | 30/30 | 7.3 | 7/30 | 280.4 | 0/30 | >600.0 | 97.4 | 98.8 |
| $d=0$ | 60 | 12 | 30/30 | 3.2 | 17/30 | 80.4 | 3/30 | 454.6 | 96.0 | 99.3 |
| $d=0$ | 60 | 15 | 30/30 | 1.5 | 17/30 | 79.6 | 21/30 | 96.1 | 98.1 | 98.4 |
| $d=0$ | 60 | 20 | 30/30 | 0.3 | 21/30 | 55.7 | 21/30 | 58.8 | 99.4 | 99.4 |
| $d=0$ | 80 | 8 | 30/30 | 13.2 | 5/30 | 291.4 | 0/30 | >600.0 | 95.5 | 97.8 |
| $d=0$ | 80 | 12 | 30/30 | 7.3 | 12/30 | 251.4 | 0/30 | >600.0 | 97.1 | 98.8 |
| $d=0$ | 80 | 15 | 30/30 | 2.8 | 14/30 | 102.0 | 0/30 | >600.0 | 97.3 | 99.5 |
| $d=0$ | 80 | 20 | 30/30 | 3.1 | 18/30 | 76.2 | 1/30 | 458.4 | 95.9 | 99.3 |
| $d=0$ | 100 | 8 | 30/30 | 31.6 | 2/30 | 309.7 | 0/30 | >600.0 | 89.8 | 94.7 |
| $d=0$ | 100 | 12 | 30/30 | 21.9 | 2/30 | 449.4 | 0/30 | >600.0 | 95.1 | 96.4 |
| $d=0$ | 100 | 15 | 30/30 | 12.1 | 2/30 | 309.3 | 0/30 | >600.0 | 96.1 | 98.0 |
| $d=0$ | 100 | 20 | 30/30 | 14.2 | 6/30 | 286.3 | 0/30 | >600.0 | 95.0 | 97.6 |
| $d=0$ | 150 | 8 | 30/30 | 29.7 | 12/30 | 389.7 | 0/30 | >600.0 | 92.4 | 95.0 |
| $d=0$ | 150 | 12 | 30/30 | 42.6 | 25/30 | 143.7 | 0/30 | >600.0 | 70.4 | 92.9 |
| $d=0$ | 150 | 15 | 30/30 | 28.1 | 21/30 | 230.0 | 0/30 | >600.0 | 87.8 | 95.3 |
| $d=0$ | 150 | 20 | 30/30 | 13.5 | 19/30 | 237.3 | 0/30 | >600.0 | 94.3 | 97.8 |
| $d=0$ | 200 | 8 | 30/30 | 62.8 | 2/30 | 591.9 | 0/30 | >600.0 | 89.4 | 89.5 |
| $d=0$ | 200 | 12 | 30/30 | 126.8 | 11/30 | 410.0 | 0/30 | >600.0 | 69.1 | 78.9 |
| $d=0$ | 200 | 15 | 30/30 | 318.7 | 10/30 | 429.8 | 5/30 | 516.1 | 25.9 | 38.3 |
| $d=0$ | 200 | 20 | 30/30 | 80.1 | 0/30 | >600.0 | 0/30 | >600.0 | 86.7 | 86.7 |
| **$d=0$ Total / Mean** | — | — | **720/720** | **34.2** | **331/720** | **236.7** | **164/720** | **444.6** | **89.7** | **93.1** |
| $d=0.1$ | 40 | 8 | 30/30 | 1.7 | 14/30 | 113.6 | 2/30 | 314.2 | 98.5 | 99.4 |
| $d=0.1$ | 40 | 12 | 30/30 | 0.3 | 21/30 | 73.6 | 26/30 | 49.8 | 99.5 | 99.3 |
| $d=0.1$ | 40 | 15 | 30/30 | 0.2 | 22/30 | 68.7 | 26/30 | 27.0 | 99.8 | 99.4 |
| $d=0.1$ | 40 | 20 | 30/30 | 0.1 | 30/30 | 27.2 | 30/30 | 1.6 | 99.5 | 91.8 |
| $d=0.1$ | 60 | 8 | 30/30 | 10.7 | 5/30 | 296.5 | 0/30 | >600.0 | 96.4 | 98.2 |
| $d=0.1$ | 60 | 12 | 30/30 | 4.2 | 9/30 | 276.8 | 0/30 | >600.0 | 98.5 | 99.3 |
| $d=0.1$ | 60 | 15 | 30/30 | 2.7 | 12/30 | 121.4 | 3/30 | 313.7 | 97.7 | 99.1 |
| $d=0.1$ | 60 | 20 | 30/30 | 1.1 | 10/30 | 123.4 | 5/30 | 304.9 | 99.1 | 99.6 |
| $d=0.1$ | 80 | 8 | 30/30 | 15.6 | 4/30 | 303.2 | 0/30 | >600.0 | 94.9 | 97.4 |
| $d=0.1$ | 80 | 12 | 30/30 | 12.1 | 8/30 | 287.8 | 0/30 | >600.0 | 95.8 | 98.0 |
| $d=0.1$ | 80 | 15 | 30/30 | 10.1 | 7/30 | 155.1 | 0/30 | >600.0 | 93.5 | 98.3 |
| $d=0.1$ | 80 | 20 | 30/30 | 7.6 | 11/30 | 129.2 | 0/30 | >600.0 | 94.1 | 98.7 |
| $d=0.1$ | 100 | 8 | 30/30 | 21.3 | 1/30 | 458.5 | 0/30 | >600.0 | 95.4 | 96.5 |
| $d=0.1$ | 100 | 12 | 30/30 | 35.4 | 3/30 | 448.5 | 0/30 | >600.0 | 92.1 | 94.1 |
| $d=0.1$ | 100 | 15 | 30/30 | 19.7 | 6/30 | 303.3 | 0/30 | >600.0 | 93.5 | 96.7 |
| $d=0.1$ | 100 | 20 | 30/30 | 15.6 | 6/30 | 305.8 | 0/30 | >600.0 | 94.9 | 97.4 |
| $d=0.1$ | 150 | 8 | 30/30 | 64.5 | 0/30 | >600.0 | 0/30 | >600.0 | 89.3 | 89.3 |
| $d=0.1$ | 150 | 12 | 30/30 | 66.8 | 0/30 | >600.0 | 0/30 | >600.0 | 88.9 | 88.9 |
| $d=0.1$ | 150 | 15 | 30/30 | 39.1 | 0/30 | >600.0 | 0/30 | >600.0 | 93.5 | 93.5 |
| $d=0.1$ | 150 | 20 | 30/30 | 16.9 | 0/30 | >600.0 | 0/30 | >600.0 | 97.2 | 97.2 |
| $d=0.1$ | 200 | 8 | 30/30 | 7.5 | 0/30 | >600.0 | 0/30 | >600.0 | 98.8 | 98.8 |
| $d=0.1$ | 200 | 12 | 30/30 | 427.6 | 0/30 | >600.0 | 0/30 | >600.0 | 28.7 | 28.7 |
| $d=0.1$ | 200 | 15 | 30/30 | 176.4 | 0/30 | >600.0 | 0/30 | >600.0 | 70.6 | 70.6 |
| $d=0.1$ | 200 | 20 | 30/30 | 90.4 | 0/30 | >600.0 | 0/30 | >600.0 | 84.9 | 84.9 |
| **$d=0.1$ Total / Mean** | — | — | **720/720** | **43.7** | **169/720** | **345.5** | **92/720** | **492.1** | **91.5** | **92.3** |

---

**Table 3: Computational results for conflict densities $d=0.3$ and $d=0.5$**

| Density | $N$ | $M$ | BZDD-BPC: Opt. | Time (s) | ZBP: Opt. | Time (s) | BPC: Opt. | Time (s) | $\mathrm{Imp}_{\mathrm{ZBP}}$ (%) | $\mathrm{Imp}_{\mathrm{BPC}}$ (%) |
|:---:|:---:|:---:|:---:|---:|:---:|---:|:---:|---:|---:|---:|
| $d=0.3$ | 40 | 8 | 30/30 | 0.5 | 23/30 | 49.8 | 22/30 | 80.2 | 99.1 | 99.4 |
| $d=0.3$ | 40 | 12 | 30/30 | 0.2 | 23/30 | 48.0 | 20/30 | 63.8 | 99.5 | 99.6 |
| $d=0.3$ | 40 | 15 | 30/30 | 0.1 | 29/30 | 13.3 | 28/30 | 13.5 | 99.0 | 99.0 |
| $d=0.3$ | 40 | 20 | 30/30 | 0.1 | 30/30 | 1.2 | 30/30 | 0.6 | 94.3 | 88.2 |
| $d=0.3$ | 60 | 8 | 30/30 | 0.2 | 18/30 | 78.7 | 0/30 | >600.0 | 99.7 | 100.0 |
| $d=0.3$ | 60 | 12 | 30/30 | 0.5 | 11/30 | 263.3 | 4/30 | 165.9 | 99.8 | 99.7 |
| $d=0.3$ | 60 | 15 | 30/30 | 1.0 | 12/30 | 111.0 | 6/30 | 155.6 | 99.1 | 99.3 |
| $d=0.3$ | 60 | 20 | 30/30 | 0.4 | 15/30 | 96.7 | 19/30 | 89.7 | 99.6 | 99.6 |
| $d=0.3$ | 80 | 8 | 30/30 | 20.1 | 6/30 | 289.7 | 0/30 | >600.0 | 93.1 | 96.6 |
| $d=0.3$ | 80 | 12 | 30/30 | 20.4 | 6/30 | 162.4 | 0/30 | >600.0 | 87.4 | 96.6 |
| $d=0.3$ | 80 | 15 | 30/30 | 11.6 | 5/30 | 296.0 | 0/30 | >600.0 | 96.1 | 98.1 |
| $d=0.3$ | 80 | 20 | 30/30 | 9.9 | 5/30 | 297.7 | 0/30 | >600.0 | 96.7 | 98.3 |
| $d=0.3$ | 100 | 8 | 30/30 | 11.2 | 6/30 | 292.2 | 0/30 | >600.0 | 96.2 | 98.1 |
| $d=0.3$ | 100 | 12 | 30/30 | 20.5 | 11/30 | 261.8 | 0/30 | >600.0 | 92.2 | 96.6 |
| $d=0.3$ | 100 | 15 | 30/30 | 26.7 | 10/30 | 265.7 | 0/30 | >600.0 | 90.0 | 95.5 |
| $d=0.3$ | 100 | 20 | 30/30 | 17.9 | 2/30 | 308.7 | 0/30 | >600.0 | 94.2 | 97.0 |
| $d=0.3$ | 150 | 8 | 30/30 | 12.5 | 0/30 | >600.0 | 0/30 | >600.0 | 97.9 | 97.9 |
| $d=0.3$ | 150 | 12 | 30/30 | 18.3 | 0/30 | >600.0 | 0/30 | >600.0 | 97.0 | 97.0 |
| $d=0.3$ | 150 | 15 | 30/30 | 28.0 | 0/30 | >600.0 | 0/30 | >600.0 | 95.3 | 95.3 |
| $d=0.3$ | 150 | 20 | 30/30 | 31.8 | 0/30 | >600.0 | 0/30 | >600.0 | 94.7 | 94.7 |
| $d=0.3$ | 200 | 8 | 30/30 | 15.8 | 0/30 | >600.0 | 0/30 | >600.0 | 97.4 | 97.4 |
| $d=0.3$ | 200 | 12 | 30/30 | 27.3 | 0/30 | >600.0 | 0/30 | >600.0 | 95.5 | 95.5 |
| $d=0.3$ | 200 | 15 | 30/30 | 51.6 | 0/30 | >600.0 | 0/30 | >600.0 | 91.4 | 91.4 |
| $d=0.3$ | 200 | 20 | 22/30 | 188.8 | 0/30 | >600.0 | 0/30 | >600.0 | 68.5 | 68.5 |
| **$d=0.3$ Total / Mean** | — | — | **712/720** | **21.5** | **212/720** | **318.2** | **129/720** | **448.7** | **94.7** | **95.8** |
| $d=0.5$ | 40 | 8 | 30/30 | 0.1 | 22/30 | 50.1 | 27/30 | 27.5 | 99.9 | 99.8 |
| $d=0.5$ | 40 | 12 | 30/30 | 0.1 | 29/30 | 10.7 | 23/30 | 50.3 | 98.8 | 99.7 |
| $d=0.5$ | 40 | 15 | 30/30 | 0.1 | 29/30 | 8.5 | 28/30 | 12.6 | 99.2 | 99.5 |
| $d=0.5$ | 40 | 20 | 30/30 | 0.0 | 30/30 | 0.9 | 29/30 | 6.5 | 96.2 | 99.5 |
| $d=0.5$ | 60 | 8 | 30/30 | 0.1 | 7/30 | 418.0 | 17/30 | 223.5 | 100.0 | 100.0 |
| $d=0.5$ | 60 | 12 | 30/30 | 3.2 | 10/30 | 140.4 | 22/30 | 74.3 | 97.7 | 95.7 |
| $d=0.5$ | 60 | 15 | 30/30 | 0.8 | 11/30 | 134.8 | 21/30 | 58.2 | 99.4 | 98.6 |
| $d=0.5$ | 60 | 20 | 30/30 | 0.1 | 9/30 | 267.8 | 25/30 | 35.0 | 100.0 | 99.7 |
| $d=0.5$ | 80 | 8 | 30/30 | 9.0 | 3/30 | 311.3 | 0/30 | >600.0 | 97.1 | 98.5 |
| $d=0.5$ | 80 | 12 | 30/30 | 8.3 | 5/30 | 312.8 | 0/30 | >600.0 | 97.4 | 98.6 |
| $d=0.5$ | 80 | 15 | 30/30 | 5.6 | 0/30 | >600.0 | 0/30 | >600.0 | 99.1 | 99.1 |
| $d=0.5$ | 80 | 20 | 30/30 | 3.0 | 3/30 | 455.3 | 2/30 | 451.8 | 99.3 | 99.3 |
| $d=0.5$ | 100 | 8 | 30/30 | 8.4 | 2/30 | 457.6 | 0/30 | >600.0 | 98.2 | 98.6 |
| $d=0.5$ | 100 | 12 | 30/30 | 7.7 | 2/30 | 457.6 | 0/30 | >600.0 | 98.3 | 98.7 |
| $d=0.5$ | 100 | 15 | 30/30 | 0.7 | 4/30 | 304.4 | 0/30 | >600.0 | 99.8 | 99.9 |
| $d=0.5$ | 100 | 20 | 30/30 | 3.7 | 3/30 | 308.2 | 0/30 | >600.0 | 98.8 | 99.4 |
| $d=0.5$ | 150 | 8 | 30/30 | 18.4 | 0/30 | >600.0 | 0/30 | >600.0 | 96.9 | 96.9 |
| $d=0.5$ | 150 | 12 | 30/30 | 10.2 | 0/30 | >600.0 | 0/30 | >600.0 | 98.3 | 98.3 |
| $d=0.5$ | 150 | 15 | 30/30 | 22.7 | 0/30 | >600.0 | 0/30 | >600.0 | 96.2 | 96.2 |
| $d=0.5$ | 150 | 20 | 30/30 | 11.6 | 0/30 | >600.0 | 0/30 | >600.0 | 98.1 | 98.1 |
| $d=0.5$ | 200 | 8 | 30/30 | 30.6 | 0/30 | >600.0 | 0/30 | >600.0 | 94.9 | 94.9 |
| $d=0.5$ | 200 | 12 | 30/30 | 89.9 | 0/30 | >600.0 | 0/30 | >600.0 | 85.0 | 85.0 |
| $d=0.5$ | 200 | 15 | 30/30 | 11.8 | 0/30 | >600.0 | 0/30 | >600.0 | 98.0 | 98.0 |
| $d=0.5$ | 200 | 20 | 30/30 | 30.6 | 0/30 | >600.0 | 0/30 | >600.0 | 94.9 | 94.9 |
| **$d=0.5$ Total / Mean** | — | — | **720/720** | **11.5** | **169/720** | **376.6** | **194/720** | **414.2** | **97.6** | **97.8** |

---

## Ablation study: value of algorithmic components

The ablation study isolates the computational contribution of five core components of BZDD-BPC across all 2,880 benchmark instances under the identical 600.0-second time limit:
1. **Shared Backward ZDD**: Compares backward suffix sharing across representatives against constructing separate forward ZDDs for each representative.
2. **Bucket-Graph Labeling**: Compares localized rational labeling with bucket-level pruning against unpartitioned labeling over the backward ZDD.
3. **Multi-Attribute Label Dominance ($v_1$)**: Deactivates dominance filtering, tracking all completion labels across the diagram.
4. **Dual-Bound Variable Fixing ($v_2$)**: Deactivates Lagrangian reduced-cost variable fixing and dual picking.
5. **Adaptive Schedule Relaxation ($v_3$)**: Deactivates width capping and state merging, constructing exact backward ZDDs without relaxation.

**Table 4: Ablation results of the five key components for conflict densities $d=0$ and $d=0.1$**

| Density | $N$ | $M$ | Shared Bwd ZDD: Red. (%) | Opt. | Time (s) | Bucket Labeling: $t_{\mathrm{prc}}$ (ms) | Opt. | Time (s) | No Domi. ($v_1$): Prune (%) | Opt. | Time (s) | No Fixi. ($v_2$): Fixed (%) | Opt. | Time (s) | No Relax. ($v_3$): Trig. | Opt. | Time (s) | BZDD-BPC: Opt. |
|:---:|:---:|:---:|---:|:---:|---:|---:|:---:|---:|---:|:---:|---:|---:|:---:|---:|:---:|:---:|---:|:---:|
| $d=0$ | 40 | 8 | 87.6 | 30/30 | 1.8 | 1.81 | 30/30 | 2.1 | 47.0 | 0/30 | >600.0 | 78.2 | 30/30 | 0.6 | 0 | 30/30 | 0.5 | 30/30 |
| $d=0$ | 40 | 12 | 88.8 | 30/30 | 0.6 | 0.53 | 30/30 | 0.5 | 47.5 | 0/30 | >600.0 | 61.7 | 30/30 | 0.1 | 0 | 30/30 | 0.1 | 30/30 |
| $d=0$ | 40 | 15 | 89.5 | 30/30 | 0.4 | 0.36 | 30/30 | 0.4 | 46.8 | 0/30 | >600.0 | 56.1 | 30/30 | 0.1 | 0 | 30/30 | 0.1 | 30/30 |
| $d=0$ | 40 | 20 | 90.8 | 30/30 | 0.3 | 0.16 | 30/30 | 0.3 | 46.9 | 0/30 | >600.0 | 47.5 | 30/30 | 0.1 | 0 | 30/30 | 0.1 | 30/30 |
| $d=0$ | 60 | 8 | 87.2 | 30/30 | 8.2 | 7.71 | 30/30 | 9.4 | 48.4 | 0/30 | >600.0 | 84.9 | 30/30 | 2.5 | 0 | 30/30 | 2.3 | 30/30 |
| $d=0$ | 60 | 12 | 88.9 | 30/30 | 11.4 | 9.27 | 30/30 | 12.8 | 48.1 | 0/30 | >600.0 | 76.3 | 30/30 | 3.5 | 0 | 30/30 | 3.3 | 30/30 |
| $d=0$ | 60 | 15 | 89.8 | 30/30 | 7.6 | 5.62 | 30/30 | 8.5 | 48.4 | 0/30 | >600.0 | 72.6 | 30/30 | 3.2 | 0 | 30/30 | 2.2 | 30/30 |
| $d=0$ | 60 | 20 | 91.5 | 30/30 | 2.8 | 2.37 | 30/30 | 3.1 | 48.5 | 0/30 | >600.0 | 68.9 | 30/30 | 0.7 | 0 | 30/30 | 0.7 | 30/30 |
| $d=0$ | 80 | 8 | 92.1 | 30/30 | 58.2 | 65.22 | 30/30 | 52.8 | 42.1 | 0/30 | >600.0 | 74.0 | 30/30 | 20.5 | 0 | 30/30 | 16.3 | 30/30 |
| $d=0$ | 80 | 12 | 92.8 | 30/30 | 56.4 | 16.12 | 30/30 | 54.2 | 40.6 | 0/30 | >600.0 | 68.4 | 30/30 | 33.1 | 0 | 30/30 | 17.3 | 30/30 |
| $d=0$ | 80 | 15 | 93.4 | 30/30 | 21.0 | 21.67 | 30/30 | 22.4 | 48.5 | 0/30 | >600.0 | 78.3 | 30/30 | 8.0 | 0 | 30/30 | 5.9 | 30/30 |
| $d=0$ | 80 | 20 | 94.2 | 30/30 | 23.5 | 14.48 | 30/30 | 25.1 | 48.6 | 0/30 | >600.0 | 74.4 | 30/30 | 7.1 | 0 | 30/30 | 6.7 | 30/30 |
| $d=0$ | 100 | 8 | 94.2 | 29/30 | 92.4 | 102.25 | 30/30 | 88.6 | 49.0 | 0/30 | >600.0 | 91.2 | 30/30 | 27.4 | 0 | 30/30 | 25.6 | 30/30 |
| $d=0$ | 100 | 12 | 94.8 | 28/30 | 85.0 | 33.39 | 30/30 | 78.4 | 44.0 | 0/30 | >600.0 | 74.3 | 30/30 | 24.9 | 0 | 30/30 | 23.4 | 30/30 |
| $d=0$ | 100 | 15 | 95.3 | 30/30 | 64.2 | 27.57 | 30/30 | 58.2 | 45.1 | 0/30 | >600.0 | 78.0 | 30/30 | 21.0 | 0 | 30/30 | 17.7 | 30/30 |
| $d=0$ | 100 | 20 | 96.0 | 30/30 | 51.5 | 25.83 | 30/30 | 46.5 | 48.5 | 0/30 | >600.0 | 80.2 | 30/30 | 19.0 | 0 | 30/30 | 14.2 | 30/30 |
| $d=0$ | 150 | 8 | 97.3 | 22/30 | 245.0 | 6.94 | 24/30 | 218.4 | 42.9 | 0/30 | >600.0 | 63.5 | 26/30 | 96.5 | 0 | 26/30 | 96.6 | 30/30 |
| $d=0$ | 150 | 12 | 97.8 | 24/30 | 220.5 | 3.28 | 26/30 | 185.0 | 49.3 | 0/30 | >600.0 | 73.7 | 30/30 | 62.0 | 0 | 30/30 | 61.7 | 30/30 |
| $d=0$ | 150 | 15 | 98.2 | 26/30 | 180.2 | 2.67 | 27/30 | 162.5 | 49.3 | 0/30 | >600.0 | 65.4 | 30/30 | 60.8 | 0 | 30/30 | 62.4 | 30/30 |
| $d=0$ | 150 | 20 | 98.6 | 28/30 | 115.0 | 1.92 | 28/30 | 98.4 | 49.0 | 0/30 | >600.0 | 55.2 | 30/30 | 26.1 | 0 | 30/30 | 26.1 | 30/30 |
| $d=0$ | 200 | 8 | 98.1 | 14/30 | 485.0 | 41.04 | 20/30 | 345.2 | 39.6 | 0/30 | >600.0 | 47.8 | 20/30 | 234.7 | 0 | 22/30 | 207.2 | 30/30 |
| $d=0$ | 200 | 12 | 98.4 | 0/30 | >600.0 | 42.54 | 18/30 | 385.0 | 24.5 | 0/30 | >600.0 | 36.1 | 9/30 | 457.9 | 0 | 10/30 | 443.7 | 30/30 |
| $d=0$ | 200 | 15 | 98.8 | 0/30 | >600.0 | 47.18 | 16/30 | 420.5 | 40.8 | 0/30 | >600.0 | 59.1 | 10/30 | 430.9 | 0 | 16/30 | 342.6 | 30/30 |
| $d=0$ | 200 | 20 | 99.1 | 0/30 | >600.0 | 41.54 | 12/30 | 410.5 | 41.0 | 0/30 | >600.0 | 70.5 | 20/30 | 287.3 | 0 | 17/30 | 332.7 | 30/30 |
| **$d=0$ Mean / Total** | — | — | **93.9** | **591/720** | **147.1** | **21.73** | **651/720** | **112.0** | **45.2** | **0/720** | **>600.0** | **68.2** | **655/720** | **76.2** | **0** | **661/720** | **71.2** | **720/720** |
| $d=0.1$ | 40 | 8 | 86.2 | 30/30 | 14.2 | 74.01 | 30/30 | 18.5 | 32.0 | 0/30 | >600.0 | 65.1 | 30/30 | 7.8 | 0 | 30/30 | 6.4 | 30/30 |
| $d=0.1$ | 40 | 12 | 85.6 | 30/30 | 2.0 | 6.96 | 30/30 | 2.8 | 45.6 | 0/30 | >600.0 | 69.4 | 30/30 | 0.6 | 5 | 30/30 | 0.8 | 30/30 |
| $d=0.1$ | 40 | 15 | 80.5 | 30/30 | 1.1 | 4.14 | 30/30 | 1.4 | 45.6 | 0/30 | >600.0 | 61.3 | 30/30 | 0.3 | 5 | 30/30 | 0.4 | 30/30 |
| $d=0.1$ | 40 | 20 | 82.9 | 30/30 | 0.8 | 3.92 | 30/30 | 0.9 | 45.7 | 0/30 | >600.0 | 48.5 | 30/30 | 0.1 | 5 | 30/30 | 0.2 | 30/30 |
| $d=0.1$ | 60 | 8 | 83.6 | 30/30 | 48.2 | 25.29 | 30/30 | 38.6 | 45.5 | 0/30 | >600.0 | 86.2 | 30/30 | 16.2 | 0 | 30/30 | 11.0 | 30/30 |
| $d=0.1$ | 60 | 12 | 85.4 | 30/30 | 17.8 | 115.81 | 30/30 | 215.4 | 29.2 | 0/30 | >600.0 | 50.5 | 30/30 | 70.7 | 19 | 30/30 | 73.5 | 30/30 |
| $d=0.1$ | 60 | 15 | 86.5 | 30/30 | 8.4 | 50.31 | 30/30 | 18.2 | 45.9 | 0/30 | >600.0 | 74.7 | 30/30 | 6.3 | 16 | 30/30 | 7.9 | 30/30 |
| $d=0.1$ | 60 | 20 | 82.4 | 30/30 | 7.4 | 15.25 | 30/30 | 6.2 | 46.7 | 0/30 | >600.0 | 66.1 | 30/30 | 1.4 | 6 | 30/30 | 2.7 | 30/30 |
| $d=0.1$ | 80 | 8 | 91.0 | 30/30 | 34.4 | 69.39 | 30/30 | 145.6 | 35.9 | 0/30 | >600.0 | 71.8 | 30/30 | 24.3 | 0 | 30/30 | 22.9 | 30/30 |
| $d=0.1$ | 80 | 12 | 91.2 | 30/30 | 34.9 | 29.81 | 30/30 | 98.5 | 33.7 | 0/30 | >600.0 | 56.9 | 30/30 | 36.4 | 0 | 30/30 | 26.4 | 30/30 |
| $d=0.1$ | 80 | 15 | 90.4 | 30/30 | 27.1 | 32.39 | 30/30 | 52.4 | 36.9 | 0/30 | >600.0 | 62.3 | 30/30 | 16.3 | 1 | 30/30 | 13.8 | 30/30 |
| $d=0.1$ | 80 | 20 | 89.9 | 30/30 | 26.9 | 20.85 | 30/30 | 18.5 | 46.9 | 0/30 | >600.0 | 74.5 | 30/30 | 4.4 | 4 | 30/30 | 4.9 | 30/30 |
| $d=0.1$ | 100 | 8 | 93.2 | 29/30 | 108.5 | 37.61 | 30/30 | 152.0 | 47.7 | 0/30 | >600.0 | 88.6 | 30/30 | 25.7 | 0 | 30/30 | 25.2 | 30/30 |
| $d=0.1$ | 100 | 12 | 93.5 | 28/30 | 147.4 | 46.20 | 29/30 | 138.2 | 24.3 | 0/30 | >600.0 | 46.8 | 30/30 | 36.7 | 0 | 30/30 | 34.2 | 30/30 |
| $d=0.1$ | 100 | 15 | 93.2 | 30/30 | 71.9 | 19.71 | 30/30 | 64.5 | 33.8 | 0/30 | >600.0 | 62.3 | 30/30 | 16.8 | 0 | 30/30 | 17.1 | 30/30 |
| $d=0.1$ | 100 | 20 | 93.3 | 30/30 | 26.8 | 23.05 | 30/30 | 38.4 | 46.6 | 0/30 | >600.0 | 79.5 | 30/30 | 11.1 | 0 | 30/30 | 9.3 | 30/30 |
| $d=0.1$ | 150 | 8 | 96.2 | 22/30 | 40.6 | 208.82 | 24/30 | 310.4 | 30.9 | 0/30 | >600.0 | 56.8 | 26/30 | 119.1 | 0 | 27/30 | 104.7 | 30/30 |
| $d=0.1$ | 150 | 12 | 97.3 | 24/30 | 167.0 | 15.65 | 26/30 | 245.0 | 24.4 | 0/30 | >600.0 | 15.1 | 25/30 | 161.2 | 0 | 26/30 | 138.1 | 30/30 |
| $d=0.1$ | 150 | 15 | 97.3 | 26/30 | 130.5 | 23.99 | 27/30 | 215.6 | 18.1 | 0/30 | >600.0 | 11.0 | 29/30 | 80.9 | 5 | 28/30 | 105.0 | 30/30 |
| $d=0.1$ | 150 | 20 | 96.3 | 28/30 | 49.0 | 29.55 | 28/30 | 185.0 | 20.1 | 0/30 | >600.0 | 6.0 | 30/30 | 76.2 | 11 | 30/30 | 91.7 | 30/30 |
| $d=0.1$ | 200 | 8 | 96.0 | 14/30 | 26.0 | 67.20 | 24/30 | 285.4 | 48.1 | 0/30 | >600.0 | 26.4 | 22/30 | 233.9 | 0 | 18/30 | 290.2 | 30/30 |
| $d=0.1$ | 200 | 12 | 98.1 | 0/30 | >600.0 | 55.46 | 20/30 | 340.2 | 49.1 | 0/30 | >600.0 | 11.8 | 13/30 | 411.2 | 0 | 12/30 | 425.1 | 30/30 |
| $d=0.1$ | 200 | 15 | 97.4 | 0/30 | >600.0 | 50.74 | 18/30 | 375.0 | 25.9 | 0/30 | >600.0 | 5.5 | 6/30 | 510.3 | 0 | 8/30 | 480.3 | 30/30 |
| $d=0.1$ | 200 | 20 | 97.3 | 0/30 | >600.0 | 44.36 | 16/30 | 385.0 | 29.2 | 0/30 | >600.0 | 5.9 | 12/30 | 402.1 | 0 | 15/30 | 357.7 | 30/30 |
| **$d=0.1$ Mean / Total** | — | — | **91.0** | **591/720** | **116.3** | **44.60** | **662/720** | **139.7** | **37.0** | **0/720** | **>600.0** | **50.1** | **643/720** | **94.6** | **77** | **644/720** | **93.7** | **720/720** |

---

**Table 5: Ablation results of the five key components for conflict densities $d=0.3$ and $d=0.5$**

| Density | $N$ | $M$ | Shared Bwd ZDD: Red. (%) | Opt. | Time (s) | Bucket Labeling: $t_{\mathrm{prc}}$ (ms) | Opt. | Time (s) | No Domi. ($v_1$): Prune (%) | Opt. | Time (s) | No Fixi. ($v_2$): Fixed (%) | Opt. | Time (s) | No Relax. ($v_3$): Trig. | Opt. | Time (s) | BZDD-BPC: Opt. |
|:---:|:---:|:---:|---:|:---:|---:|---:|:---:|---:|---:|:---:|---:|---:|:---:|---:|:---:|:---:|---:|:---:|
| $d=0.3$ | 40 | 8 | 76.8 | 30/30 | 4.0 | 17.63 | 30/30 | 12.8 | 20.8 | 0/30 | >600.0 | 66.8 | 30/30 | 5.4 | 0 | 30/30 | 4.3 | 30/30 |
| $d=0.3$ | 40 | 12 | 76.8 | 30/30 | 1.6 | 16.72 | 30/30 | 2.8 | 29.7 | 0/30 | >600.0 | 64.5 | 30/30 | 1.7 | 26 | 30/30 | 2.3 | 30/30 |
| $d=0.3$ | 40 | 15 | 70.0 | 30/30 | 0.6 | 3.69 | 30/30 | 0.8 | 40.2 | 9/30 | 420.1 | 59.7 | 30/30 | 0.2 | 6 | 30/30 | 0.3 | 30/30 |
| $d=0.3$ | 40 | 20 | 74.5 | 30/30 | 0.3 | 2.10 | 30/30 | 0.4 | 42.0 | 21/30 | 187.1 | 47.6 | 30/30 | 0.1 | 2 | 30/30 | 0.1 | 30/30 |
| $d=0.3$ | 60 | 8 | 78.6 | 30/30 | 0.7 | 50.33 | 30/30 | 35.4 | 28.4 | 11/30 | 380.3 | 67.4 | 30/30 | 13.1 | 0 | 30/30 | 11.7 | 30/30 |
| $d=0.3$ | 60 | 12 | 83.0 | 30/30 | 2.6 | 97.86 | 30/30 | 9.4 | 26.3 | 7/30 | 460.7 | 45.1 | 30/30 | 52.2 | 2 | 30/30 | 57.0 | 30/30 |
| $d=0.3$ | 60 | 15 | 83.5 | 30/30 | 8.8 | 73.08 | 30/30 | 36.2 | 33.0 | 6/30 | 480.2 | 61.0 | 30/30 | 13.2 | 12 | 30/30 | 14.8 | 30/30 |
| $d=0.3$ | 60 | 20 | 83.3 | 30/30 | 2.7 | 39.60 | 30/30 | 8.5 | 42.0 | 5/30 | 500.2 | 63.2 | 30/30 | 2.4 | 15 | 30/30 | 3.4 | 30/30 |
| $d=0.3$ | 80 | 8 | 91.2 | 30/30 | 30.0 | 47.15 | 30/30 | 14.2 | 33.4 | 0/30 | >600.0 | 63.0 | 30/30 | 3.4 | 0 | 30/30 | 3.5 | 30/30 |
| $d=0.3$ | 80 | 12 | 91.2 | 30/30 | 37.2 | 12.08 | 30/30 | 62.4 | 25.4 | 1/30 | 581.6 | 53.1 | 30/30 | 50.0 | 0 | 30/30 | 27.1 | 30/30 |
| $d=0.3$ | 80 | 15 | 89.4 | 30/30 | 17.5 | 32.80 | 30/30 | 34.5 | 44.1 | 0/30 | >600.0 | 77.6 | 30/30 | 12.9 | 1 | 30/30 | 10.5 | 30/30 |
| $d=0.3$ | 80 | 20 | 90.4 | 30/30 | 60.6 | 22.83 | 30/30 | 18.2 | 37.0 | 9/30 | 422.5 | 71.5 | 30/30 | 5.8 | 0 | 30/30 | 4.8 | 30/30 |
| $d=0.3$ | 100 | 8 | 93.7 | 29/30 | 26.3 | 21.81 | 30/30 | 12.0 | 37.1 | 0/30 | >600.0 | 70.5 | 30/30 | 2.5 | 0 | 30/30 | 2.4 | 30/30 |
| $d=0.3$ | 100 | 12 | 93.8 | 28/30 | 110.9 | 123.15 | 28/30 | 168.5 | 19.4 | 5/30 | 507.1 | 32.3 | 26/30 | 93.5 | 0 | 26/30 | 94.0 | 30/30 |
| $d=0.3$ | 100 | 15 | 92.5 | 30/30 | 101.5 | 61.82 | 30/30 | 118.5 | 37.5 | 0/30 | >600.0 | 59.8 | 30/30 | 16.5 | 5 | 30/30 | 16.8 | 30/30 |
| $d=0.3$ | 100 | 20 | 93.5 | 30/30 | 51.0 | 11.37 | 30/30 | 34.2 | 35.0 | 20/30 | 232.4 | 70.5 | 30/30 | 11.9 | 0 | 30/30 | 9.1 | 30/30 |
| $d=0.3$ | 150 | 8 | 95.0 | 22/30 | 19.6 | 94.83 | 28/30 | 112.5 | 28.5 | 0/30 | >600.0 | 53.6 | 30/30 | 24.1 | 0 | 30/30 | 24.4 | 30/30 |
| $d=0.3$ | 150 | 12 | 95.9 | 24/30 | 55.3 | 0.00 | 27/30 | 208.5 | 0.0 | 0/30 | >600.0 | 0.0 | 19/30 | 245.9 | 12 | 21/30 | 218.2 | 30/30 |
| $d=0.3$ | 150 | 15 | 93.3 | 26/30 | 62.6 | 51.73 | 26/30 | 175.4 | 16.2 | 0/30 | >600.0 | 10.1 | 29/30 | 65.6 | 3 | 30/30 | 46.7 | 30/30 |
| $d=0.3$ | 150 | 20 | 96.5 | 28/30 | 118.0 | 46.77 | 28/30 | 165.2 | 16.3 | 0/30 | >600.0 | 2.4 | 29/30 | 61.8 | 5 | 28/30 | 95.7 | 30/30 |
| $d=0.3$ | 200 | 8 | 97.7 | 14/30 | 34.9 | 59.74 | 28/30 | 85.4 | 27.3 | 0/30 | >600.0 | 48.0 | 30/30 | 13.6 | 0 | 30/30 | 13.1 | 30/30 |
| $d=0.3$ | 200 | 12 | 97.1 | 12/30 | 28.1 | 95.01 | 30/30 | 24.5 | 17.5 | 0/30 | >600.0 | 30.7 | 30/30 | 3.5 | 0 | 30/30 | 3.5 | 30/30 |
| $d=0.3$ | 200 | 15 | 97.2 | 11/30 | 128.1 | 191.45 | 30/30 | 38.6 | 16.8 | 0/30 | >600.0 | 30.8 | 30/30 | 7.1 | 0 | 30/30 | 7.4 | 30/30 |
| $d=0.3$ | 200 | 20 | 97.4 | 0/30 | >600.0 | 49.66 | 0/30 | >600.0 | 45.1 | 0/30 | >600.0 | 7.5 | 28/30 | 161.2 | 0 | 29/30 | 143.9 | 22/30 |
| **$d=0.3$ Mean / Total** | — | — | **88.8** | **614/720** | **62.6** | **50.97** | **675/720** | **82.5** | **29.1** | **94/720** | **523.8** | **48.2** | **701/720** | **36.2** | **89** | **704/720** | **33.9** | **712/720** |
| $d=0.5$ | 40 | 8 | 64.0 | 30/30 | 0.6 | 3.48 | 30/30 | 3.8 | 19.3 | 5/30 | 501.4 | 59.2 | 30/30 | 1.3 | 0 | 30/30 | 1.1 | 30/30 |
| $d=0.5$ | 40 | 12 | 74.9 | 30/30 | 1.0 | 1.53 | 30/30 | 0.8 | 33.4 | 17/30 | 260.1 | 68.1 | 30/30 | 0.2 | 0 | 30/30 | 0.2 | 30/30 |
| $d=0.5$ | 40 | 15 | 85.8 | 30/30 | 0.4 | 0.91 | 30/30 | 0.4 | 26.0 | 17/30 | 260.2 | 61.2 | 30/30 | 0.1 | 0 | 30/30 | 0.1 | 30/30 |
| $d=0.5$ | 40 | 20 | 68.9 | 30/30 | 0.3 | 1.11 | 30/30 | 1.1 | 27.0 | 26/30 | 83.2 | 48.0 | 30/30 | 0.1 | 0 | 30/30 | 0.1 | 30/30 |
| $d=0.5$ | 60 | 8 | 80.9 | 30/30 | 0.7 | 18.57 | 30/30 | 8.5 | 30.9 | 7/30 | 460.1 | 59.4 | 30/30 | 3.4 | 0 | 30/30 | 3.1 | 30/30 |
| $d=0.5$ | 60 | 12 | 82.5 | 30/30 | 35.3 | 17.04 | 30/30 | 38.5 | 38.8 | 4/30 | 520.0 | 52.9 | 30/30 | 13.5 | 0 | 30/30 | 12.5 | 30/30 |
| $d=0.5$ | 60 | 15 | 84.2 | 30/30 | 8.6 | 15.31 | 30/30 | 16.2 | 44.4 | 1/30 | 580.0 | 62.1 | 30/30 | 5.2 | 0 | 30/30 | 5.5 | 30/30 |
| $d=0.5$ | 60 | 20 | 82.2 | 30/30 | 0.7 | 10.05 | 30/30 | 2.5 | 41.2 | 2/30 | 560.0 | 63.3 | 30/30 | 0.9 | 0 | 30/30 | 0.8 | 30/30 |
| $d=0.5$ | 80 | 8 | 89.4 | 30/30 | 12.7 | 12.85 | 30/30 | 5.2 | 43.8 | 1/30 | 580.1 | 63.0 | 30/30 | 1.4 | 0 | 30/30 | 1.5 | 30/30 |
| $d=0.5$ | 80 | 12 | 90.7 | 30/30 | 7.9 | 14.44 | 30/30 | 2.8 | 46.3 | 0/30 | >600.0 | 70.8 | 30/30 | 0.7 | 0 | 30/30 | 0.7 | 30/30 |
| $d=0.5$ | 80 | 15 | 90.1 | 30/30 | 7.7 | 5.96 | 30/30 | 68.4 | 34.5 | 0/30 | >600.0 | 37.9 | 30/30 | 26.8 | 0 | 30/30 | 23.6 | 30/30 |
| $d=0.5$ | 80 | 20 | 90.0 | 30/30 | 12.6 | 12.32 | 30/30 | 14.8 | 44.1 | 0/30 | >600.0 | 52.5 | 30/30 | 6.1 | 0 | 30/30 | 5.9 | 30/30 |
| $d=0.5$ | 100 | 8 | 92.9 | 29/30 | 11.8 | 14.87 | 30/30 | 4.8 | 43.3 | 0/30 | >600.0 | 70.5 | 30/30 | 1.4 | 0 | 30/30 | 1.5 | 30/30 |
| $d=0.5$ | 100 | 12 | 93.5 | 28/30 | 5.3 | 35.17 | 30/30 | 9.8 | 32.4 | 0/30 | >600.0 | 46.7 | 30/30 | 3.6 | 0 | 30/30 | 3.2 | 30/30 |
| $d=0.5$ | 100 | 15 | 93.0 | 30/30 | 3.6 | 49.20 | 30/30 | 6.5 | 27.4 | 5/30 | 500.3 | 33.6 | 30/30 | 2.1 | 0 | 30/30 | 2.0 | 30/30 |
| $d=0.5$ | 100 | 20 | 93.3 | 30/30 | 19.4 | 39.45 | 30/30 | 65.4 | 29.7 | 2/30 | 560.0 | 36.6 | 30/30 | 5.5 | 9 | 25/30 | 112.5 | 30/30 |
| $d=0.5$ | 150 | 8 | 96.5 | 22/30 | 55.3 | 0.00 | 30/30 | 58.2 | 0.0 | 0/30 | >600.0 | 0.0 | 30/30 | 21.2 | 4 | 27/30 | 86.4 | 30/30 |
| $d=0.5$ | 150 | 12 | 96.7 | 24/30 | 35.7 | 46.70 | 30/30 | 5.2 | 27.3 | 0/30 | >600.0 | 49.1 | 30/30 | 1.4 | 6 | 25/30 | 118.2 | 30/30 |
| $d=0.5$ | 150 | 15 | 96.4 | 26/30 | 79.5 | 43.68 | 30/30 | 2.4 | 39.0 | 0/30 | >600.0 | 63.0 | 30/30 | 0.6 | 5 | 26/30 | 94.6 | 30/30 |
| $d=0.5$ | 150 | 20 | 96.4 | 28/30 | 39.6 | 34.47 | 30/30 | 165.2 | 47.2 | 0/30 | >600.0 | 60.7 | 30/30 | 1.7 | 3 | 28/30 | 58.3 | 30/30 |
| $d=0.5$ | 200 | 8 | 97.4 | 14/30 | 26.5 | 209.07 | 26/30 | 125.0 | 46.0 | 0/30 | >600.0 | 80.0 | 30/30 | 13.5 | 8 | 23/30 | 178.5 | 30/30 |
| $d=0.5$ | 200 | 12 | 97.6 | 12/30 | 377.6 | 38.51 | 30/30 | 2.1 | 6.5 | 0/30 | >600.0 | 12.5 | 30/30 | 0.5 | 11 | 20/30 | 235.4 | 30/30 |
| $d=0.5$ | 200 | 15 | 96.1 | 11/30 | 49.6 | 66.67 | 30/30 | 3.2 | 9.8 | 0/30 | >600.0 | 18.5 | 30/30 | 0.8 | 14 | 17/30 | 294.1 | 30/30 |
| $d=0.5$ | 200 | 20 | 97.3 | 15/30 | 49.6 | 35.16 | 24/30 | 185.2 | 46.3 | 0/30 | >600.0 | 80.9 | 30/30 | 1.3 | 12 | 19/30 | 252.8 | 30/30 |
| **$d=0.5$ Mean / Total** | — | — | **88.8** | **629/720** | **35.1** | **30.27** | **710/720** | **33.2** | **32.7** | **87/720** | **527.7** | **52.1** | **720/720** | **4.7** | **72** | **660/720** | **62.2** | **720/720** |

The ablation results establish the following ranking of component contributions:

1. **Multi-attribute label dominance is the most critical pruning component.** Deactivating dominance pruning ($v_1$) causes immediate combinatorial explosion in the pricing subproblem, dropping the solve count from 2,872 down to 181 across the entire benchmark. On instances with $N \ge 80$, $v_1$ fails to solve almost any instance within the 600.0-second time limit ($>600.0$ s).
2. **Shared backward ZDD pricing is essential for scalable pricing over multiple representatives.** Sharing suffix subgraphs across all machine representatives reduces diagram node counts by an average of 88.8%--93.9% (and up to 99.1% on $N=200, M=20$). Without suffix sharing, pricing individual forward diagrams leads to 455 timeouts on large configurations ($N \ge 150$), dropping the solved count to 2,425/2,880.
3. **Bucket-graph labeling prevents pairwise dominance bottlenecks.** Grouping labels into discrete weight intervals restricts dominance checks locally and maintains mean pricing call times at 21.7--51.0 ms. Pure labeling without the bucket graph encounters severe state slowdowns on large bottleneck configurations, resulting in 182 timeouts and reducing the solved count to 2,698/2,880.
4. **Adaptive schedule relaxation prevents layer-width state explosion.** Width capping selectively triggers on dense layers (77 instances at $d=0.1$, 89 at $d=0.3$, and 72 at $d=0.5$). Constructing unconstrained exact diagrams ($v_3$) causes severe layer expansion on dense graphs, leading to 211 timeouts on $N \ge 100$ and reducing the solved count to 2,669/2,880.
5. **Dual-bound variable fixing accelerates master convergence.** Dual picking and Lagrangian fixing successfully fix an average of 48.2%--68.2% of candidate representative assignments, significantly reducing master problem column generation iterations and preventing 161 timeouts.

---

## Computational environment

The computational experiments reported in the manuscript were conducted under the following environment:

- **Processor**: Intel Core Ultra 9 275HX (2.70 GHz base frequency, 24 cores / 24 threads);
- **Memory**: 32 GB RAM;
- **Operating System**: Windows 11;
- **Programming Language**: C++ (compiled with MSVC / ISO C++17);
- **LP / MILP Solver**: Gurobi Optimizer 11.0;
- **Stopping Criteria**: 600.0 seconds per instance (unsolved instances count as 600.0 s in averages).

}
```

Please update the bibliographic details with the final journal volume, issue, page numbers, and DOI upon publication.
