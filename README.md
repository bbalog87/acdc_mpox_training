# Mpox Nextstrain Training Workflow

This repository provides a **step-by-step, hands-on training workflow** for building a Nextstrain phylogenetic analysis of Monkeypox (Mpox) genomes. It is designed for capacity building in genomic epidemiology and public health.

---

## 🔑 Key Features
- Ready-to-use **Bash pipeline** (`scripts/run_nextstrain_mpx.sh`)
- Support for **multi-country Mpox datasets**
- Integration of **Nextclade QC**, **MAFFT alignment**, **FastTree/IQ-TREE**, and **Auspice visualization**
- Clean repo structure for **teaching and reproducibility**
- Compatible with **conda/mamba environment** (`env/mpox_ns.yml`)

---

## 📂 Repository Layout
- `data/` → input genomes, metadata, and exclusion lists  
- `config/` → Auspice configuration and color schemes  
- `scripts/` → pipeline scripts and helpers  
- `builds/` → auto-generated results (ignored in git)  
- `docs/` → slides, diagrams, teaching notes  
- `env/` → reproducible conda environment  
### Project  Directory Structure
```
mpx-nextstrain-training/
├── README.md
├── LICENSE
├── data/
│   ├── genomes/              # input FASTA sequences
│   ├── metadata.tsv          # required sample metadata
│   └── exclude.txt           # list of excluded samples
├── config/
│   ├── colors.tsv            # color scheme for Auspice
│   └── auspice_config.json   # Auspice display config
├── builds/                   # output directory (ignored in git)
│   └── mpx_africa/
│       ├── qc/               # Nextclade outputs
│       ├── align/            # alignment files
│       ├── tree/             # tree files & node data
│       ├── auspice/          # JSON for visualization
│       └── report/           # PNGs, methods, logs
├── scripts/
│   ├── run_nextstrain_mpx.sh # end-to-end Bash pipeline
│   └── utils/                # helper scripts (QC filters, env setup)
├── docs/
│   ├── slides/               # training slides (PDF/PowerPoint)
│   ├── workflow_diagram.png  # visual schematic
│   └── notes.md              # teaching notes
├── env/
│   └── environment.yml       # conda/mamba environment
└── .gitignore

```

---


## Getting Started

### Step 0 — Build the folder layout
First, set up the project directory structure by running the script below.  
This will create all necessary folders (`data/`, `config/`, `builds/`, `env/`) for the workflow.

```bash
bash scripts/create_mpx_nextstrain_structure.sh
```

### Step 1 — Create environment
```bash
mamba env create -f env/mpox_ns.yml
mamba activate mpox_ns
```

# 2. Run pipeline
bash scripts/run_nextstrain_mpx.sh

# 3. Visualize results
auspice view --datasetDir builds/mpx_africa/auspice
