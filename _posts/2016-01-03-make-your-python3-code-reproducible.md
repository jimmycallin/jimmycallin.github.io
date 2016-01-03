---
layout: post
title: Make your scientific Python 3 code reproducible by deterministic hashing
---

As of Python 3.3, the built in `hash` function is randomized by default -- [for very good reasons](https://mail.python.org/pipermail/python-announce-list/2012-March/009394.html)! In short, having a deterministic hash function in production code is quite a security risk. We can see the effect of a randomized hash function by outputting the content of a set on multiple runs:

    python -c "print(set(['A','B','C']))"
    {'A', 'C', 'B'}
    python -c "print(set(['A','B','C']))"
    {'B', 'C', 'A'}


For scientific code, though, this could potentially be a problem. Since we want to make sure our work is as reproducible as possible, any stochastic elements will potentially make it difficult to regenerate previous experimental results. 

As an example, I have some code that outputs a dictionary of features in the following format:

    Number=Sing|Definite=Ind|Gender=Com|Case=Nom

The problem arises whenever I rerun the output and want to see if any differences arises between runs. Since the order of the dictionary output is determined by the hashing of the original keys, I cannot use e.g. `git diff` to compare the previous commited output with the current one. 

If you have complete control and knowledge of your environment, your best bet is to either use `OrderedDict` and its counterparts whenever the sorting order cannot be assumed, or to simply sort the items of your dictionary ahead of iteration.

Most of the time, we don't have this luxury. I quite often rely upon separate libraries and scripts for my work that I cannot in good faith start modifying. Instead, we could ahead of running the experiments set the environment variable [`PYTHONHASHSEED`](https://docs.python.org/3.3/using/cmdline.html#envvar-PYTHONHASHSEED) to an integer value.

    export PYTHONHASHSEED=0
    python run_experiment.py


At last, I want to emphasize: this should only be done with the purpose of having a non-random ordering of sets and dictionaries! If you actually want a deterministic hash value of e.g. a string, you should almost always use [hashlib](https://docs.python.org/3/library/hashlib.html).
