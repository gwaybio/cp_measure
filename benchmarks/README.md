# Benchmarks — cp_measure vs CellProfiler

This directory contains scripts that compare `cp_measure` output against
CellProfiler ground-truth measurements on the same images, computing
Pearson r and R² per feature.

## What the benchmark shows

`benchmark.py` loads real Cell Painting images (5 channels) and pre-computed
CellProfiler measurements, runs `cp_measure.featurizer.featurize()` on the same
images, then reports per-feature correlation statistics.

## Data

The benchmark data is published on Zenodo:

| Dataset | DOI | Contents |
|---------|-----|----------|
| Images | [10.5281/zenodo.18809998](https://doi.org/10.5281/zenodo.18809998) | Raw Cell Painting TIFFs |
| Masks | [10.5281/zenodo.18810081](https://doi.org/10.5281/zenodo.18810081) | Segmentation masks (.npz) |
| CellProfiler reference | [10.5281/zenodo.18810082](https://doi.org/10.5281/zenodo.18810082) | `cellprofiler_data.parquet` |

> **Note:** `benchmark.py` currently hardcodes a local datastore path
> (`/datastore/alan/cp_measure/benchmark_upload/`). To run it yourself,
> download the three Zenodo datasets above into a single directory and pass
> that path via `BENCH_DIR` at the top of the script, or update the script
> to accept it as a command-line argument.

## Running the benchmark

```bash
# 1. Download Zenodo data to a local directory, e.g. ~/cp_measure_data/
# 2. Edit BENCH_DIR in benchmark.py, then:
uv run python benchmarks/benchmark.py --limit 10   # quick smoke test (10 pairs)
uv run python benchmarks/benchmark.py              # full run
```

Output files written to `benchmarks/`:
- `cpm_cellprofiler_joint.parquet` — per-cell median values side-by-side
- `benchmark_summary.parquet` — per-feature Pearson r and R²

## Interpreting results

- **R² > 0.95**: strong equivalence — safe to treat as interchangeable
- **R² 0.80–0.95**: reasonable agreement — use with awareness
- **R² < 0.80**: meaningful divergence — investigate before relying on feature

## Feature name mapping

See [`../schema/cellprofiler_mapping.json`](../schema/cellprofiler_mapping.json)
for the correspondence between `cp_measure` feature names and CellProfiler 4.x
feature names.

Key structural differences:
- **Shape features**: `Area` → `AreaShape_Area` (prefix added, no channel)
- **Intensity**: `Intensity_MeanIntensity` → `Intensity_MeanIntensity_{image}` (channel appended)
- **Texture**: `AngularSecondMoment_3_00_256` → `Texture_AngularSecondMoment_{image}_3_00_256` (prefix added, channel inserted mid-string)
- **Granularity**: `Granularity_1` → `Granularity_1_{image}` (channel appended)
- **RadialDistribution**: `RadialDistribution_FracAtD_1of4` → `RadialDistribution_FracAtD_{image}_1of4` (channel inserted mid-string)
