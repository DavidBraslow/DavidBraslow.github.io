---
layout: post
title:      "Regression with Long Datasets in Pandas"
date:       2019-05-16 21:37:18 +0000
permalink:  regression_with_long_datasets_in_pandas
---


For my capstone project, I am taking a crack at the [LANL Earthquake Prediction Competition](https://www.kaggle.com/c/LANL-Earthquake-Prediction) on Kaggle. This competition aims to use acoustic data recorded by ground sensors to predict the timing of earthquakes in a laboratory experiment. The training dataset for this competition has over 600M rows representing acoustic measurements of the soil, and the target is the remaining time until the next earthquake occurs. Since the acoustic signal and the target are the only variables in the dataset, the datafile is not too large to process on my moderately-powerful PC. However, working with such a long dataset is a new challenge for me, and I thought I'd share some of what I learned while trying to wrangle it.

## Understanding the problem

The evaluation of models for this competition relies on multiple, smaller test datasets. There are ~2,600 test datasets with 150,000 rows each, representing a small acoustic snapshot.  The goal is to predict a single value representing the time until the next earthquake after the end of that snapshot. Therefore, my model has to be able to create a prediction on small subsets of the original train dataset. This helps narrow down the universe of possible solutions, because any solution has to work on small subsamples of acoustic data.

## Understanding the data

With a bit of data exploration, it is possible to see that the train data has some strange patterns in the target variable: time until next earthquake. We would expect the values of the target value to decrease linearly until there is an earthquake, and then spike upwards after hitting zero. However, the values do not decrease linearly as time passes, but rather in a step-wise linear fashion. 

[STEP CHART]

According to the creator of the Kaggle competition, this is a result of the acoustic measuring device: "The data is recorded in bins of 4096 samples. Withing those bins seismic data is recorded at 4MHz, but there is a 12 microseconds gap between each bin, an artifact of the recording device." This information could be useful for processing the training data, because I could process the bins of 4,096 samples separately. However, there is no information about the binning of the samples in the test dataset, so this technique would not be usable for making predictions for those datasets. I therefore have to treat all the acoustic data as if it is coming in a continuous stream.

## Chunking the data

In order to create a model that works on acoustic snippets with 150,000 rows, I need to cut the original train dataset into snippets of that size to feed into my model. In theory, I could use the train dataset to create 600M snippets, each one consisting of a single unique row and the following 149,999 rows. However, my computer is not powerful enough to run any sort of sophisticated models on 600M observations. I decided to try to create 10,000 snippets that I would use for modeling purposes.  

One thing that I decided was that I wanted my snippets to include as much variation as possible. I could have created 10,000 snippets easily using only the first 160,000 rows and the technique described above. However, this would mean that I am ignoring the vast majority of my training data. Since the dataset is extremely noisy, I want to make sure that my model performs well in all the conditions present there, not just the particular vibrations in the very start. I therefore want my snippets to span the entire dataset.

### Loading the dataset

Fortunately, the dataset is small enough that it only takes about 2 minutes to load the entire dataset into memory as a Pandas DataFrame. However, it turns out that loading the whole dataset into memory is the easy part. Accessing specific rows in that dataset in memory turns out to be surprisingly time consuming.

### Index-based slicing with Pandas

The traditional way to access a set of rows in a Pandas DataFrame is with the `.iloc` method. For example, I could create a snippet like so:

```
snippet = df.iloc[10000000: 10150000]
```

In theory, I could create a loop to generate 10,000 such snippets and call it a day. However, this requires the computer to index the dataset repeatedly from memory. The method for doing so starts at index 0 of the dataset and then goes down the index until it reaches the desired indices. As I started to access snippets further and further down, the time it took got longer and longer. As a result, I looked for alternative approaches.

### Creating a generator

Rather than using the whole dataset loaded in memory, I was able to use a TextFileReader object to read and load in small chucks of the train dataset at a time. I did this using the `iterator` and `chunksize` arguments of the Pandas method `read_csv`. This turned out to be 10 times faster than the index-based slicing!

### Feature engineering

After loading each snippet, I ran a number of feature engineering functions on it to summarize the contents. I then saved the results in a new dataset with one row per snippet - much more manageable!

## Conclusion

Smartly designing functions to use memory wisely is a luxury with normal size datasets, but is mandatory with large datasets. Poorly optimized algorithms will halt you in your tracks before you even get to the modeling stage. With this approach, I was able to proceed to the modeling stage relatively quickly.
