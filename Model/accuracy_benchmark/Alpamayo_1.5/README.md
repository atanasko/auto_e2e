# Evaluate Alpamayo_1.5 model

## Download KITScenes-Multimodal val subset 

### Clone KITScenes github repository

```
$ git clone https://github.com/KIT-MRT/kitscenes.git
```

### Create virtual environment

```
$ cd kitscenes
$ python3 -m venv .venv && source .venv/bin/activate
```
### Install requirements
```
$ python -m pip install -U pip
$ pip install -U huggingface_hub hf_xet
$ pip install ratarmount
$ pip install -e .
```
### Download KITScenes validation split
```
$ export KITSCENES_ROOT=<KITSCENES_ROOT_DIR>
$ hf download KIT-MRT/KITScenes-Multimodal \
    --repo-type dataset \
    --include "data/val/*" \
    --local-dir "$KITSCENES_ROOT"
```
### Mount the root directory
```
$ bash scripts/mount_kitscenes_tars.sh <KITSCENES_ROOT_DIR>
$ export KITSCENES_ROOT=/tmp/$USER/ratarmount/kitscenes
```
If needed umount the directory
```
$ for d in /tmp/$USER/ratarmount/kitscenes/data/*/; do
  ratarmount -u "$d" || fusermount -u "$d"
done
```

### Prepare KITScenes evaluation data for val split
```
$ python prepare_kitscenes_e2e.py --out-dir <KITSCENES_VAL_EVAL_DATA>
```

## Evaluate Alpamayo_1.5 model
### Clone  Alpamayo_1.5 github repository
```
$ git clone https://github.com/NVlabs/alpamayo1.5
```
### Create virtual environment 
```
$ cd aalpamayo1.5
$ python3 -m venv .venv && source .venv/bin/activate
$ python -m pip install -U pip wheel setuptools
$ pip install -e . --no-deps
$ python - <<'EOF' \
    import tomllib \
    deps = tomllib.load(open("pyproject.toml", "rb"))["project"]["dependencies"] \
    open("requirements.txt", "w").write("\n".join(d for d in deps if not d.startswith("flash-attn")) + "\n") \
    print(open("requirements.txt").read()) \
EOF
$ pip install "torch>=2.8" torchvision
$ pip install -r requirements.txt
$ pip install ninja packaging
$ MAX_JOBS=8 pip install "flash-attn>=2.8.3" --no-build-isolation
```
### Generate predictions data and evaluation results
```
$ nohup python eval_alpamayo15_kitscenes.py --data-dir <KITSCENES_VAL_EVAL_DATA> \
    --image-root /tmp/$USER/ratarmount/kitscenes/data \
    --out <OUTPUT_DIR>/alpamayo15 --num-samples 3 --seed 42 > <OUTPUT_DIR>/alpamayo15/run.log 2>&1 &
```
