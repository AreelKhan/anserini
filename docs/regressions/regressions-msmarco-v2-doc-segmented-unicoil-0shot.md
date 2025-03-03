# Anserini Regressions: MS MARCO (V2) Document Ranking

**Model**: uniCOIL (with doc2query-T5 expansions) zero-shot on segmented documents (segment-only encoding) - _Deprecated_, see below

This page describes regression experiments for document ranking _on the segmented version_ of the MS MARCO (V2) document corpus using the dev queries, which is integrated into Anserini's regression testing framework.
Here, we cover experiments with the uniCOIL model trained on the MS MARCO V1 passage ranking test collection, applied in a zero-shot manner, with doc2query-T5 expansion.

The uniCOIL model is described in the following paper:

> Jimmy Lin and Xueguang Ma. [A Few Brief Notes on DeepImpact, COIL, and a Conceptual Framework for Information Retrieval Techniques.](https://arxiv.org/abs/2106.14807) _arXiv:2106.14807_.

**NOTE**: As an important detail, there is the question of what text we feed into the encoder to generate document representations.
Initially, we fed only the segment text, but later we realized that prepending the title of the document improves effectiveness.
This regression captures segment-only encoding and is kept around primarily for archival purposes; you probably don't want to use this one unless you're running ablation experiments.
The version that uses title/segment encoding can be found [here](regressions-msmarco-v2-doc-segmented-unicoil-0shot-v2.md).

The exact configurations for these regressions are stored in [this YAML file](../../src/main/resources/regression/msmarco-v2-doc-segmented-unicoil-0shot.yaml).
Note that this page is automatically generated from [this template](../../src/main/resources/docgen/templates/msmarco-v2-doc-segmented-unicoil-0shot.template) as part of Anserini's regression pipeline, so do not modify this page directly; modify the template instead.

From one of our Waterloo servers (e.g., `orca`), the following command will perform the complete regression, end to end:

```bash
python src/main/python/run_regression.py --index --verify --search --regression msmarco-v2-doc-segmented-unicoil-0shot
```

We make available a version of the MS MARCO document corpus that has already been processed with uniCOIL (per above), i.e., we have applied doc2query-T5 expansions, performed model inference on every document, and stored the output sparse vectors.
Thus, no neural inference is involved.

From any machine, the following command will download the corpus and perform the complete regression, end to end:

```bash
python src/main/python/run_regression.py --download --index --verify --search --regression msmarco-v2-doc-segmented-unicoil-0shot
```

The `run_regression.py` script automates the following steps, but if you want to perform each step manually, simply copy/paste from the commands below and you'll obtain the same regression results.

## Corpus Download

Download, unpack, and prepare the corpus:

```bash
# Download
wget https://rgw.cs.uwaterloo.ca/JIMMYLIN-bucket0/data/msmarco_v2_doc_segmented_unicoil_0shot.tar -P collections/

# Unpack
tar -xvf collections/msmarco_v2_doc_segmented_unicoil_0shot.tar -C collections/

# Rename (indexer is expecting corpus under a slightly different name)
mv collections/msmarco_v2_doc_segmented_unicoil_0shot collections/msmarco-v2-doc-segmented-unicoil-0shot
```

To confirm, `msmarco_v2_doc_segmented_unicoil_0shot.tar` is 62 GB and has an MD5 checksum of `889db095113cc4fe152382ccff73304a`.
With the corpus downloaded, the following command will perform the remaining steps below:

```bash
python src/main/python/run_regression.py --index --verify --search --regression msmarco-v2-doc-segmented-unicoil-0shot \
  --corpus-path collections/msmarco-v2-doc-segmented-unicoil-0shot
```

## Indexing

Sample indexing command:

```bash
target/appassembler/bin/IndexCollection \
  -collection JsonVectorCollection \
  -input /path/to/msmarco-v2-doc-segmented-unicoil-0shot \
  -generator DefaultLuceneDocumentGenerator \
  -index indexes/lucene-index.msmarco-v2-doc-segmented-unicoil-0shot/ \
  -threads 24 -impact -pretokenized \
  >& logs/log.msmarco-v2-doc-segmented-unicoil-0shot &
```

The path `/path/to/msmarco-v2-doc-segmented-unicoil-0shot/` should point to the corpus downloaded above.

The important indexing options to note here are `-impact -pretokenized`: the first tells Anserini not to encode BM25 doclengths into Lucene's norms (which is the default) and the second option says not to apply any additional tokenization on the uniCOIL tokens.
Upon completion, we should have an index with 124,131,414 documents.

For additional details, see explanation of [common indexing options](../../docs/common-indexing-options.md).

## Retrieval

Topics and qrels are stored [here](https://github.com/castorini/anserini-tools/tree/master/topics-and-qrels), which is linked to the Anserini repo as a submodule.

After indexing has completed, you should be able to perform retrieval as follows:

```bash
target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.msmarco-v2-doc-segmented-unicoil-0shot/ \
  -topics tools/topics-and-qrels/topics.msmarco-v2-doc.dev.unicoil.0shot.tsv.gz \
  -topicReader TsvInt \
  -output runs/run.msmarco-v2-doc-segmented-unicoil-0shot.unicoil-0shot.topics.msmarco-v2-doc.dev.unicoil.0shot.txt \
  -parallelism 16 -impact -pretokenized -hits 10000 -selectMaxPassage -selectMaxPassage.delimiter "#" -selectMaxPassage.hits 1000 &
target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.msmarco-v2-doc-segmented-unicoil-0shot/ \
  -topics tools/topics-and-qrels/topics.msmarco-v2-doc.dev2.unicoil.0shot.tsv.gz \
  -topicReader TsvInt \
  -output runs/run.msmarco-v2-doc-segmented-unicoil-0shot.unicoil-0shot.topics.msmarco-v2-doc.dev2.unicoil.0shot.txt \
  -parallelism 16 -impact -pretokenized -hits 10000 -selectMaxPassage -selectMaxPassage.delimiter "#" -selectMaxPassage.hits 1000 &
```

Evaluation can be performed using `trec_eval`:

```bash
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.100 tools/topics-and-qrels/qrels.msmarco-v2-doc.dev.txt runs/run.msmarco-v2-doc-segmented-unicoil-0shot.unicoil-0shot.topics.msmarco-v2-doc.dev.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.1000 tools/topics-and-qrels/qrels.msmarco-v2-doc.dev.txt runs/run.msmarco-v2-doc-segmented-unicoil-0shot.unicoil-0shot.topics.msmarco-v2-doc.dev.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -M 100 -m map -c -M 100 -m recip_rank tools/topics-and-qrels/qrels.msmarco-v2-doc.dev.txt runs/run.msmarco-v2-doc-segmented-unicoil-0shot.unicoil-0shot.topics.msmarco-v2-doc.dev.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.100 tools/topics-and-qrels/qrels.msmarco-v2-doc.dev2.txt runs/run.msmarco-v2-doc-segmented-unicoil-0shot.unicoil-0shot.topics.msmarco-v2-doc.dev2.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.1000 tools/topics-and-qrels/qrels.msmarco-v2-doc.dev2.txt runs/run.msmarco-v2-doc-segmented-unicoil-0shot.unicoil-0shot.topics.msmarco-v2-doc.dev2.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -M 100 -m map -c -M 100 -m recip_rank tools/topics-and-qrels/qrels.msmarco-v2-doc.dev2.txt runs/run.msmarco-v2-doc-segmented-unicoil-0shot.unicoil-0shot.topics.msmarco-v2-doc.dev2.unicoil.0shot.txt
```

## Effectiveness

With the above commands, you should be able to reproduce the following results:

| **MAP@100**                                                                                                  | **uniCOIL (with doc2query-T5) zero-shot**|
|:-------------------------------------------------------------------------------------------------------------|-----------|
| [MS MARCO V2 Doc: Dev](https://microsoft.github.io/msmarco/TREC-Deep-Learning.html)                          | 0.2218    |
| [MS MARCO V2 Doc: Dev2](https://microsoft.github.io/msmarco/TREC-Deep-Learning.html)                         | 0.2270    |
| **MRR@100**                                                                                                  | **uniCOIL (with doc2query-T5) zero-shot**|
| [MS MARCO V2 Doc: Dev](https://microsoft.github.io/msmarco/TREC-Deep-Learning.html)                          | 0.2243    |
| [MS MARCO V2 Doc: Dev2](https://microsoft.github.io/msmarco/TREC-Deep-Learning.html)                         | 0.2291    |
| **R@100**                                                                                                    | **uniCOIL (with doc2query-T5) zero-shot**|
| [MS MARCO V2 Doc: Dev](https://microsoft.github.io/msmarco/TREC-Deep-Learning.html)                          | 0.7551    |
| [MS MARCO V2 Doc: Dev2](https://microsoft.github.io/msmarco/TREC-Deep-Learning.html)                         | 0.7550    |
| **R@1000**                                                                                                   | **uniCOIL (with doc2query-T5) zero-shot**|
| [MS MARCO V2 Doc: Dev](https://microsoft.github.io/msmarco/TREC-Deep-Learning.html)                          | 0.9056    |
| [MS MARCO V2 Doc: Dev2](https://microsoft.github.io/msmarco/TREC-Deep-Learning.html)                         | 0.9097    |

## Reproduction Log[*](../../docs/reproducibility.md)

To add to this reproduction log, modify [this template](../../src/main/resources/docgen/templates/msmarco-v2-doc-segmented-unicoil-0shot.template) and run `bin/build.sh` to rebuild the documentation.

+ Results reproduced by [@lintool](https://github.com/lintool) on 2022-06-06 (commit [`236b386`](https://github.com/castorini/anserini/commit/236b386ddc11d292b4b736162b59488a02236d6c))
