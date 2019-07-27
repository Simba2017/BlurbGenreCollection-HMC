# BlurbGenreCollection_Classification: Hierarchical text classification of categories using capsule networks

This page contains the implementation of several neural network architectures (CNN, LSTM, capsule network) designed for a multi-label text classification task with an underlying hierarchical structure.


The neural networks take as an input a collection of lists of tokens of fixed length, that are referenced by their ID and, as well as a collection of label sets. In case a hierarchy is provided to the program, label correction methods can be applied to create consistent predictions in respect to  the underlying hierarchy. 
Furthermore, the final layers of the neural networks can be pre-initilized with label co-occurrence, as described in [Baker et al.](http://aclweb.org/anthology/W17-2339).

The neural networks and an additional SVM baseline were applied to the BlurbGenreCollection_EN datset, consisting of book blurbs and their respective hierarchically structured writing genres. The datset can be downloaded on the [Language Technology page of the Universität Hamburg](https://www.inf.uni-hamburg.de/en/inst/ab/lt/resources/data/blurb-genre-collection.html).

This page also consists of methods to evaluate the performance of each classifier. An in-depth description and evaluation of the neural networks on the BlurbGenreCollection datset can be found [here](https://www.inf.uni-hamburg.de/en/inst/ab/lt/teaching/theses/completed-theses/2018-ba-aly-blurbs.pdf).


# System Requirement

The system was tested on Debian/Ubuntu Linux with a GTX 1080TI and TITAN X.

# Installation

1. Clone repository: 

  ```
  https://github.com/Raldir/BlurbGenreCollection_Classification.git
  ```
  
2. Install a dataset

   1. Either the BlurbGenreCollection-Dataset:

      ```
      cd BlurbGenreCollection_Classification && wget https://fiona.uni-hamburg.de/ca89b3cf/blurbgenrecollectionen.zip && unzip blurbgenrecollectionen.zip -d datasets
      ```
  
   2. Or install your own Dataset:
   
       The abstract class `loader_abstract` needs to be extended by your custom class that loads your dataset. Please adjust the return values of the methods to match the descriptions. The method `load_data_multiLabel()` should return a list of three sets: train, dev and test. Each collection is a list of tuples with each tuple being `(String, Set of Strings)` for the text and its respective set of labels. 
       
The method `read_relations()` only needs to be implemented if a hierarchy exists. It should contain two sets -- the first consists of relation-pairs `(parent, child)` as Strings and the second set contains genres that have neither a parent nor a child. Furthermore, replace the following line with the name of your new loader_class: ` data_helpers.py: Line 15`. For further reference, please take a look at `loader.py` which loads the BlurbGenreCollection dataset.
      Finally, `read_all_genres` stores co_occurences in a file to make the loading process quicker -- if the dataset changes please adjust the name so that the correct co_occurences are being loaded (only for label hierarchy relevant).
         

  
3. Install project packages:

```
pip install -r code/requirements.txt
```
 
 4. Further packages needed:
 ```
pip install stop-words
python -m spacy download en
python -m spacy download en_core_web_sm
 ```
 
5. Install word embeddings for the English language, e.g.:
```
mkdir resources && cd resources && wget https://dl.fbaipublicfiles.com/fasttext/vectors-wiki/wiki.en.vec
``` 
We recommend to put them into a ./resources folder. Please ensure to adjust the path and filename in case you decide to use different embeddings/path.

# Hierarchical Multi-label Classification

Running the main.py will run the complete Pipeline if in train mode: Loading the data, preprocessing and training the classifier. 
The preprocessed data is stored in the resources folder, to save time in sequential runs. Same applies to the computation of the embedding matrix, which is stored for a fixed sequence length.
However, for the first execution, please run the holdout_train mode (or any mode that uses the dev set) so that the stored preprocessed token collections are generated for each split.  

| Option |  Description | Default|
|--------|-------------|---|
| --mode | Mode, e.g. train on dev using holdout, train on dev and train for testing(train_final) | train_holdout |
| --classifier | Select between CNN, LSTM and capsule | capsule |
| --lang | Datset to be used | EN |
| --level| Max Genre Level of the hierarchy| 1 |

The level setting can only be used if the program is provided with a hierarchy, otherwise the networks handle the data as a traditional multi-label classification task.

General Settings:

| Option |  Description | Default|
|--------|-------------|---|
| --sequence_length | Maximum sequence imput length of text | 100 |
| --epochs | Number of epochs to train the classifier | 60 |
| --use_statc | Whether the embedding layer should not be trainable | False |
| --use_early_stop |Uses early stopping during training | False |
| --batch_size |Set minibatch size | 32 |
| --learning_rate |The learning rate of the classifier | 0.0005 |
| --learning_decay |Whether to use learning decay, 1 indicates no decay, 0 max.| 1 |
| --init_layer |Whether to initialize the final layer with label co-occurence.| False |
| --iterations |How many classifiers to be trained, only relevant for train_n_models_final | 3 |
| --activation_th |Activation threshold of the final layer | 0.5 |
| --adjust_hierarchy |Postprocessing hierarchy correction | None|
| --correction_th |Threshold for threshold-label correction method | False |

Please note, that `--init_layer, --correction_th --adjust_hierarchy` are only usable, if the hierarchy of a dataset is given as input as well.


Capsule settings:

| Option |  Description | Default|
|--------|-------------|---|
| --dense_capsule_dim |Dimensionality of capsules on final layer| 16 |
| --n_channels | Number of capsules per feature map | 50 |

LSTM settings:

| Option |  Description | Default|
|--------|-------------|---|
| --lstm_units | Number of units in the lstm | 700 |

CNN settings:

| Option |  Description | Default|
|--------|-------------|---|
| --num_filters | Number of filters for each window size | 500 |


*Example:*  
`python3.5 main.py --mode train_holdout --classifier cnn --lang EN --sequence_length 100 --learning_rate 0.001 --learning_decay 1 
`


For further inquries: 5aly@informatik.uni-hamburg.de
