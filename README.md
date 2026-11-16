# Mineral data representation

An initial model combining Raman spectra, mineral properties and Kate's redox assignments. A CNN and two MLPs each produce 32 values.
These and input-availability flags form a 64-dimensional embedding.

The [notebook](notebooks/Mineral_Fusion_Walkthrough.ipynb) trains a composition model, checks a
specimen holdout, plots reconstructions and interactive 3D PCA, UMAP and t-SNE views,
and lists example nearest neighbors.
It generates the current results on every run.

## Setup

Use Python 3.12. From the repository root on Windows:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m ipykernel install --prefix .venv --name geofm --display-name "Python (GeoFM)"
.\.venv\Scripts\python.exe -m jupyterlab
```

macOS/Linux:

```bash
python3.12 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python -m ipykernel install --prefix .venv --name geofm --display-name "Python (GeoFM)"
.venv/bin/python -m jupyterlab
```

Open the notebook, select **Python (GeoFM)**, and run all cells. Training uses the CPU.
The 3D viewer includes a projection selector and Mindat names on hover. Clicking a point
pins its name above the plot. A checkbox controls that caption.
The tested environment is Windows with Python 3.12.10. macOS/Linux has not been tested.

For execution without opening Jupyter:

```powershell
.\.venv\Scripts\python.exe -m jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=1800 notebooks/Mineral_Fusion_Walkthrough.ipynb
```

After saving the executed notebook, export a browser view with:

```powershell
.\.venv\Scripts\python.exe -m jupyter nbconvert --to html notebooks/Mineral_Fusion_Walkthrough.ipynb
```

On macOS/Linux, use `.venv/bin/python`. Open `notebooks/Mineral_Fusion_Walkthrough.html`
in a browser. The embedded 3D plot works offline. Generated HTML is ignored by Git.

## Data and outputs

`data/processed/` contains the prepared input snapshot: aligned arrays, masks, record
identities, feature definitions and Raman source metadata.
`data/raw/Redox_Charge_Balanced.csv` supplies formula coefficients and example matching checks.
`data/raw/mindat.json` supplies mineral context directly on each run.
`data/raw/mindat-synonyms.json` preserves the identity links.
`data/raw/mindat_paragenesis.json` contains Mindat's mineral-mode lists, retrieved on
16 November 2026: 66 codes, 12,945 assignments and 5,600 mineral IDs.
Microprobe workbooks are kept locally in `data/raw/extracted/Microprobe_Data/` as unused
source material. Other unused raw files have been removed.

The snapshot includes 10,736 records and 3,871 Raman specimens. Spectra were normalized
and averaged per base RRUFF specimen ID on a 0–1400 cm⁻¹ grid. Mineral properties are
linked by unique names and documented synonyms. Redox input includes ion proportions and
elemental weight fractions calculated from Kate's formula coefficients. The model uses
fractions from 0 to 1. Unsupported formulas receive missing-value masks.

Running the notebook writes two files to `results/`:

- `embeddings.csv`: one vector per record, with identities, availability flags and split.
- `minerals.csv`: record vectors averaged by Mindat mineral ID.

Results are ignored by Git and replaced on the next run. Figures and validation tables
stay in the notebook. 

## Validation and limits

The notebook checks six recorded specimen joins and splits specimen families 70/15/15.
Identical spectral averages stay together. Numeric scaling uses training data, validation
selects the checkpoint, and test reconstruction is compared with a training-mean baseline.
R070540 is excluded from fitting and mineral averages because its source questions the identity.

Species overlap between training and test. Reconstruction and neighborhood results are
preliminary. PCA, UMAP and t-SNE show the same mineral representatives.

## Sources

- Lafuente et al. (2015), [The RRUFF project](https://doi.org/10.1515/9783110417104-003).
- [Mindat API](https://api.mindat.org/v1/schema/redoc/).
- [Mindat paragenetic modes](https://www.mindat.org/paragen.php), based on
  [Hazen and Morrison (2022)](https://doi.org/10.2138/am-2022-8099).
- Kate's supplied redox CSV and its accompanying notes.

Third-party data retain their source terms. The virtual environment and generated results
are not part of the repository.

A presence-only reference uses the same split and mean-squared-error objective. The
notebook compares Raman test error and strong-peak recall between the two encodings.
An additional three-seed check compares Raman-only, presence fusion and composition
fusion on the same Raman-bearing training specimens, with a fixed split and checkpoint
selection by Raman validation error. Results stay in the notebook.
Atomic weights come from [periodictable](https://periodictable.readthedocs.io/en/latest/api/mass.html).

Mineral context uses 155 Mindat features: 76 element indicators, 9 crystal-system
indicators, hardness minimum/maximum, two measured-density fields and 66 paragenetic
indicators. Mode assignments are linked by Mindat ID. Minerals missing from every mode
list have masked mode features. Parent codes 45 and 47 are represented by their subdivisions.
Assignments cover 7,069 records and 5,573 mineral IDs in this dataset, including 3,667
of the 3,871 Raman specimens.
The ages on the mode page describe stages of mineral evolution; locality ages require
separate records. Type-locality country and locality-age inputs remain pending because
the API returned HTTP 504. The notebook runs from the saved files without API access.
