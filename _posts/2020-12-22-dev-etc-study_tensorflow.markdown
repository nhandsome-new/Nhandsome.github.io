---
layout: post
title:  "[Study] Introduction to Tensorflow"
subtitle:   "Subtitle - Coursera, Intoroduction to Tensorflow"
categories: dev
tags: nlp
comments: true
---
## Describe
> Summarizing Coursera lectures, [`Introduction to Tensorflow`](https://www.coursera.org/learn/intro-tensorflow/home/welcome)<br>

## 目次
- [Week 1](#jump1)
- [Week 2](#jump2)
  - [Load CSV](#jump3)
  - [Data Preprocessing](#jump4)
<br><br><br>

## <a name="jump1">Week 1</a>
　**What are tensor and tensorflow?**<br>
　Pipeline that calcuate tensors and learn models predicting what we want
<br><br>
**Tensor : N-dimensional array of data**
- tf.constant produces constant tensors and tf.Variable tensors can be modified.
- stock, slice, reshape : change tensor dimensions
- tf.Variable : The values can be changed by tf.Variable.assign


```python
## tensorflow opperation can take naive python types and Numpy array as operands
a_py = [1,2]
b_np = np.array([3,4])
c_tf = tf.constant([5,6])

temp = tf.multiply(tf.add(a_py, b_np),c_tf)
temp.numpy()


## Example of Linear Regression with Tensorflow
## low level code
## Loss Function (MES)
## Reduce_mean helps to calculate means by several options (row, col, total)
def loss_mes(X, Y, w0, w1):
  Y_hat = X * w0 + w1
  errors = (Y_hat - Y)**2
  return tf.reduce_mean(errors)

## Gradient Function
## tf.GradientTape()
## GradientsTape records loss data into tape in order to calculate the gradient
## gradient(loss_values, for which layer)
def compute_gradients(X, Y, w0, w1):
  with tf.GradientTape() as tape:
    loss = loss_mes(X, Y, w0, w1)
  return tape.gradient(loss, [w0, w1])
s
## we can check results of compute_gradients
## define shape of model and loss function
## get gradient
## let the model learn with the gradients
## assign_sub is a function such as 'a -= b' 
X = tf.constant(range(10), dtype=float(32))
Y = 2 * X + 10 

w0 = tf.Variable(0.0)
w1 = tf.Variable(0.0)

dw0, dw1 = compute_gradients(X, Y, w0, w1)

## Set steps and learning_rate
STEPS = 1000
LEARNING_RATE = 0.2
MSG = 'STEP {step} - loss: {loss}, w0: {w0}, w1: {w1}'

for step in range(0, STEP + 1):
  dw0, dw1 = compute_gradients(X, Y, w0, w1)
  w0.assign_sub(dw0 * LEARNING_RATE)
  w1.assign_sub(dw1 * LEARNING_RATE)

  if step%100 == 0:
    loss = loss_mes(X, Y, w0, w1)
    print(MSG.format(step=step, loss=loss, w0=w0.numpy(), w1=w1.numpy()))
```

<br><br><br>


## <a name="jump2">Week 2</a>
　**Design and Build a Tensorflow Input Data Pipeline**<br>

**tf.data API**
- Build complex input pipelines for multiple data types
  - Image data(RGB), Text Data and so on
  - Needed to be vectorizated
  - tf.data API can help to control these various data types
- Handle large amount of data and perform complex transfortation.<br>

Features(featcols) are columns using for making model, needed to be numeric data(in One vector) to calculate.<br>

How to deal with `Many csv files`  with `many columns` having `various types`??
- Ont Hot : Change categorial data to One Hot vectors
- Bucketize : Split a numeric data into categories based on range
- Embedding : Represent data as a lower-dimensional, dense vector<br>

<br><br><br>

### <a name="jump3">Load CSV</a>
　Example for loading data from CSV files into a tf.data.Dataset. Assume target values are [0,1] lists.
```python
import numpy as np
import tensorflow as tf

## Download data from TRAIN_DATA_URL and TEST_DATA_URL
TRAIN_DATA_URL = ''
TEST_DATA_URL = ''
train_file_path = tf.keras.utils.get_file("train.csv", TRAIN_DATA_URL)
test_file_path = tf.keras.utils.get_file("eval.csv", TEST_DATA_URL)

## Set one column the model is intedned to predict
LABEL_COL = 'target'
LABEL = [0,1]

## Definite get_dataset function based on tf.data.experimental.make_csv_dataset.
def get_dataset(file_path, **kwargs):
  dataset = tf.data.experimental.make_csv_dataset(
    file_path,
    batch_size = 5,
    label_name = LABEL_COL,
    na_value = '?',
    num_epochs = 1,
    ignore_errors = True,
    **kwargs
  )
  return dataset
```
> We can get dataset from CSV files by using get_dataset() function. Each dataset is
- having 5 data (batch_size)
- using LABEL_COL as the target (label_name)
- recognizing '?' as NA data
- [`and so on`](https://www.tensorflow.org/api_docs/python/tf/data/experimental/make_csv_dataset)

This function will use all available columns and the names from CSV file, but if we need to use specific columns then can use 'column_names' and 'select_columns' option.

```python
SELECTED_COLUMNS = ['target','age','sex','class']
temp_dataset = get_dataset(train_file_path, 
  selected_columns = SELECTED_COLUMNS)

## Take n numbers of dataset
n = 5
temp_dataset.take(n)
```

<br><br><br>

### <a name="jump4">Data Preprocessing</a>
　A CSV file can contain variable types of data, so we need to convert from those mixed types to a fiexed length vectors before feeding the data into our model.

**Continuous Data**
Need to 
- set Default values
- pack them into a single column
- normalize

```python
## Select countinous data columns
SELECTED_COLUMNS = ['target','age','n_siblings_spouses','fare']

## set DEFAULTS values and make dataset function
DEFAULTS = [0, 0.0, 0.0, 0.0]
temp_dataset = get_dataset(train_file_path, 
  selected_columns = SELECTED_COLUMNS, 
  column_defaults = DEFUALTS)

## pack function will pack all the continuous columns
## tf.stack
def pack(features, label):
  return tf.stack(list(features.values()), axis=-1), label

## We can get n numbers dataset continuous data are packed
packed_dataset = temp_dataset.map(pack)
n = 1
packed_dataset.take(n)
```

Here a PackNumericFeatures class for handling numeric columns
```python
class PackNumericFeatures(object):
  def __init__(self, names):
    self.names = names

  def __call__(self, features, labels):
    numeric_features = [features.pop(name) for name in self.names]
    numeric_features = [tf.cast(feat, tf.float32) for feat in numeric_features]
    numeric_features = tf.stack(numeric_features, axis=-1)
    features['numeric'] = numeric_features

    return features, labels

## Select numeric feature columns
NUMERIC_FEATURES = ['age','n_siblings_spouses','parch', 'fare']

## Pack all of numeric feature and save in 'numeric' column
packed_train_data = raw_train_data.map(
    PackNumericFeatures(NUMERIC_FEATURES))
packed_test_data = raw_test_data.map(
    PackNumericFeatures(NUMERIC_FEATURES))
```

And continous data should be normalized
```python
import pandas as pd
## We could get information of dataset description from pd.describe()
## from whole of data
MEAN = np.array(desc.T['mean'])
STD = np.array(desc.T['std'])

## Make normalize function
def normalize_numeric_data(data, mean, std):
  return (data-mean)/std

##partial : when use a function with specific inputs
normalizer = functools.partial(normalize_numeric_data, mean=MEAN, std=STD)

## Make 'numeric' feature column 
## tf.feature_columns.numeric_column API will be run on each batch with normalizer_fn option
## it has information about MEAN and STD and normalize data of each batch
numeric_column = tf.feature_column.numeric_column('numeric', 
  normalizer_fn=normalizer, 
  shape=[len(NUMERIC_FEATURES)])
numeric_columns = [numeric_column]

## tf.keras.layers.DenseFeatures produces a dense tensor following given columns
numeric_layer = tf.keras.layers.DenseFeatures(numeric_columns)
numeric_layer(example_batch).numpy()
```

<br><br>

**Categorical Data**
Categorical data columns have contents which should be a one of set of options. 

```python
## set categorical data columns
CATEGORIES = {
    'sex': ['male', 'female'],
    'class' : ['First', 'Second', 'Third']
}

## create categorical_columns that will be feature columns of tensorflow model
categorical_columns = []

## make lists having categorical data information with tf.feature_column 
for feature, vocab in CATEGORIES.items():
  cat_col = tf.feature_column.categorical_column_with_vocabulary_list(
        key=feature, vocabulary_list=vocab)
  categorical_columns.append(tf.feature_column.indicator_column(cat_col))

## tf.keras.layers.DenseFeatures creates a dense tensor based on given columns
categorical_layer = tf.keras.layers.DenseFeatures(categorical_columns)
categorical_layer(example_batch)
```

<br><br>

**Combined preprocessing layer**
We created two feature column collections above and can make input layer that will extract and preprocess both input types

```python
preprocessing_layer = tf.keras.layers.DenseFeatures(categorical_columns + numeric_columns)
preprocessing_layer(example_batch).numpy()[1]

## preprocessed dataset using 2nd batch data, such as
## ['sex','class','age','n_siblings_spouses','parch', 'fare']
## [1,0,22,1,34,43]
```