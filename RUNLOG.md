# RUNLOG — Mpox Nextstrain Training Build

Repository: `mpx_nextstrain/`  
Date: 2025-09-08 (Africa/Addis_Ababa)

> Compact, shareable record of what was run, what worked, and the resulting artifacts. Import this file into your GitHub repo.

---

## 0) Inputs & Layout

**Repo layout** (relevant paths)
```
mpx_nextstrain/
├─ data/
│  ├─ genomes/                      # input FASTA (one record per sample)
│  ├─ nextstrain_metadata.tsv       # >= strain, date (YYYY-MM-DD), country
│  └─ exclude.txt                   # built during QC & subsampling
├─ ref/
│  └─ mpxv_ref.gb                   # annotated GenBank
├─ builds/mpx_africa/               # all outputs
├─ config/                          # colors.tsv (+ optional UI config)
├─ narratives/                      # Nextstrain narrative(s)
└─ scripts/                         # stepwise scripts
```

**Environment**
- Conda env: `mpox_ns`
- Tools: `nextclade`, `augur`, `auspice`, `mafft`, `fasttree`
- Nextclade dataset cached under: `env/nextclade_datasets/mpox`

---

## 1) QC with Nextclade

**Script:** `qc_nextcalde.sh`  
**Status:** ✅ completed after CLI fixes  
**Notes:** Modern Nextclade uses `--output-all` and proper dataset names (e.g., `nextstrain/mpox/all-clades`).  
**Key artifacts:**
- `builds/mpx_africa/qc/nextclade.csv` (semicolon-delimited)
- `builds/mpx_africa/qc/nextclade.json`
- `data/exclude.txt` (IDs failing QC thresholds: missing>3000, private>2000, qc<0.8)

---

## 2) Filter → Sanitize → Subsample → Align

**Script:** `run_filter_align.sh`  
**Status:** ✅ completed  
**What it did:**
- Concatenated FASTAs to avoid “too many open files”
- Filtered by metadata & `data/exclude.txt`
- **Sanitized** non-IUPAC characters (`<`, `>`, `=` → `N`) prior to MAFFT
- **Balanced subsampling** for teaching (caps):  
  - ≤1 per `(country, year, month)`  
  - ≤5 per `(country, clade)` if `clade` exists  
  - ≤20 per `country`  
- Added all non-kept IDs to `data/exclude.txt`
- MAFFT with `--nthreads 12`

**Key artifacts:**
- `builds/mpx_africa/filtered/filtered.clean.fasta`
- `builds/mpx_africa/sub_keep/train.fasta`
- `builds/mpx_africa/sub_keep/train.tsv`
- `builds/mpx_africa/align/aligned.fasta`

---

## 3) Tree Build

**Script:** `build_tree.sh`  
**Status:** ✅ completed  
**Method:** FastTree (approximately ML)  
**Artifact:** `builds/mpx_africa/tree/tree_raw.nwk`

---

## 4) Time-Scaling / Refine

**Script:** `refine.sh`  
**Status:** ✅ completed  
**Options:** `--timetree --date-confidence --coalescent skyline`  
**Artifacts:**
- `builds/mpx_africa/tree/tree_time.nwk`
- `builds/mpx_africa/tree/branch_lengths.json`

---

## 5) Mutations

**Script:** `gen_mutations.sh`  
**Status:** ⚠️ partial  
**Outcome:**
- ✅ Nucleotide mutations: `builds/mpx_africa/tree/nuc_muts.json`
- ⏭️ Amino-acid mutations: skipped (Augur version expects internal-node ancestral sequences FASTA)

> If needed later: reconstruct via `augur reconstruct-sequences --tree tree_time.nwk --mutations nuc_muts.json --internal-nodes --output ancestral_nt.fasta`, then `augur translate ...`

---

## 6) Traits (Discrete Geography)

**Script:** `traits.sh`  
**Status:** ✅ completed (warning: only one state found = OK for demo)  
**Artifact:** `builds/mpx_africa/tree/traits.json`

---

## 7) Frequencies (Optional)

**Script:** `frequencies.sh`  
**Status:** ⏭️ skipped  
**Reason:** Reference tip (`NC_003310.1`) not in metadata → Augur `frequencies` failed on this env.  
**If needed:** prune to metadata tips (`augur filter --tree ... --metadata ... --output-tree tree_time.pruned.nwk`) then run `augur frequencies --method kde ...`.

---

## 8) Export to Auspice

**Script:** `export.sh`  
**Status:** ✅ completed; schema validation OK  
**Artifact:** `builds/mpx_africa/auspice/mpx_training.json`

---

## 9) Serve (Auspice) + Narrative

**Scripts:** `view_data.sh`, `make_narrative.sh`  
**Status:** ✅ working after removing `--port` flag and fixing prefixes

**Open locally**
- Dataset: `http://localhost:4000/mpx_training`  
- Narrative: `http://localhost:4000/narratives/mpx_training`

**Prefix alias (optional):** also served as `/mpx/training` by copying/aliasing:
- `builds/mpx_africa/auspice/mpx/training.json`
- `narratives/mpx/training.md`

---

## 10) Config (Colors & Meta)

**Script:** `make_config.sh` (Augur-compatible, no `--config`)  
**Status:** ✅ completed  
**What it did:**  
- Auto-generated `config/colors.tsv` (country → color) from metadata  
- Re-exported with `--colors`  
- Post-edited dataset JSON meta (`title`, `maintainers`, `description`) via a tiny Python step

**Artifacts:**
- `config/colors.tsv`
- Updated `builds/mpx_africa/auspice/mpx_training.json`

---

## Validation — quick status check

Add as `scripts/status_check.sh`:
```bash
#!/usr/bin/env bash
set -e
need=(
  builds/mpx_africa/qc/nextclade.csv
  builds/mpx_africa/align/aligned.fasta
  builds/mpx_africa/tree/tree_raw.nwk
  builds/mpx_africa/tree/tree_time.nwk
  builds/mpx_africa/tree/branch_lengths.json
  builds/mpx_africa/tree/nuc_muts.json
  builds/mpx_africa/tree/traits.json
  builds/mpx_africa/auspice/mpx_training.json
)
miss=0; for f in "${need[@]}"; do [[ -f "$f" ]] && echo "✅ $f" || { echo "❌ $f"; miss=1; }; done
[[ -f config/colors.tsv ]] && echo "✅ config/colors.tsv" || echo "⚠️  config/colors.tsv (optional)"
exit $miss
```

---

## Troubleshooting encountered (and fixes)

- **Nextclade CLI change:** `--output-dir` removed → use `--output-all` + `--output-selection`  
- **Dataset name:** use `nextstrain/mpox/all-clades` (not just `mpox`)  
- **Too many open files:** concatenate inputs; or `ulimit -n 4096`  
- **MAFFT error on `<`, `>`, `=`:** sanitize sequences to `N` before align  
- **Augur align flags:** don’t pass `--metadata/--exclude` to `align`; do them in `filter`  
- **AA mutations:** require ancestral NT sequences FASTA (version-dependent)  
- **Frequencies  KeyError:** reference tip lacks metadata → prune tree or add reference row  
- **Auspice `--port` unsupported:** omit; default port 4000  
- **Auspice 404:** mismatch of prefixes (`mpx_training` vs `mpx/training`) → add alias files or adjust narrative front-matter

---

## Share & Reproducibility

- Pin dataset (optional): `nextclade dataset get --name nextstrain/mpox/all-clades --tag <tag>`  
- Export env: `conda env export > env/environment.lock.yml`  
- Keep this `RUNLOG.md` updated per run (date, key warnings)  

