# SpaceNet 8 flooded-road experiment package

Status: syntax-checked only. NOT executed by the assistant (no GPU, no PyTorch, no SpaceNet 8 data in its sandbox).
Expect to fix small data-layout issues in prepare_data.py on first run (CSV column names, 'flooded' property).

## 1. Environment
python 3.10+, `pip install -r requirements.txt` (use the CUDA build of torch for your GPU), 1 GPU with >=12 GB.

## 2. Download (public AWS bucket, no account needed)
    aws s3 ls --no-sign-request s3://spacenet-dataset/spacenet/SN8_floods/tarballs/
    aws s3 cp --no-sign-request s3://spacenet-dataset/spacenet/SN8_floods/tarballs/Germany_Training_Public.tar.gz raw/
    aws s3 cp --no-sign-request s3://spacenet-dataset/spacenet/SN8_floods/tarballs/Louisiana-East_Training_Public.tar.gz raw/
    cd raw && for f in *.tar.gz; do tar xzf $f; done && cd ..
(If a file name differs, use the `aws s3 ls` output. Look at the CSV and one road geojson before continuing:
`head -3 raw/Germany_Training_Public/*.csv`.)

## 3. Prepare, split
    python prepare_data.py --raw raw --out data/processed      # prints columns used + dataset_stats.json
    python make_splits.py --manifest data/processed/manifest.csv --out data/splits.csv --split_seed 42

Check: dataset_stats.json should show a non-zero flooded share and no "missing 'flooded' property" warning.

## 4. Quick debug run (minutes), then the full study
    python train.py --data data/processed --splits data/splits.csv --protocol A --variant siam_full --seed 0 --epochs 2 --max_tiles 8 --workers 0 --out debug
    ./run_all.sh          # 30 runs; aggregate.py + make_figures.py run at the end

Seeds 0,1,2 change init/crops/augmentation only; the split (seed 42) is fixed. Outputs: runs/<protocol>/<variant>/seed<k>/metrics.json.
Send me results_summary.txt (and ideally the metrics.json files) and I will fill the paper from them only.
