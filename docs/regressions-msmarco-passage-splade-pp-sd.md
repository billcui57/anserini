# Anserini Regressions: MS MARCO Passage Ranking

**Model**: SPLADE++ CoCondenser-SelfDistil

This page describes regression experiments, integrated into Anserini's regression testing framework, using the [SPLADE++ CoCondenser-SelfDistil](https://huggingface.co/naver/splade-cocondenser-selfdistil) model on the [MS MARCO passage ranking task](https://github.com/microsoft/MSMARCO-Passage-Ranking), as described in the following paper:

> Thibault Formal, Carlos Lassance, Benjamin Piwowarski, and Stéphane Clinchant. [From Distillation to Hard Negative Sampling: Making Sparse Neural IR Models More Effective.](https://dl.acm.org/doi/10.1145/3477495.3531857) _Proceedings of the 45th International ACM SIGIR Conference on Research and Development in Information Retrieval_, pages 2353–2359.

The exact configurations for these regressions are stored in [this YAML file](../src/main/resources/regression/msmarco-passage-splade-pp-sd.yaml).
Note that this page is automatically generated from [this template](../src/main/resources/docgen/templates/msmarco-passage-splade-pp-sd.template) as part of Anserini's regression pipeline, so do not modify this page directly; modify the template instead and then run `bin/build.sh` to rebuild the documentation.

From one of our Waterloo servers (e.g., `orca`), the following command will perform the complete regression, end to end:

```bash
python src/main/python/run_regression.py --index --verify --search --regression msmarco-passage-splade-pp-sd
```

We make available a version of the MS MARCO passage corpus that has already been encoded with SPLADE++ CoCondenser-SelfDistil.

From any machine, the following command will download the corpus and perform the complete regression, end to end:

```bash
python src/main/python/run_regression.py --download --index --verify --search --regression msmarco-passage-splade-pp-sd
```

The `run_regression.py` script automates the following steps, but if you want to perform each step manually, simply copy/paste from the commands below and you'll obtain the same regression results.

## Corpus Download

Download the corpus and unpack into `collections/`:

```bash
wget https://rgw.cs.uwaterloo.ca/JIMMYLIN-bucket0/data/msmarco-passage-splade_distil_cocodenser_medium.tar -P collections/
tar xvf collections/msmarco-passage-splade_distil_cocodenser_medium.tar -C collections/
```

To confirm, `msmarco-passage-splade_distil_cocodenser_medium.tar` is 4.9 GB and has MD5 checksum `f77239a26d08856e6491a34062893b0c`.
With the corpus downloaded, the following command will perform the remaining steps below:

```bash
python src/main/python/run_regression.py --index --verify --search --regression msmarco-passage-splade-pp-sd \
  --corpus-path collections/msmarco-passage-splade-pp-sd
```

## Indexing

Sample indexing command:

```bash
target/appassembler/bin/IndexCollection \
  -collection JsonVectorCollection \
  -input /path/to/msmarco-passage-splade-pp-sd \
  -index indexes/lucene-index.msmarco-passage-splade-pp-sd/ \
  -generator DefaultLuceneDocumentGenerator \
  -threads 16 -impact -pretokenized -storeDocvectors \
  >& logs/log.msmarco-passage-splade-pp-sd &
```

The path `/path/to/msmarco-passage-splade-pp-sd/` should point to the corpus downloaded above.

The important indexing options to note here are `-impact -pretokenized`: the first tells Anserini not to encode BM25 doc lengths into Lucene's norms (which is the default) and the second option says not to apply any additional tokenization on the pre-encoded tokens.
Upon completion, we should have an index with 8,841,823 documents.

For additional details, see explanation of [common indexing options](common-indexing-options.md).

## Retrieval

Topics and qrels are stored in [`tools/topics-and-qrels/`](../tools/topics-and-qrels/).
The regression experiments here evaluate on the 6980 dev set questions; see [this page](experiments-msmarco-passage.md) for more details.

After indexing has completed, you should be able to perform retrieval as follows:

```bash
target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.msmarco-passage-splade-pp-sd/ \
  -topics tools/topics-and-qrels/topics.msmarco-passage.dev-subset.splade-pp-sd.tsv.gz \
  -topicreader TsvInt \
  -output runs/run.msmarco-passage-splade-pp-sd.splade-pp-sd.topics.msmarco-passage.dev-subset.splade-pp-sd.txt \
  -impact -pretokenized &
```

Evaluation can be performed using `trec_eval`:

```bash
tools/eval/trec_eval.9.0.4/trec_eval -c -m map tools/topics-and-qrels/qrels.msmarco-passage.dev-subset.txt runs/run.msmarco-passage-splade-pp-sd.splade-pp-sd.topics.msmarco-passage.dev-subset.splade-pp-sd.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -M 10 -m recip_rank tools/topics-and-qrels/qrels.msmarco-passage.dev-subset.txt runs/run.msmarco-passage-splade-pp-sd.splade-pp-sd.topics.msmarco-passage.dev-subset.splade-pp-sd.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.100 tools/topics-and-qrels/qrels.msmarco-passage.dev-subset.txt runs/run.msmarco-passage-splade-pp-sd.splade-pp-sd.topics.msmarco-passage.dev-subset.splade-pp-sd.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.1000 tools/topics-and-qrels/qrels.msmarco-passage.dev-subset.txt runs/run.msmarco-passage-splade-pp-sd.splade-pp-sd.topics.msmarco-passage.dev-subset.splade-pp-sd.txt
```

## Effectiveness

With the above commands, you should be able to reproduce the following results:

| **AP@1000**                                                                                                  | **SPLADE++ CoCondenser-SelfDistil**|
|:-------------------------------------------------------------------------------------------------------------|-----------|
| [MS MARCO Passage: Dev](https://github.com/microsoft/MSMARCO-Passage-Ranking)                                | 0.3837    |
| **RR@10**                                                                                                    | **SPLADE++ CoCondenser-SelfDistil**|
| [MS MARCO Passage: Dev](https://github.com/microsoft/MSMARCO-Passage-Ranking)                                | 0.3778    |
| **R@100**                                                                                                    | **SPLADE++ CoCondenser-SelfDistil**|
| [MS MARCO Passage: Dev](https://github.com/microsoft/MSMARCO-Passage-Ranking)                                | 0.9118    |
| **R@1000**                                                                                                   | **SPLADE++ CoCondenser-SelfDistil**|
| [MS MARCO Passage: Dev](https://github.com/microsoft/MSMARCO-Passage-Ranking)                                | 0.9846    |

## Reproduction Log[*](reproducibility.md)

To add to this reproduction log, modify [this template](../src/main/resources/docgen/templates/msmarco-passage-splade-pp-sd.template) and run `bin/build.sh` to rebuild the documentation.

+ Results reproduced by [@justram](https://github.com/justram) on 2023-03-08 (commit [`03f95a8`](https://github.com/castorini/anserini/commit/03f95a8e1ae09ab09efe046bfcbd3a4cdda691b4))
