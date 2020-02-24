# Generate paper titles

We train a model that generates titles for mathematical papers based on their abstracts. This proof-of-concept repo will demonstrate AzureML workflow for a typical data science project. Concretely we set up pipelines to:

- grab data from the arXiv
- process the data
- train a simple NMT attention model in PyTorch

The NMT model we use here is strictly for demonstration purposes and was forked from here: https://github.com/pcyin/pytorch_basic_nmt

TODO: retitle this repo with a title generated by the model and with the above as the input :-)

## Running this pipeline

- TODO: Grab raw arxiv-data
- TODO: Process raw text
- Build vocab 
```
python .\vocab.py --train-src=data/arxiv/abstract --train-tgt=data/arxiv/title data/vocab.json
```
- TODO: Train model

On CPU:
```
python nmt.py train --train-src=data/arxiv/abstract --train-tgt=data/arxiv/title --dev-src=data/arxiv/abstract.dev --dev-tgt=data/arxiv/title.dev --vocab=data/vocab.json
```

On GPU:
```
python nmt.py train --cuda --train-src=data/arxiv/abstract --train-tgt=data/arxiv/title --dev-src=data/arxiv/abstract.dev --dev-tgt=data/arxiv/title.dev --vocab=data/vocab.json
```

----

Below are the instructions taken verbatim from https://github.com/pcyin/pytorch_basic_nmt for training their NMT model.

## A Basic PyTorch Implementation of Attentional Neural Machine Translation

This is a basic implementation of attentional neural machine translation (Bahdanau et al., 2015, Luong et al., 2015) in Pytorch 0.4.
It implements the model described in [Luong et al., 2015](https://arxiv.org/abs/1508.04025), and supports label smoothing, beam-search decoding and random sampling.
With 256-dimensional LSTM hidden size, it achieves 28.13 BLEU score on the IWSLT 2014 Germen-English dataset (Ranzato et al., 2015).

This codebase is used for instructional purposes in Stanford [CS224N Nautral Language Processing with Deep Learning]( http://web.stanford.edu/class/cs224n/) and CMU [11-731 Machine Translation and Sequence-to-Sequence Models](http://www.phontron.com/class/mtandseq2seq2018/).

### File Structure

* `nmt.py`: contains the neural machine translation model and training/testing code.
* `vocab.py`: a script that extracts vocabulary from training data
* `util.py`: contains utility/helper functions

### Example Dataset

We provide a preprocessed version of the IWSLT 2014 German-English translation task used in (Ranzato et al., 2015) [[script]](https://github.com/harvardnlp/BSO/blob/master/data_prep/MT/prepareData.sh). To download the dataset:

```bash
wget http://www.cs.cmu.edu/~pengchey/iwslt2014_ende.zip
unzip iwslt2014_ende.zip
```

Running the script will extract a`data/` folder which contains the IWSLT 2014 dataset.
The dataset has 150K German-English training sentences. The `data/` folder contains a copy of the public release of the dataset. Files with suffix `*.wmixerprep` are pre-processed versions of the dataset from Ranzato et al., 2015, with long sentences chopped and rared words replaced by a special `<unk>` token. You could use the pre-processed training files for training/developing (or come up with your own pre-processing strategy), but for testing you have to use the **original** version of testing files, ie., `test.de-en.(de|en)`.

### Environment

The code is written in Python 3.6 using some supporting third-party libraries. We provided a conda environment to install Python 3.6 with required libraries. Simply run

```bash
conda env create -f environment.yml
```

### Usage

Each runnable script (`nmt.py`, `vocab.py`) is annotated using `dotopt`.
Please refer to the source file for complete usage.

First, we extract a vocabulary file from the training data using the command:

```bash
python vocab.py \
    --train-src=data/train.de-en.de.wmixerprep \
    --train-tgt=data/train.de-en.en.wmixerprep \
    data/vocab.json
```

This generates a vocabulary file `data/vocab.json`. 
The script also has options to control the cutoff frequency and the size of generated vocabulary, which you may play with.

To start training and evaluation, simply run `data/train.sh`. 
After training and decoding, we call the official evaluation script `multi-bleu.perl` to compute the corpus-level BLEU score of the decoding results against the gold-standard.

### License

This work is licensed under a Creative Commons Attribution 4.0 International License.

# AML Pipelines

Following this pipelines tutorial: https://github.com/Azure/aml-object-recognition-pipeline

Painpoint:
- Debuging script is painful! Getting packages into the compute instance is a little painful (especially for a new data scientist using AML!). Process of:
    - build pipeline
    - submit to AML
    - wait for pipeline to get started, 5+ minutes
    - get error message (after clicking around on a few tabs is it Output + logs or Raw JSON)
    - make code change and rebuild the pipeline, start again
- Compute instance vs DSVM
    - On DSVM the notebook experiance lives in the notebook directory, so we got used to putting out code in three
    - On compute instance notebook seems to live in a really strange place: `/mnt/batch/tasks/shared/LS_root/mounts/clusters/citest/`
- Good development environment is key. Mine was a little janky (to say the least). Looking for better solutions that this:
    - Final workflow: have VS Code SSH into compute instance where my code lives. In parallel have a jupyter notebook open in the same compute instance with a terminal to run jobs.
    - This requires: setting up SSH on the compute instance, and adding SSH keys for each place you want to work from. That's okay, but to grant an additional machine SSH access I had to SSH in from another machine and manually update the ssh config. Worse still, I found that my second machine forgot that I had granted it SSH access so I ended up being forced to work from machine 1.
    - Jupyter on compute instance lands you in a strange directory `/mnt/batch/tasks/shared/LS_root/mounts/clusters/<compute-instance-name>/`. To get parity with VS code have to navigate here too. This is not a good user experience.
- Relative imports: Modules that reuse logic need to live in the same directory.
    - Would be nice to have concept of "shared module"
    - Example:
        - `script1.py` and `script2.py` both use the same class `MyClass` in `myclass.py`.
        - The structure `module1/{script1.py}`, `module2/{script2.py}`, `shared/{myclass.py}` does not work since we pass `source_directory=os.path.dirname(os.path.abspath(__file__)),`
        - Instead using `module1_and_2/{script1.py, script2.pt, myclass.py}` will work.
- Unable to see data from previous pipeline step:
    - Wasted 2 days figuring this out: https://docs.microsoft.com/en-us/azure/machine-learning/how-to-debug-pipelines#troubleshooting-tips
    - Solution: makesure you use `os.makedirs(args.output_dir, exist_ok=True)`
    - This issue is easily solved if you've seen it before, very hard to solve otherwise!
- Be careful changing names of modules!
    - I changed the name of a module, but forgot to change one of the places it was being called. This was causing strange, hard to debug errors pointing to things that were no longer there in my code. The answer (I think!) is that aml is caching the steps you uploaded previously, so it was still pointing to code that no longer exists?


Broken:
- Ingest step dumps data in 'raw_data_dir', but preprocess step cannot find the directory.
    - Ans: need to add os.makedirs.
    - This is made very hard to debug as the ingest step is completed successfully...

Things that are nice:
- If you are _not too far_ from standard data science packages, the environment is a bonus! Most standard data science libraries are there already, and adding one or two with conda is pretty simple. (Discovering what you need to add is less nice - see pain point above)

Questions:
- See data in default storage. It would be nice to have easy (any?) way to view the data being written into and out of the steps of a pipeline.
- Is there a preferred way to iterate/develop pipelines. Integration with VS Code would be nice, but has some issues (as I mentioned in pain points).