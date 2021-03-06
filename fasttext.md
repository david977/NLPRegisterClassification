## Getting data to train format:
```bash
pramah@biarc:~/CORE_registers_correct_classes/single_labels/fastText$ head -n 17589 ../train_file.txt > core.train
```
## Getting data to test format:
```bash
 head -n 2210 ../test_file.txt > core.test
```
## Our First Supervised Classifier:
```bash
 ./fasttext supervised -input ../core.train -output model_core
```
## Predicting on Devset data:
```bash
 ./fasttext predict model_core.bin -
 ```
 Tried a few labels from the devset data. Using our devset data, we get following result:
 ```bash
 ./fasttext test model_core.bin ../core.valid
N       2486
P@1     0.422
R@1     0.211
 ```
## Preprocessing in FastText:
 ```bash
 cat data/core.preprocessed.txt | tr '[:upper:]' '[:lower:]' | sed -e "s/'/ ' /g" -e 's/"//g' -e 's/\./ \. /g' -e 's/<br \/>/ /g' -e 's/,/ , /g' -e 's/(/ ( /g' -e 's/)/ ) /g' -e 's/\!/ \! /g' -e 's/\?/ \? /g' -e 's/\;/ /g' -e 's/\:/ /g' |  sed 's/\,//g' | sed 's/\.//g' > data/core.preprocessed6.txt
 
 Shuffling dataset:
perl -MList::Util -e 'print List::Util::shuffle <>' data/core.preprocessed6.txt > data/core.preprocessed7.txt

 ```
 ## Modeling with Processed data:
  ```bash
 pramah@biarc:~/CORE_registers_correct_classes/single_labels/fastText$ ./fasttext supervised -input ../data/core.processed.train -output model_processed
Read 21M words
Number of words:  266229
Number of labels: 26
Progress: 100.0% words/sec/thread:  755646 lr:  0.000000 avg.loss:  2.058319 ETA:   0h 0m 0s
 ```
 ## Testing the accuracy of the model: 
   ```bash
 pramah@biarc:~/CORE_registers_correct_classes/single_labels/fastText$ ./fasttext test model_processed.bin ../data/core.processed2.test
N       2212
P@1     0.457
R@1     0.457

   ```
   
  With Only epoch of 25, we get following:
  
 ```bash
  pramah@biarc:~/CORE_registers_correct_classes/single_labels/fastText$ ./fasttext test model_processed.bin ../data/core.processed2.test
N       2212
P@1     0.664
R@1     0.664
  ```
 And with learning rate of 1.0:
```bash
N       2212
P@1     0.688
R@1     0.688
```
With a learning rate of 1 and epoch 25:
```bash
N       2212
P@1     0.691
R@1     0.691

```
With learning rate 1, epoch 25 and wordNgram of 2:
```bash
N       2212
P@1     0.697
R@1     0.697

```
## This is how Hierarchical softmax was carried out:
```bash
 ./fasttext supervised -input ../data/core.processed.train -output model_processed -lr 1.0 -epoch 25 -wordNgrams 2 -bucket 200000 -dim 50 -loss hs
```
The accuracy was little less though:
```bash
./fasttext test model_processed.bin ../data/core.processed2.test                          N       2212
P@1     0.634
R@1     0.634
```
  There is no much difference when wordNgram used is 3.
 ## This is how we do autotune validation:  
```bash 

pramah@biarc:~/CORE_registers_correct_classes/single_labels/fastText$ ./fasttext supervised -input ../data/core.processed.train -output model_core -autotune-validation ../data/core.processed2.test
```

## This is how to test on the model 
```bash
./fasttext test model_core.bin ../data/core.processed2.test

Progress: 100.0% Trials:    5 Best score:  0.647830 ETA:   0h 0m 0s
Training again with best arguments
Read 21M words
Number of words:  266229
Number of labels: 26
```

Following was the result: 
```bash
N       2212
P@1     0.646
R@1     0.646

```
