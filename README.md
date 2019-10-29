# Causal selection 

This package has two interfaces:

(1) **score_vocab**. Given text, vocab, outcomes, and confounds, the algorithm scores each word according to how well it explains the outcome, _controlling for confounds_. 

(2) **evaluate_vocab**. Given text, vocab, outcomes, and confounds, this algorithm evaluates the overall ability of the entire vocab in explaining the outcome, _controlling for counfounds_.

This package is based on the following two papers:

1. _Deconfounded Lexicon Induction for Interpretable Social Science_ [(Pryzant, et al. 2019)](https://nlp.stanford.edu/pubs/pryzant2018lexicon.pdf)
2. _Interpretable Neural Architectures for Attributing an Ad’s Performance to its Writing Style_ [(Pryzant, et al. 2019)](https://nlp.stanford.edu/pubs/pryzant2018emnlp.pdf)



## Install

```
$ pip3 install causal-selection
```

## Test

You can use `test.py` and the included data to run an integration test.

```
$ python3 test.py
```
No output means the test passed. This file also contains example usage.

## Example

Let's say we have a file, `descriptions.csv`, which contains product descriptions for Nike and Addidas shoes:


| Description   | Brand   | Sales |
|---------------|---------|-------|
| buy now !     | addidas | 15    |
| nike shoes !  | nike    | 35    |
| fresh shoes ! | nike    | 32    |


We want to find the words that are most predictive of sales, controlling for brand in order to ignore words like "nike" which are merely a function of confounding circumstance rather than a part of the writing style. 

We can obtain a dictionary mapping individual words to their importance with `score_vocab`:

```
importance_scores = selection.score_vocab(
	vocab=['buy', 'now' '!', 'nike', 'fresh'],
	csv="descriptions.csv"
	name_to_type={
		'Description': 'input',
		'Brand': 'control',
		'Sales': 'predict',
	})
```

`importance_scores` will contain a list of `(word, score)` tuples.

If we want to evaluate the overal ability of our vocabulary's ability to make causal inferences about sales, we can use . `evaluate_vocab`:

```
informativeness = selection.score_vocab(
	vocab=['buy', 'now' '!', 'nike', 'fresh'],
	csv="descriptions.csv"
	name_to_type={
		'Description': 'input',
		'Brand': 'control',
		'Sales': 'predict',
	})
```
`informativeness` will be a float that reflects the vocabulary's abiltiy to predict sales, _beyond_ the brand's ability to predict sales.


### `score_vocab(vocab, csv="", delimiter="", df=None, name_to_type={})`

**Arguments**
* **vocab**: list(str). The vocabulary to use.  You can include **ngrams** by combining words with a space, e.g. `['a', 'b', 'a b']`.
* **csv**: str. Path to a csv of data. The column corresponding to your "input" variable needs to already be tokenized, and 
      each token is separated by whitespace.
* **delimiter**: str. Delimiter to use when reading the csv.
* **df**: pandas.df. The data we want to analyze over.
* **name_to_type**: dict. A mapping from column names to whether they are "input", "predict", or "control" variables. You can only have one "input" variable (the text).  You can have 1+ "predict" and 1+ "control" variables, and they can be categorical  (e.g. `['a', 'b', 'a']`) or continuous (e.g. `[1.0, 0.9, 0.1]`).
**Returns**
A mapping: outcome variable name => outcome variable class => a list of tuples containing vocab elements and their score (i.e. how important each feature is for that level of the outcome).


### `score_vocab(vocab, csv="", delimiter="", df=None, name_to_type={})`

**Arguments**
These arguments are all the same as `score_vocab()`. 

**Returns**
The _informativeness coefficient_ of the vocab, which measures the strength of the text's causal effects that can be attributed to the vocab. 

## Tips

* For a continuous variable X, give the algorithm _log(X)_ instead of just X.
* The algorithm is sensitive to hyperparameter settings (number of training steps, hidden dimension, etc). Try several different settings to get the best scores possible.
