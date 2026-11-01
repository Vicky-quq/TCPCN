# TCPCN

TCPCN is a polygon topology reconstruction and evaluation toolkit for cell instance masks. The main pipeline is `TCPCN.py` and exports polygon meshes as PLY and LabelMe JSON files; scripts under `evaluation/` are evaluation utilities for segmentation, boundary, and topology analysis.

## Installation

Python 3.9+ is recommended. Create an environment and install the main pipeline dependencies:

```bash
conda create -n tcpcn python=3.10
conda activate tcpcn
pip install numpy opencv-python matplotlib networkx scikit-image scipy
```

Additional dependencies for evaluation are listed in the [evaluation guide](evaluation/README.md#dependencies).

## Quick Start

Run all commands from the repository root, where `TCPCN.py` is located. Prepare a directory of instance masks:

```text
data/
└── masks/
    ├── sample01_masks.tif
    └── sample02_masks.tif
```

Convert the masks to polygons:

```bash
python TCPCN.py --mask_dir data/masks --output_dir results
```

This saves PLY meshes in `results/ply/` and LabelMe JSON files in `results/json/`. Convexity correction and four-way junction splitting are both disabled by default.

To also generate visualizations, place the corresponding original images in `data/images/` and run:

```bash
python TCPCN.py --mask_dir data/masks --output_dir results --img_dir data/images --vis_dir results/vis
```

For example, `data/images/sample01.tif` pairs with `data/masks/sample01_masks.tif`.

## Input Requirements

- **Masks:** single-channel 2D images with background `0` and a distinct positive integer label for each cell. Binary masks are separated into instances by connected components; touching cells therefore need distinct labels.
- **Formats:** `.tif`, `.tiff`, `.png`, and `.jpg` are recognized. Use TIFF or PNG to preserve integer mask labels.
- **Directories:** place files directly in the input directories; subdirectories are not scanned.
- **Optional original images:** keep images and masks in separate directories with matching pixel dimensions. Each image basename must occur in exactly one mask filename, such as `sample01.tif` and `sample01_masks.tif`.

Without `--img_dir`, all supported mask files are processed. With `--img_dir`, processing follows the original images and uses the first mask filename containing each image basename; images without a matching mask are skipped.

## Command-Line Arguments

| Argument | Default | Description |
| --- | --- | --- |
| `-m`, `--mask_dir` | Required | Input directory of instance masks. |
| `-o`, `--output_dir` | Required | Output root directory for `ply/` and `json/`. |
| `-i`, `--img_dir` | Omitted | Original image directory; enables image-based mask pairing. |
| `-v`, `--vis_dir` | Omitted | Visualization output directory; requires `--img_dir`. |
| `--save_ply` | `1` | Save PLY meshes. |
| `--save_json` | `1` | Save LabelMe JSON polygons. |
| `--fix_convexity` | `0` | Apply convexity correction to reconstructed polygons. |
| `--split_4way` | `0` | Split four-way junctions into three-way junctions. |
| `--k_size` | `51` | Morphological kernel size in pixels for skeleton preprocessing (`51` means a 51 x 51 kernel). |

For the four switches, `1` enables the option and `0` disables it. At least one of `--save_ply` and `--save_json` must be enabled.

Convexity correction and four-way junction splitting are both disabled by default. Use `--fix_convexity 1` to enable convexity correction, and `--split_4way 1` to enable four-way junction splitting. Either option can be enabled on its own.

The default `--k_size 51` uses a 51 × 51 pixel kernel and is the setting used for the datasets in this study. For other datasets, adjust the kernel size according to cell size and spacing in pixels, and inspect the resulting skeletons and polygons to assess the reconstruction.

Use `python TCPCN.py --help` to view the command-line help.

## Outputs

For the mask-only quick-start command, the output for `sample01_masks.tif` is:

```text
results/
├── json/
│   └── sample01_masks.json
└── ply/
    └── sample01_masks.ply
```

With `--img_dir data/images --vis_dir results/vis`, outputs use the original image basename instead:

```text
results/
├── json/
│   └── sample01.json
├── ply/
│   └── sample01.ply
└── vis/
    ├── sample01_pipeline.png
    └── sample01_junctions.png
```

| Output | Contents |
| --- | --- |
| LabelMe JSON | Polygon annotations in image coordinates: origin at the upper left, positive Y downward. |
| ASCII PLY | Planar vertex-and-face mesh: origin at the lower left, positive Y upward, `z = 0`; faces are counterclockwise and share vertex indices. |
| Pipeline visualization | Mask, detected junctions, skeleton/graph, and polygons overlaid on the original image. |
| Junction visualization | Polygon edges with degree-2 junctions in yellow and degree-4 junctions in red. |

## Evaluation and Benchmark

- [Evaluation guide](evaluation/README.md): dependencies, data preparation, segmentation metrics, boundary scores, and topology analysis.
- [Benchmark guide](benchmark/README.md): dataset downloads, released results, and commands for reproducing the reported evaluation.

Evaluation and ground-truth annotations are optional when processing your own masks.

## Scope and Limitations

TCPCN is designed for the topological post-processing of static two-dimensional (2D) cell instance segmentation masks. It generally provides more stable reconstructions when individual cell contours within a tissue can be reasonably approximated by low-complexity polygons with relatively few sides and exhibit limited local concavity. For cells with highly curved contours, deep local concavities, or highly irregular shapes, automated reconstruction performance may be limited, and users may need to inspect and manually correct the results. For low-quality masks containing substantial noise, incomplete boundaries, or segmentation errors, improving the segmentation or using complementary tools is recommended. In addition, TCPCN does not perform cross-frame cell tracking or temporal-consistency analysis; dynamic datasets therefore require appropriate temporal analysis tools.
