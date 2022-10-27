# Adapting Pretrained Text-to-Text Models for Long Text Sequences

This repo contains code/checkpoints to reproduce the results of the paper: [Adapting Pretrained Text-to-Text Models for Long Text Sequences](https://arxiv.org/abs/2209.10052). We further pretrain the [BART](https://arxiv.org/abs/1910.13461) model for long sequence tasks, setting new state-of-the-art on abstract summarization of long texts (e.g., GovReport, BookSum, SummScreen, QMSum). Our implementation is based on a custom fork of [fairseq](https://github.com/facebookresearch/fairseq) and [xformers](https://github.com/facebookresearch/xformers).

## Environment Setup
Our model is developed using A100 GPUs and CUDA version 11.6, PyTorch 1.12.1.

# Install xformers and fairseq by running `pip install -e .` under their directory. Install apex following [https://github.com/NVIDIA/apex](https://github.com/NVIDIA/apex).

* Install Triton -- to suppress errors from xformers
```
pip install triton
```

* Install summarizaztion pyrouge and rouge_score
```
pip install -U  git+https://github.com/bheinzerling/pyrouge.git
pip install rouge_score
```



## Model Checkpoints

Model Description                           | Download
--------------------------------------| ---
Pretrained Checkpoint   | [model_100k.pt](https://dl.fbaipublicfiles.com/lsbart/model_100k.pt)
Finetuned on GovReport  | [model_gov.py](https://dl.fbaipublicfiles.com/lsbart/model_fd.pt)
Finetuned on SummScreen-fd | [model_fd.py](https://dl.fbaipublicfiles.com/lsbart/model_fd.pt)
Finetuned on BookSum    | [model_book.py](https://dl.fbaipublicfiles.com/lsbart/model_book.pt)
Dictionary file    | [model_book.py](https://dl.fbaipublicfiles.com/lsbart/dict.txt)


## Code structure

### Tasks
* Pretraining task: [fairseq-py/fairseq/tasks/long_denoising.py]()
* Summarization task: [fairseq-py/fairseq/tasks/summarization.py]()

### Architectures
* Pooling layers: [fairseq-py/fairseq/models/long_transformers/pooling_layers.py]()
* Block Attention: [xformers/xformers/components/attention/block_noglobal.py]().
* Integration to fairseq's transformer architecture: [fairseq-py/fairseq/modules/multihead_attention.py]()

Apart from the block attention implemented with native PyTorch operations, we also provides a faster version within xformers implemented with [Triton](https://github.com/openai/triton): [xformers/xformers/components/attention/blocksparse_local.py](). This implementation brings about 20-30% efficiency gains and slightly worse results. To enable this options, simply pass `--attention-name bs_local`.


### Instruction to finetuning the pretrained model

1. Prepare raw data. Organize you data as **{train|val|test}.{src|tgt}**, where each line corresponds to an example. 
2. Under fairseq-py/, binarize the data following `bash ./scripts/summarization/binarize.sh`. For query-based summarization, check `fairseq-py/scripts/summarization/qmsum_preprocess.sh`
3. The hyperparameters we used for each dataset can be found at [fairseq-py/fb_sweep/long_finetune/sweep_summ.py](). After downloading the checkpoints and put them under checkpoints/, use the following script to run finetuning:

```
bash scripts/summarization/ft_summ.sh
```


### Using released summarizations checkpoints

#### Generating summarizes on summscreen
```
python scripts/summarization/long_generate.py \
            --model-dir ../checkpoints/model_fd.pt \
            --data-dir /fsx/xwhan/data/summscreen/fd-bin \
            --save-dir /fsx/xwhan/data/summscreen/fd/val.hypo.best \
            --split val \
            --bsz 4
```
This script will print ROUGE numbers calculated by [rouge_score](https://pypi.org/project/rouge-score/), which is used by [Scrolls](https://www.scrolls-benchmark.com/leaderboard). In our paper, we reported the rouge scores using [files2rouge](https://github.com/pltrdy/files2rouge). Please follow their repo to install file2rouge and download standord-corenlp for tokenization.


## Cite
```
@article{xiong2022adapting,
  title={Adapting Pretrained Text-to-Text Models for Long Text Sequences},
  author={Xiong, Wenhan and Gupta, Anchit and Toshniwal, Shubham and Mehdad, Yashar and Yih, Wen-tau},
  journal={arXiv preprint arXiv:2209.10052},
  year={2022}
}
```

## License
CC-BY-NC 4.0