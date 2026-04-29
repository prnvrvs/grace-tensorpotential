# macOS Installation

This page shows one working way to install `grace-tensorpotential` on macOS with TensorFlow Metal.

## Recommended Setup

Use a fresh Python 3.11 conda environment and install the TensorFlow Metal stack explicitly.

```bash
conda create -n grace python=3.11 -y
conda activate grace
python -m pip install --upgrade pip
python -m pip install "tensorflow==2.18.1" "tensorflow-metal==1.2.0" "tf_keras<2.19"
```

Verify that TensorFlow sees the GPU before installing GRACE:

```bash
python -c "import tensorflow as tf; print(tf.__version__); print(tf.config.list_physical_devices('GPU'))"
```

You should see `GPU:0`.

## Install GRACE

Clone the repository and install it:

```bash
git clone https://github.com/prnvrvs/grace-tensorpotential.git
cd grace-tensorpotential
python -m pip install -e .
```

If you prefer a non-editable install:

```bash
python -m pip install .
```

## Runtime Dependencies

If you want to run the ASE examples and foundation-model paths, install:

```bash
python -m pip install pyyaml pandas==2.3.3 pytz python-dateutil mpmath ase scipy sympy matscipy tqdm questionary rich
```

## Check GRACE on GPU

Run a minimal probe:

```bash
python - <<'PY'
import tensorflow as tf
from tensorpotential.calculator import grace_fm

print("gpu_before", tf.config.list_physical_devices("GPU"))
calc = grace_fm("GRACE-1L-OMAT")
print("calc", type(calc).__name__)
print("gpu_after", tf.config.list_physical_devices("GPU"))
PY
```

Expected output:

- `gpu_before` includes `GPU:0`
- `calc` is `TPCalculator`
- `gpu_after` still includes `GPU:0`

## Run An ASE MD

Example:

```python
from ase.build import bulk
from ase.md.langevin import Langevin
from ase import units
from ase.md.velocitydistribution import MaxwellBoltzmannDistribution
from tensorpotential.calculator import grace_fm

atoms = bulk("Fe", "bcc", a=2.87, cubic=True) * (4, 4, 4)
atoms.calc = grace_fm("GRACE-1L-OMAT")

MaxwellBoltzmannDistribution(atoms, temperature_K=300)
dyn = Langevin(atoms, timestep=1.0 * units.fs, temperature_K=300, friction=0.02)

for step in range(1, 101):
    dyn.run(1)
    if step % 10 == 0 or step == 1:
        print(step, atoms.get_potential_energy())
```

## Notes

- If GPU detection fails, make sure you are not inside a contaminated shell with `CONDA_PREFIX` or a custom `PYTHONPATH`.
- If you are developing the repo, `pip install -e .` is fine for code changes.
- If you only want the most stable runtime path, use `python -m pip install .`.
