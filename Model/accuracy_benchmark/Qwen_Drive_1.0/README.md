# Evaluate Qwen-drive-1.0 model

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

## Evaluate Qwen-drive-1.0 model
### Clone  Qwen-Drive-1.0 github repository
```
$ clone https://github.com/QwenLM/Qwen-Drive-1.0.git qwen-drive
```
### Create virtual environment 
```
$ cd qwen-drive
$ python3 -m venv .venv && source .venv/bin/activate
pip install torch==2.8.0 torchvision==0.23.0 --index-url https://download.pytorch.org/whl/cu128
$ pip install -r requirements.txt --no-build-isolation
$ pip install -e . --no-deps
```
### Download model weights
```
$ hf download Qwen/Qwen-Drive-1.0-4B --local-dir ./Qwen-Drive-1.0-4B
```
### Generate predictions data
```
$nohup python scripts/run_planning.py --model Qwen-Drive-1.0-4B --planner Qwen-Drive-1.0-4B/planner-sft \
    --scenes <KITSCENES_VAL_EVAL_DATA>/kitscenes_val_e2e.jsonl \
    --image-root /tmp/$USER/ratarmount/kitscenes/data  \
    --output <OUTPUT_DIR>/qwendrive/predictions.jsonl \
    --num-samples 6 --seed 0 > <OUTPUT_DIR>/qwendrive/run.log 2>&1 & 
```

### Generate evaluation results
```
$ python eval_qwendrive10_kitscenes.py --predictions <OUTPUT_DIR>/qwendrive/predictions.jsonl \
    --data-dir <KITSCENES_VAL_EVAL_DATA> --out <OUTPUT_DIR>/qwendrive/metrics
```



