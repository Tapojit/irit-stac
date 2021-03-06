This is the STAC codebase.

## Prerequisites

1. git (to keep up with educe/attelo changes)
2. Anaconda or miniconda (`conda --version` should say 2.2.6 or higher)
3. STAC corpus (released separately)

## Installation (basics, development mode)

Both educe and attelo supply `requirements.txt` files which can be
processed by pip.

1. Linux users: (Debian/Ubuntu)
   (NB: this step may be obsoleted by requiring conda)

        sudo apt-get install python-dev libyaml-dev

2. Fetch the irit-stac code if you have not done so already

        git clone https://github.com/irit-melodi/irit-stac.git
        cd irit-stac

3. Create a sandbox environment.  We assume you will be using Anaconda
   or miniconda.  Once you have installed it, you should be able to
   create the environment with

        conda env create

   If that doesn't work, make sure your anaconda version is up to date,
   and the conda bin directory is in your path (it might be installed in
   `/anaconda` instead of `$HOME/anaconda`).

4. Switch into your STAC sandbox

        source activate irit-stac

   Note that whenever you want to use STAC things, you would need to run
   this command.

5. Install the irit-stac code in development mode.
   This should automatically fetch the educe/attelo dependencies.

        pip install -r requirements.txt

   At this point, if somebody tells you to update the STAC code, it
   should be possible to just `git pull` and maybe
   `pip install -r requirements.txt` again if attelo/educe need to be
   updated. No further installation will be needed.

6. Install NLTK data files

        python setup-nltk.py

7. Link the STAC corpus in (STAC has not yet been released, so here
   the directory "Stac" refers to the STAC SVN directory)

        ln -s /path/to/Stac/data data


## Full installation (Toulouse/Edinburgh)

You only need to do this if you intend to use the `irit-stac parse`
or `irit-stac serve` command, i.e. if you're participating in
discourse parser experiments or integration work between the
parsing pipeline and the dialogue manager.

1. Do the basic install above.

2. Download [tweet-nlp][tweet-nlp] part of speech tagger
   and put the jar file (ark-tweet- in the
   lib/ directory (ie. on the STAC SVN root at the same level as
   code/ and data/).

3. Download and install corenlp-server (needs Apache Maven!)

        cd irit-stac
        mkdir lib
        cd lib
        git clone https://github.com/kowey/corenlp-server
        cd corenlp-server
        mvn package

## Usage (Toulouse)

Running the pieces of infrastructure here should consist of running
`irit-stac <subcommand>` from the STAC SVN root.

Folks (likely in Edinburgh) who just want to run the parsing pipeline
server should skip this section and go to "Usage (Edinburgh)" instead.

### Basics

Using the harness consists of two steps, gathering the features, and
running the n-fold cross validation loop

    irit-stac gather
    irit-stac evaluate

If you stop an evaluation (control-C) in progress, you can resume it
by running

    irit-stac evaluate --resume

The harness will try to detect what work it has already done and pick
up where it left off.

### Configuration

There is a small configuration module that you can edit
in `stac/harness/local.py`

It lets you control things such as which corpora to run on,
which decoders and learners to try, and how to do feature
extraction.

It tries to be self-documenting.

### Standalone parser

You can also use this infrastructure to parse new soclog files,
using models built from features you have collected.

    irit-stac gather
    irit-stac model
    irit-stac parse code/parser/sample.soclog /tmp/parser-output

### Scores and reports

You can get a sense of how things are going by inspecting the various
intermediary results:

1. output files: Outputs for any decoders in a fold that happen to
   have finished running (for a given fold N, see
   `TMP/latest/scratch-current/fold-N/output.*`),

2. fold reports : At the end of each fold, we will summarise all of
   the counts into a simple Precision/Recall/F1 report for attachment
   and labelling (for a given fold N, see
   `TMP/latest/scratch-current/fold-N/reports-*`),

3. full reports: If we make it through the entire experiment, we will
   produce a cross-validation summary combining the counts from all
   folds and several other things
   (`TMP/latest/eval-current/reports-*`).

### Cleanup

The harness produces a lot of output, and can take up potentially a lot
of disk space in the process.  If you have saved results you want to
keep, you can run the command

    irit-stac clean

This will delete *all* scratch directories, along with any evaluation
directories that look incomplete (no scores).

### Output files

There are two main directories for output:

* The `data/SNAPSHOTS` directory is meant for intermediary results that
you want to save. You have to copy files into here manually (more on
that later).
Because this directory can take up space, it does not feel quite right
to dump it on the public GitHub repo. We'll need to think about where to
store our snapshots later (possibly some IRIT-local SVN?).

* The `TMP` directory is where the test harness does all its work.  Each
`TMP/<timestamp>` directory corresponds to a set of feature files
generated by `irit-stac gather`.  For convenience, the harness will
maintain a `TMP/latest` symlink pointing to one of these directories.

Within the each feature directory, we can have a number of evaluation
and scratch directories. This layout is motivated by us wanting to
suppport ongoing changes to our learning/decoding algorithms
independently of feature collection. So the thinking is that we may
have multiple evaluations for any given set of features. Like the
feature directories, these are named by timestamp (with
`eval-current` and `scratch-current` symlinks for convenience).

* scratch directories: these are considered relatively ephemeral
  (hence them being deleted by `irit-stac clean`). They contain
  all the models and counts saved by harness during evaluation.

* eval directories: these contain things we would consider more
  essential for reproducing an evaluation. They contain the
  feature files (hardlinked from the parent dir) along with the
  fold listing and the cross-fold validation scores. If you hit
  any interesting milestones in development, it may be good to
  manually copy the eval directory to SNAPSHOTS, maybe with a
  small README explaining what it is, or at least a vaguely
  memorable name. This directory should be fairly self-contained.

## Usage (Edinburgh)

First make sure that the standalone parser works for you

    irit-stac parse code/parser/sample.soclog /tmp/parser-output

The parsing pipeline server has the same function as the standalone
parser but accepts inputs and sends outputs back over a network
connection

    irit-stac server --port 7777

Note that if you launch

    irit-stac server --port 7777 --incremental

The server will assume that every connection is *appending* to an
input in progress, and will generate a new output based on the
extended input (you'll have to restart the server for new inputs)


[tweet-nlp]: http://www.ark.cs.cmu.edu/TweetNLP/
