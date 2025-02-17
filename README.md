# QE-fusion

> [**Don't Rank, Combine! Combining Machine Translation Hypotheses Using Quality Estimation**](https://arxiv.org/abs/2401.06688)  
> Giorgos Vernikos, Andrei Popescu-Belis

## Overview
QE-fusion⚛ is a novel approach that uses Quality Estimation (QE) metrics to improve translations by fusing different translation candidates. QE-fusion leverages the potential complementarity of a candidate pool by spotting variations among candidates and using a QE metric to select the spans that boost overall translation quality.

<p align="left">
  <img src="qe-fusion_fig.png" width="800">
</p>

This repository contains code for sampling translations from an LLM or an MT model, implementing QE-fusion, and other reranking approaches, and scoring the corresponding outputs.

## Installation

This project requires `Python 3.10`, `PyTorch 1.13.1`, and transformers `4.34.0`.

It's advisable to set up a separate environment for this project and install the necessary dependencies:

```
conda create -n qe-fusion python=3.10
conda activate qe-fusion
pip install -r requirements.txt
```

(for BLEURT)
```
git clone https://github.com/google-research/bleurt.git
cd bleurt; pip install .;
wget https://storage.googleapis.com/bleurt-oss-21/BLEURT-20.zip .; unzip BLEURT-20.zip; 
cd ..;
```

## Obtain Translation Hypotheses

The script assumes that the test set of each language pair (can be downloaded from [sacreBLEU](https://github.com/mjpost/sacrebleu)) is located in the folder `data/<s>-<t>/` under the names `test.<s>` and `test.<t>` where `<s>` and `<t>` are the source and target languages.
```
mkdir -p data/en-de
sacrebleu -t wmt14 -l en-de --echo src > data/en-de/test.en
sacrebleu -t wmt14 -l en-de --echo ref > data/en-de/test.de 
```

The first step is to obtain multiple translation hypotheses for each input. To do so, we either use an LLM via few-shot (8-shot) learning or an MT model from which we sample *k* outputs using the `llm_query.py` script.

```
python llm_query.py --model nllb-1.3b --lp en-de --exemplars 8 --bsize 4 --decoding_alg greedy
python llm_query.py --model nllb-1.3b --lp en-de --exemplars 8 --bsize 4 --decoding_alg beam --num_beams 5 --sample 1 --cuda 0
python llm_query.py --model nllb-1.3b --lp en-de --exemplars 8 --bsize 4 --decoding_alg beam --num_beams 10 --sample 5 --cuda 0
python llm_query.py --model nllb-1.3b --lp en-de --exemplars 8 --bsize 4 --decoding_alg sample --num_beams 1 --sample 5 --temperature 0.6 --cuda 1
python llm_query.py --model nllb-1.3b --lp en-de --exemplars 8 --bsize 4 --decoding_alg sample --num_beams 5 --sample 5 --temperature 0.6 --cuda 2
python llm_query.py --model nllb-1.3b --lp en-de --exemplars 8 --bsize 4 --decoding_alg sample --num_beams 10 --sample 5 --temperature 0.6 --cuda 3
```
> This script also supports greedy decoding and beam search via the `decoding_alg` parameter.

## Fusion of Hypotheses

To fuse the generated hypotheses use the `select_outputs.py` script:

```
python select_outputs.py --model nllb-1.3b --lp en-de --generation sample-t0.6 --cands_pool 5 --criterion cometqe --method fusion
```
> To implement reranking approaches such as QE-reranking and MBR modify the `--criterion` and `--method` parameters accordingly.

## Evaluation

To evaluate the quality of the produced translations use the `score_outs.py` script:

```
python score_outs.py --model nllb-1.3b --lp en-de --cands_pool 5 --generation sample-t0.6 --criterion cometqe/cometqe-fusion-beam5-kbest0  --metrics bleu chrf comet bleurt
```

---
## Reference
Please feel free to cite our paper if you use our code or proposed algorithm.:
```
@misc{vernikos2024dont,
      title={Don't Rank, Combine! Combining Machine Translation Hypotheses Using Quality Estimation}, 
      author={Giorgos Vernikos and Andrei Popescu-Belis},
      year={2024},
      eprint={2401.06688},
      archivePrefix={arXiv},
      primaryClass={cs.CL}
}
```

---
## Contact
Please feel free to raise an issue or contact me in case you require any help setting up the repo!
