# Evaluation Guide

The scripts below evaluate segmentation, boundaries, and polygon topology. Run commands from the repository root, where `TCPCN.py` is located.

Uppercase values in commands are placeholders for your own paths or names. Each script's input requirements are described separately.

## Dependencies

After installing the dependencies in the [main README](../readme.md), install:

```bash
pip install shapely pandas tifffile labelme
```

Segmentation metric generation and the two boundary utilities also require `cellpose4`. Verify the imports with:

```bash
python -c "from cellpose4 import metrics, utils"
```

## Segmentation Metrics: `save_to_npy.py`

Compute AP, TP, FP, and FN at IoU thresholds `0.5`, `0.75`, and `0.9`.

- `--gt_root`: directory directly containing GT LabelMe JSON files with `imageHeight`, `imageWidth`, and cell annotations in `shapes`.
- `--pred_root`: directory containing one subdirectory per method. Each method directory must directly contain prediction JSON or TIFF instance masks; nested directories are not scanned.
- Predictions must match the GT image dimensions and coordinates. A prediction filename must contain its GT basename, with one matching prediction per image and method. Raster masks use background `0` and a distinct positive integer label per cell.

```bash
python -m evaluation.save_to_npy \
  --gt_root "GT_JSON_DIR" \
  --pred_root "PREDICTION_ROOT" \
  --output_dir "METRICS_OUTPUT_DIR" \
  --overlap_threshold 0.3
```

`--overlap_threshold` filters predictions by the fraction of their area overlapping a single GT instance (default `0.3`).

**Output:** `<method>_<dataset>.npy` in `--output_dir`, containing metrics, filenames, and evaluated masks. The method name comes from each method directory; the dataset name comes from the final component of `--pred_root`.

## Segmentation Plots: `segmentation/violin.py`

Plot AP and error distributions from NPY evaluation files.

- `--result_dir`: directory directly containing the NPY files.
- `--datasets` and `--algs`: dataset and method names matching `<method>_<dataset>.npy`.
- `--alg_names`: display labels, in the same order as `--algs`.

```bash
python -m evaluation.segmentation.violin \
  --result_dir "NPY_DIR" \
  --output_dir "FIGURE_OUTPUT_DIR" \
  --datasets "DATASET_NAME" \
  --algs "METHOD_NAME" \
  --alg_names "DISPLAY_NAME"
```

Pass multiple names after these options to compare datasets or methods.

**Output:** `result_segmentation.pdf` in `--output_dir`, using metrics at IoU `0.5`.

## GT Cell Diameters: `boundary/mean_diameter.py`

Summarize per-image cell diameter estimates from GT masks.

- `--gt_root`: parent directory of the dataset directories; JSON files are read directly from `GT_ROOT/<dataset>/`.
- `--datasets`: names of the dataset directories to process. GT uses LabelMe JSON with image dimensions and cell annotations.

```bash
python -m evaluation.boundary.mean_diameter \
  --gt_root "GT_ROOT" \
  --datasets "DATASET_NAME"
```

**Output:** mean and median of per-image diameter estimates, in pixels, printed to the terminal. This utility does not change the boundary-evaluation configuration.

## Boundary Scores: `boundary/BF1S.py`

Evaluate boundary precision, recall, and F-score.

**Configuration:** the script currently defines Sansha-5 and WO115-2 in its `datasets` list. Before using other data, replace that list in [`boundary/BF1S.py`](boundary/BF1S.py) with the datasets to evaluate. Each entry specifies `name` (dataset directory name), `avg_diam` (diameter in pixels used for the plot's horizontal axis), `scales` (boundary tolerance scales), and `prefix` (figure panel label). The plotted pixel widths are `scales * avg_diam`; choose the diameter and scales for your data. The diameter utility above can provide diameter estimates.

- `--gt_root`: GT JSON files are read directly from `GT_ROOT/<name>/`.
- `--pred_root`: prediction files are read directly from `PREDICTION_ROOT/<name>/<method>/`.
- `--algs`: method directory names. Predictions may be LabelMe JSON or raster instance masks, matched by GT basename and image dimensions. Method subdirectories are not scanned recursively.

```bash
python -m evaluation.boundary.BF1S \
  --gt_root "GT_ROOT" \
  --pred_root "PREDICTION_ROOT" \
  --output_dir "BOUNDARY_OUTPUT_DIR" \
  --algs "METHOD_NAME"
```

The script evaluates the datasets in its configuration and filters predictions with an overlap threshold of `0.5`.

**Output:** `boundary_evaluation_<dataset>.txt` and `boundary_score_<dataset>.pdf` in `--output_dir`. The text report records F-scores; the figure shows all three boundary metrics.

## Neighborhood Topology: `topology/NFCS.py`

Compute the Neighborhood Feature Consistency Score (NFCS) from cell polygons.

- `--gt_dir`: directory directly containing GT polygon JSON files.
- `--pred_dir`: directory directly containing prediction polygon JSON files; this can be a TCPCN output's `json/` directory.
- Use corresponding image coordinates and one prediction file per GT basename. The two directories can be located independently; no dataset/method parent structure is required.

```bash
python -m evaluation.topology.NFCS \
  --gt_dir "GT_JSON_DIR" \
  --pred_dir "PREDICTION_JSON_DIR"
```

**Output:** per-image scores and their mean, printed to the terminal. The batch command uses ROI-overlap and structural-score thresholds of `0.5`.

## Polygon Vertex Counts: `topology/polygon_analysis.py`

Compare GT and predicted polygon vertex-count distributions.

- `--gt_root`: GT JSON files are read from `GT_ROOT/<dataset>/`.
- `--pred_root`: prediction JSON files are read from `PREDICTION_ROOT/<dataset>/<method>/`.
- `--datasets` and `--algs`: dataset and method directory names to select. JSON files are scanned recursively within the selected directories and matched by GT basename.

```bash
python -m evaluation.topology.polygon_analysis \
  --gt_root "GT_ROOT" \
  --pred_root "PREDICTION_ROOT" \
  --datasets "DATASET_NAME" \
  --algs "METHOD_NAME" \
  --roi_threshold 0.5
```

`--roi_threshold` is the minimum fraction of a predicted polygon's area inside the union of GT polygons (default `0.5`).

**Output:** `vertex_analysis_<dataset>.csv` directly under `--pred_root`, with counts and percentages for GT and predictions. The script exports CSV summaries only and has no separate output-directory argument.
