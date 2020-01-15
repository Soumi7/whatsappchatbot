nmt-chatbot
===================

Table of Contents
-------------
1. [Introduction](#introduction)
2. [Setup](#setup)
3. [Custom summary values (evaluation)](#custom-summary-values-evaluation)
4. [Standard vs BPE/WPM-like (subword) tokenization, embedded detokenizer](#standard-vs-bpewpm-like-subword-tokenization-embedded-detokenizer)
5. [Rules files](#rules-files)
6. [Tests](#tests)
7. [More detailed information about training a model](#more-detailed-information-about-training-a-model)
8. [Utils](#utils)
9. [Inference](#inference)
10. [Importing nmt-chatbot](#importing-nmt-chatbot)
11. [Deploying chatbot/model](#deploying-chatbotmodel)
12. [Demo chatbot](#demo-chatbot)
13. [Changelog](#changelog)

Introduction
-------------

nmt-chatbot is the implementation of chatbot using NMT - Neural Machine Translation (seq2seq). Includes BPE/WPM-like tokenizator (own implementation). Main purpose of that project is to make an NMT chatbot, but it's fully compatible with NMT and still can be used for sentence translations between two languages.

The code is built on top of NMT but because of lack of available interfaces, some things are "hacked", and parts of the code had to be copied into that project (and will have to be maintained to follow changes in NMT).

This project forks NMT. We had to make a change in our code allowing the use of a stable TensorFlow (1.4) version. Doing so allowed us also to fix some bug before official patch as well as do couple of necessary changes.



Setup
-------------

Steps to setup project for your needs:
It is *highly* recommended that you use Python 3.6+. Python 3.4 and 3.5 is likely to work in Linux, but you will eventually hit encoding errors with 3.5 or lower in a Windows environment.

If you want to use exactly what's in tutorial made by Sentdex, use v0.1 tag. There are multiple changes after last part of tutorial.

 1. ```$ git clone --recursive https://github.com/daniel-kukiela/nmt-chatbot```  
    (or)  
    ```$ git clone --branch v0.1 --recursive https://github.com/daniel-kukiela/nmt-chatbot.git``` (for a version featured in Sentdex tutorial)
 2. ```$ cd nmt-chatbot```
 3. ```$ pip install -r requirements.txt``` TensorFlow-GPU is one of the requirements. You also need CUDA Toolkit 8.0 and cuDNN 6.1. (Windows tutorial: https://www.youtube.com/watch?v=r7-WPbx8VuY  Linux tutorial: https://pythonprogramming.net/how-to-cuda-gpu-tensorflow-deep-learning-tutorial/)
 4. ```$ cd setup```
 5. (optional) edit settings.py to your liking. These are a decent starting point for ~4GB of VRAM, you should first start by trying to raise vocab if you can. 
 6. (optional) Edit text files containing rules in the setup directory.
 7. Place training data inside "new_data" folder (train.(from|to), tst2013.(from|to), tst2013(from|to)). We have provided some sample data for those who just want to do a quick test drive.
 8. ```$ python prepare_data.py``` ...Run setup/prepare_data.py - a new folder called "data" will be created with prepared training data
 9. ```$ cd ../```
 10. ```$ python train.py``` Begin training





Tests
-------------

Every rules file has related test script. Those test scripts might be treated as some kind of unit testing. Every modification of rules files might be checked against those tests but every modification should be also followed by new test cases in those scripts.

It's important to check everything before training new model. Even slight change might break something.

Test scripts will display check status, checked sentence and eventually check result (if different than assumed).





setup/prepare_data.py:

 - walks thru files placed in "new_data" folder - train.(from|to), tst2012.(from|to)m tst2013(from|to)
 - tokenizes all sentences (based on settings and internal rules)
 - for "train" files - builds vocabulary files, makes them unique and saves up to the number of entities set in setup/settings.py file, unused vocab entities (if any - depends on settings) will be saved to separate files

train.py - starts training process


Running:
 
Whenever a model is trained, `inference.py`, when directly called, allows to "talk to" AI in interactive mode. It will start and setup everything needed to use trained model (using saved hparams file within the model folder and setup/settings.py for the rest of settings or lack of hparams file).

For every question will be printed up to a number of responses set in setup/settings.py. Every response will be marked with one of three colors:

 - green - first one with that color is a candidate to be returned. Answers to the color passed blacklist check (setup/response_blacklist.txt)
 - orange - still proper responses, but with lower than maximum score
 - red - improper response - below threshold

Steps from the question to the answers:

 1. Pass question
 2. Compute up to number of responses set in setup/settings.py
 3. Detokenize answers using either embedded detokenizer or rules from setup/answers_detokenized.txt (depending on settings)
 3. Replace responses or their parts using additional rules
 4. Score responses
 5. Return (show with interactive mode) responses

It is also possible to process a batch of the questions by simply using command redirection:

    python inference.py < input_file

or:

    python inference.py < input_file > output_file

It is possible to pass specified checkpoint as a parameter to use inference using that checkpoint, for example:

    python inference.py translate.ckpt-1000



Importing nmt-chatbot
-------------

The project allows being imported for the needs of inference. Simply embed folder within your project and import (notice: change folder name in a way it will not include dash (`-`) character - you can't import module with that character in it's name), then use:

    from nmt_chatbot.inference import inference

    print(inference("Hello!"))

(where `nmt_chatbot` is a directory name of chatbot module)

inference() takes one parameter:

 - `question` (required)

For a single question, function will return dictionary containing:

 - answers - list of all answers
 - scores - list of scores for answers
 - best_index - index of best answer
 - best_score - score of best answer

Score:

Every response starts with `score['starting_score']` value from `setup/settings.py` (10 by default) and is further modified by various checks. Please refer to `score` section of `setup/settings.py` for more details and settings. Final score is next used to pick "best" response.

With a list of questions, the function will return a list of dictionaries.

For every empty question, the function will return `None` instead of result dictionary.



Deploying chatbot/model
-----------------------

Whether model is trained, there might be a need to export only files necessary for interference (for example to be embeded and imported it other project).

That might be achieved automatically by using `prepare_for_deployment` utility:

    cd utils
    python prepare_for_deployment.py

Script will create `_deployment` folder inside project's root directory and copy all necessary files depending on my current settings.





Changelog
---------

### Master
- New response scoring engine (work in progress, suggestions and code improvements are welcome)
- Showing score modifiers in 'live' inference mode
- Fixed 'learning bpe' stage of 'prepare_data' speed issue (it's multiple times faster now)
- Improved 'prepare training set' stage of 'prepare_data' speed (should run about 1/3rd faster)
- Fixed model paths - pathhs are relative now, so model can be easily moved between different paths or even machines
- Fixed info about importing project as a module
- Updated README
- Added changelog
- Added table of contents
- Added passing checkpoint name as a parameter for inference
- Added deployment script
- Added ability to cache some `prepare_data` common steps for multiple script run
- Added epoch-based training
- Added custom decaying scheme (epoch-based)
- Added ability to return own evaluation values (will be plotted in TensorBoard)
- Updated `NMT` fork (fixed `train_ppl` graph, added evaluation outputs saved to a separate files, added ability to pass custom evaluation callback function)
- Merged latest `NMT` changes into our fork
- Various fixes and other small improvements

### v0.2
- BPE/WPM-like tokenizer
- Updated NMT
- Enabled NMT's SMP detokenizer with our embedded BPE/WPM-like tokenizer
- Fixed issue with paths on Linux and MacOS machines
- Improved pair testing utility
- Fixed command for tag cloning
- Various fixes and other small improvements, improved readme file


