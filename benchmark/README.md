# Benchmark Data and Results

This directory describes the datasets and released results used to evaluate TCPCN. For evaluation commands, see the [evaluation guide](../evaluation/README.md).

## Downloads

| Resource | Download | Contents |
| --- | --- | --- |
| Dataset | `Porphyra haitanensis.zip` on [Zenodo](https://zenodo.org/records/20301518) (DOI: [10.5281/zenodo.20301518](https://doi.org/10.5281/zenodo.20301518)) | Original microscopy images and LabelMe GT annotations. |
| Results | `tcpcn_benchmark_results_v1.0.0.zip` from the [v1.0.0 release](https://github.com/Vicky-quq/TCPCN/releases/tag/v1.0.0) | Instance masks, reconstructed polygons/meshes, visualizations, and evaluation files. |

The [dataset record](https://zenodo.org/records/20301518) describes the two subsets:

| Dataset | Images | Annotated cells |
| --- | --- | --- |
| Sansha-5 | 16 | 1,613 |
| WO115-2 | 16 | 1,746 |

## Extraction and Layout

- Place the extracted `Sansha-5/` and `WO115-2/` dataset directories inside `benchmark/Dataset/`. Each contains original images and GT JSON files.
- Place the results archive's contents inside `benchmark/Result/`, with `Sansha-5/`, `WO115-2/`, and the evaluation files directly at that level. If extraction creates a `tcpcn_benchmark_results_v1.0.0/` wrapper, place its contents in `benchmark/Result/`.

The resulting layout is:

```text
benchmark/
|-- README.md
|-- Dataset/
|   |-- Sansha-5/                  # Original images and GT JSON
|   `-- WO115-2/                   # Original images and GT JSON
`-- Result/
    |-- Sansha-5/
    |   |-- cellposesam/           # Instance masks
    |   |-- cellposesam_ours/
    |   |   |-- json/              # LabelMe polygons
    |   |   |-- ply/               # ASCII PLY meshes
    |   |   `-- vis/               # Pipeline/junction images
    |   |-- cyto3/                 # Instance masks
    |   `-- cyto3_ours/             # Same json/ply/vis layout
    |-- WO115-2/                   # Same four method directories
    |-- *.npy
    |-- *.txt
    |-- *.csv
    `-- *.pdf
```

Keep dataset and result filenames unchanged so that the evaluation scripts can match predictions to GT.

## Released Results

`cellposesam` and `cyto3` contain the original segmentation masks. The corresponding `_ours` directories contain their TCPCN reconstructions. For each dataset, each original method has 16 masks; each `_ours` method has 16 JSON files, 16 PLY files, and 32 visualization PNGs (one pipeline image and one junction image per sample).

The following evaluation files are stored directly in `benchmark/Result/`:

| Files | Count | Description |
| --- | --- | --- |
| `<method>_<dataset>.npy` | 8 | Segmentation evaluation arrays for all four methods on both datasets. |
| `boundary_evaluation_<dataset>.txt` | 2 | Boundary evaluation reports. |
| `boundary_score_<dataset>.pdf` | 2 | Boundary precision/recall/F-score figures. |
| `nfcs_<method>_<dataset>.txt` | 4 | NFCS reports for the two `_ours` methods on both datasets. |
| `vertex_analysis_<dataset>.csv` | 2 | Polygon vertex-count summaries. |
| `result_segmentation.pdf` | 1 | Combined segmentation summary figure. |

The [evaluation guide](../evaluation/README.md) explains how to prepare these files as inputs and recompute the metrics. The [main README](../readme.md) explains how to generate new TCPCN reconstructions.
