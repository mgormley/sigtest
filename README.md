# Significance Tests for NLP

This script can be used to run two types of significance tests: (1)
the paired permutation test and (2) the bootstrap test. It is designed
specifically to support a variety of Natural Language Processing data
formats and metrics (see the two lists below). 

The paired permutation test follows
[Yeh (2000)](https://arxiv.org/abs/cs/0008005) which uses the test as
described in Noreen (1989, Sec. 3A.3). For a concise description of
the bootstrap test see
[Berg-Kirkpatrick & Klein (2012)](http://www.aclweb.org/anthology/D/D12/D12-1091.pdf).

The implementation caches sentence-level sufficient statistics for the
corresponding metric, so that the resulting significance tests are
fairly fast for a large number of samples (e.g. 1 million).  In
general, the paired permutation test tends to run a bit faster than
the bootstrap.

## Installation

The `sigtest` script provided here is simply a wrapper for `java
edu.jhu.nlp.eval.SignificanceTests`. Java users may prefer to directly
invoke
[SignificanceTests](https://github.com/mgormley/pacaya-nlp/blob/master/src/main/java/edu/jhu/nlp/eval/SignificanceTests.java).


1. Install [Apache Maven](https://maven.apache.org/install.html)
   - Mac OS X: `brew install maven`
   - Ubuntu: `apt-get install maven`
2. Clone this repository.
3. Run `./sigtest` to test installation and view usage information.

## Usage

Command line options:
```bash
usage: ./sigtest --gold <arg> --pred1 <arg> --pred2 <arg> --type <arg> --metric <arg> 
       [--maxNumSentences <arg>] [--numSamples <arg>]  [--reportOut <arg>] 
       [--sigTest <arg>] [--skipPunct <arg>]
       
--gold <arg>              [Required] The path to the gold data
--pred1 <arg>             [Required] The path to the predicted data (set 1)
--pred2 <arg>             [Required] The path to the predicted data (set 2)
--type <arg>              [Required] The dataset type
--metric <arg>            [Required] The evaluation metric
--sigTest <arg>           Type of significance test
--maxNumSentences <arg>   The maximum number of sentences
--numSamples <arg>        The number of samples to use for the significance test
--reportOut <arg>         File to which to which reports should be written.
--skipPunct <arg>         Whether to skip punctuation (dependency accuracy only)
```

The `--sigtest` option can be one of `{PAIRED, BOOTSTRAP}`. The
integer `--numSamples` specifies the number of permuations or
bootstrap samples and defaults to 2^20. Various metrics and datasets
are supported (see options below).


Metrics (specified via `--metric`):

- `POS_ACC`: Accuracy of POS tags
- `DP_ACC`: Accuracy of unlabeled syntactic dependencies
- `SRL_P, SRL_R, SRL_F1`: Precision, recall, and F1 of semantic roles (dependency-based)
- `REL_P, REL_R, REL_F1`: Precision, recall, and F1 of relations 
- `NER_R, NER_P, NER_F1`: Precision, recall, and F1 of BIO-style chunking or NER


Data formats (specified via `--type`):

- `PTB`: Penn Treebank
- `CONLL_2002`: CoNLL-2000 and CoNLL-2002
- `CONLL_2003`: CoNLL-2003
- `CONLL_X`: CoNLL-X
- `CONLL_2008`: CoNLL-2008
- `CONLL_2009`: CoNLL-2009
- `CONCRETE`: Concrete (a la. http://hltcoe.github.io/)
- `SEMEVAL_2010`: SemEval 2010
- `JSON`: Pacaya's concatenated JSON

## Example Run

In this example, we first download the test data from the CoNLL-2000
shared task, followed by the predicted output from two systems. The
predicted output is simply a column of labels so we convert it to
CoNLL-2000 format with the paste / awk commands.

```bash
echo "Downloading CoNLL-2000 test data and two shared task outputs.
curl -O http://www.cnts.ua.ac.be/conll2000/chunking/test.txt.gz
gunzip test.txt.gz
curl -O http://www.cnts.ua.ac.be/conll2000/chunking/results/13335dej.txt
curl -O http://www.cnts.ua.ac.be/conll2000/chunking/results/13941koe.txt
paste test.txt 13335dej.txt | awk '{if($2=="") printf("\n"); else printf("%s\t%s\t%s\n", $1, $2, $4); }' > pred1.txt
paste test.txt 13941koe.txt | awk '{if($2=="") printf("\n"); else printf("%s\t%s\t%s\n", $1, $2, $4); }' > pred2.txt
```

Now we run `sigtest` on the downloaded data.

``` bash
./sigtest --gold test.txt --pred1 pred1.txt --pred2 pred2.txt --type CONLL_2002 --metric NER_F1
```

After about 30 seconds, the output of the first call to sigtest should be as below. 
```
0        INFO  ReporterManager - Reporter manager initialized with reportOut=null and useLogger=true
3        INFO  AnnoSentenceReader - Reading gold data of type CONLL_2002 from test.txt
170      INFO  AnnoSentenceReader - Num gold sentences: 2012
171      INFO  AnnoSentenceReader - Num gold tokens: 47377
172      INFO  AnnoSentenceReader - Longest sentence: 70
174      INFO  AnnoSentenceReader - Average sentence length: 23.547216699801194
175      INFO  AnnoSentenceReader - Reading pred1 data of type CONLL_2002 from pred1.txt
284      INFO  AnnoSentenceReader - Num pred1 sentences: 2012
284      INFO  AnnoSentenceReader - Num pred1 tokens: 47377
285      INFO  AnnoSentenceReader - Longest sentence: 70
285      INFO  AnnoSentenceReader - Average sentence length: 23.547216699801194
285      INFO  AnnoSentenceReader - Reading pred2 data of type CONLL_2002 from pred3.txt
347      INFO  AnnoSentenceReader - Num pred2 sentences: 2012
347      INFO  AnnoSentenceReader - Num pred2 tokens: 47377
348      INFO  AnnoSentenceReader - Longest sentence: 70
348      INFO  AnnoSentenceReader - Average sentence length: 23.547216699801194
2223     INFO  SignificanceTests - Score on dataset 1: 0.920889186306698
2223     INFO  SignificanceTests - REPORT: score1 = 0.920889186306698
2224     INFO  SignificanceTests - Score on dataset 2: 0.9197077235517032
2224     INFO  SignificanceTests - REPORT: score2 = 0.9197077235517032
WARNING: pseudo random number generator is not thread safe
SEED=123456789101112
35538    INFO  SignificanceTests - p-value (paired permutation): 0.2511441696699432
35538    INFO  SignificanceTests - REPORT: p-value-ppt = 0.2511441696699432
```

The reported values show that score1 (F1 on pred1.txt) is 92.08%,
score2 (F1 on pred2.txt) is 91.97%. The sigtest script defaults to a
paired permutation test with 2^20 permutations and the result is a
p-value of 0.25 -- clearly indicating that there is not a significant
difference.


Using the command below, we can run the bootstrap test `--sigTest
BOOTSTRAP` and fewer samples `--numSamples 100000` (i.e. 0.1 million
instead of ~1 million),

``` bash
./sigtest --gold test.txt --pred1 pred1.txt --pred2 pred2.txt --type CONLL_2002 --metric NER_F1 --sigTest BOOTSTRAP --numSamples 100000
```

The output will give a similar p-value as shown below.

``` bash
...
7048     INFO  SignificanceTests - p-value (bootstrap permutation): 0.24996
7048     INFO  SignificanceTests - REPORT: p-value-bts = 0.24996
```


## Contributing

If you'd like to add support for additional data formats, metrics, or
statistical significance tests please submit a pull request on
[Pacaya NLP](https://github.com/mgormley/pacaya-nlp) with
modifications to
[edu.jhu.nlp.eval.SignificanceTests](https://github.com/mgormley/pacaya-nlp/blob/master/src/main/java/edu/jhu/nlp/eval/SignificanceTests.java).


