# Kilter Project

End-to-end experimentation playground for classifying indoor climbing (Kilter board) problems with PyTorch.
The project builds on the public Kilter Board dataset from [stfamod](https://huggingface.co/datasets/stfamod/Kilter-Board-Dataset),
containing more than 200,000 bouldering routes, and provides multiple machine-learning approaches for predicting route difficulty.

Model snapshot:

- `shallowMLP_tryout.ipynb`: Multi-layer perceptron baseline reaching ~18% accuracy across 21 grades.
- `CNN.ipynb`: Convolutional variants currently achieving ~21% accuracy across the same label set.
- `GNN.ipynb`: Work-in-progress exploration of graph-based encoders.



## Features
- Custom `ClimbingDataset` loader that samples routes from SQLite exports, remaps coordinates, and builds multi-channel hold matrices.
- Multiple CNN backbones (kernel sizes 3–11) and shallow models registered under `CNN_MODEL_REGISTRY` for rapid ablation.
- `train.py` command-line training loop with early stopping, checkpointing, and configurable hyperparameters.
- Reusable training utilities (`engine.py`, `utils.py`, `utils_visualisation.py`) plus nbconvert-compatible notebooks for dataset analysis.
- Backlog.md CLI integration for tracking tasks, decisions, and implementation notes directly from the terminal.

## Project Structure
```
├── data/                  # SQLite databases, CSV exports, and archived datasets
├── models/                # Saved model checkpoints (created at runtime)
├── src/
│   ├── Data_Handler.py    # ClimbingDataset definition and data utilities
│   ├── models.py          # CNN/MLP architectures + registry
│   ├── engine.py          # Training/evaluation loop implementations
│   ├── train.py           # CLI entry point for experiments
│   └── notebooks/         # Jupyter notebooks for exploration
├── backlog/               # Backlog.md configuration and generated tasks
├── requirements.txt       # Core Python dependencies (PyTorch, Optuna, etc.)
└── README.md
```

## Getting Started
1. **Clone and enter the repo**
   ```bash
   git clone <your-fork-url>
   cd kilter
   ```
2. **Create and activate the virtual environment** (Python 3.12 recommended)
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   ```
3. **Install dependencies**
   ```bash
   python -m pip install --upgrade pip
   python -m pip install -r requirements.txt
   # Notebook automation helpers
   python -m pip install nbconvert nbclient nbformat
   ```

### Dataset Assets
The repository already includes the board databases and exports inside `data/`:
- `boards.db`, `static.db`: SQLite sources for route metadata.
- `routes_onehot.csv`: Precomputed representations for analysis.
- `dataset.zip`: Archived dataset snapshot.

No manual download is required, but ensure you have disk space (~100MB+) before running experiments.

## Training
Run experiments via `src/train.py` (execute from the `src/` directory so relative imports resolve):
```bash
cd src
python train.py \
  --num-epochs 10 \
  --patience 1 \
  --model-type CNN_K7 \
  --hidden-units-cnn 16 \
  --hidden-units-classifier 8 \
  --model-name "experiment_name"
```
Key parameters:
- `--model-type`: Chooses the architecture from `CNN_MODEL_REGISTRY`; larger kernels such as `CNN_K9` capture broader hold context, while shallower options like `shallowCNN` train faster.
- `--hidden-units-cnn`: Sets the number of convolutional feature maps; increase for richer representations at the cost of VRAM and training time.
- `--hidden-units-classifier`: Governs the fully connected head width; use higher values when classes are hard to separate, lower values for lighter models.
- `--lr` / `--weight-decay`: Optimizer hyperparameters exposed for quick schedule tweaks without editing code.
- `--map`, `--label-filter`, `--max-samples`: Control dataset preprocessing—coordinate remapping, difficulty selection, and dataset truncation (see `Data_Handler.py`).

Models are saved to `models/<model-name>.pt` and best checkpoints land in `src/checkpoints/` when early stopping triggers.

### Evaluating Dataloaders & Dataset Stats
Notebooks under `src/notebooks/` provide exploratory analysis. Highlights:
- `CNN.ipynb` (`CNN`): Compares convolutional backbones and reports accuracy curves across kernel sizes.
- `Custom_Dataset.ipynb` (`custom_dataset`): Inspects dataset composition, hold distributions, and dataloader throughput.
- `shallowMLP_tryout.ipynb` (`shallow_MLP_tryout`): Benchmarks fully connected baselines against the CNN family.
- `Static_Data_loader.ipynb` (`static_data`): Demonstrates loading static board snapshots and validating preprocessing routines.

To execute headlessly:
```bash
python -m nbconvert --to notebook --execute --inplace src/notebooks/Custom_Dataset.ipynb
```
Alternatively, launch Jupyter Lab/Notebook if you prefer an interactive workflow.

## Task Management with Backlog.md
Project task management is powered by [Backlog.md](https://github.com/MrLesk/Backlog.md).

## Contributing
1. Use Backlog tasks to plan work before touching code.
2. Keep notebooks and scripts reproducible—document new CLI flags or data requirements in the README.
3. Run training or evaluation commands relevant to your changes and summarize results in task notes.
4. Prefer small, reviewable changesets and attach implementation notes via Backlog before marking tasks done.

## License
License information is currently unspecified. Until a license file is added, treat the project as private/internal.
