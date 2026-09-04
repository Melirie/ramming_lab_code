# Does zero-shot scGPT beat a standard integration pipeline on SLE PBMCs?

Independent benchmarks report that, used naively, single-cell foundation models
(scGPT, Geneformer, UCE) often fail to beat much simpler pipelines. This repo
tests that claim on a clinically relevant autoimmune dataset: SLE and healthy
control PBMCs from **Perez et al. (2022), *Science*** (~1.2M cells, ~260
donors; downloaded pre-annotated from CZI CELLxGENE).

Two cell representations are built from a ~50k-cell donor-balanced subsample
(≈200 cells/donor) and compared on **cross-donor cell type label transfer**:

| Representation | What it is |
|---|---|
| `X_pca` | QC → normalize → log1p → HVG → PCA (no batch correction; ablation only) |
| `X_scVI` | Classical baseline — the PCA/HVG pipeline above + scVI batch integration |
| `X_scGPT` | Zero-shot embedding from a pretrained scGPT model (no fine-tuning) |


## Repository structure

| File | Purpose |
|---|---|
| `00_load_format_subset.ipynb` | Download the CELLxGene `.h5ad` + scGPT model weights, subsample to ~200 cells/donor, sanity-check the matrix. |
| `01_standard_sc_workflow.ipynb` | QC, normalization, HVG, PCA, scVI batch integration (`batch_key="library_uuid"`). Writes `adata_integrated.h5ad`. |
| `02_scgpt2_google_colab.ipynb` | Run on Google Colab (GPU). Installs a pinned scGPT environment and computes zero-shot scGPT embeddings for `adata_integrated.h5ad` → `adata_embedded.h5ad`. |
| `03_inspect_embedded_scgpt.ipynb` | Evaluation: UMAPs per representation, GroupKFold (by donor) kNN label transfer, macro-F1/accuracy, per-class F1, confusion matrices. |
| `plots/` | Saved output figures (`confusion_matrices.png`, `embedding_f1_barplot.png`). |

Notebooks are numbered in the order they're meant to run. `02` is separate
because scGPT inference needed Colab's GPU; everything else runs locally.
