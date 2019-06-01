# MSCS Project

This repository overviews an Artificial Intelligence design project which I completed while completing my Master's in Computer Science at Georgia Tech. The source code for this project is not publically available, in compliance with the Georgia Institute of Technology Academic Honor Code<sup><a href="https://policylibrary.gatech.edu/student-affairs/academic-honor-code">[1]</a></sup>.

Instead, I have substituted my raw code with a guided walkthrough of the project itself, the steps I went through to complete it, and the skills I acquired by solving it.

# Overview

Machine learning offers a number of methods for classifying data into discrete categories, such as k-means clustering<sup><a href="https://en.wikipedia.org/wiki/K-means_clustering">[2]</a></sup>. Decision trees<sup><a href="https://en.wikipedia.org/wiki/Decision_tree">[3]</a></sup> provide a structure for such categorization, based on a series of decisions that led to separate distinct outcomes.

In this assignment, I worked with decision trees to perform binary classification according to some decision boundary. I built and then trained decision trees which were capable of solving useful classification problems. Finally, I implemented strategies to measure their performance.

<p align="center"><img width="420" height="220" src=images/decision-tree.png></img></p>
<div align="center"><b>Fig 1. Basic Decision Tree</b></div>

# Hand Crafting

I was given some sample data to build decision trees by hand. In the following table, the "Datum" column represents a sample, and the "A1" through "A4" represent attributes that can be true or false for a given sample. Finally, the "y" column indicates if the data belongs to a group known as "y". If data doesn't belong to "y", it belongs to "x".

Using these attributes and samples, I built a decision tree to determine if data belongs in "y" or "x".

| Datum	| A1  | A2  | A3  | A4  |  y  |
| ----- | --- | --- | --- | --- | --- |
| x1    |  1  |  0  |  0  |  0  |  1  |
| x2    |  1  |  0  |  1  |  1  |  1  |
| x3    |  0  |  1  |  0  |  0  |  1  |
| x4    |  0  |  1  |  1  |  0  |  0  |
| x5    |  1  |  1  |  0  |  1  |  1  |
| x6    |  0  |  1  |  0  |  1  |  0  |
| x7    |  0  |  0  |  1  |  1  |  1  |
| x8    |  0  |  0  |  1  |  0  |  0  |

I selected tests to be as small as possible (in terms of attrributes), and chose to break ties among tests with the same number of attributes by selecting the one which classified the greatest number of examples correctly. If multiple tests had the same number of attributes and classified the same number of examples, I broke the tie using attributes with lower index numbers.

# Initial Evaluation

I used a Confusion Matrix<sup><a href="https://en.wikipedia.org/wiki/Confusion_matrix">[4]</a></sup> to help evaluate the performance of my decision tree. I focused on three performance metrics:

* Precision ( True Positives / (True Positives + False Positives) )
* Recall ( True Positives / (True Positives + False Negatives) )
* Accuracy ( Correct Classifications / Total Number of Examples )

Due to the limited supply of data, and the very small set of attributes to examine, the performance was only a bit better than random guessing. And while it was certainly better, it varied too wildly to yield meaningful results. This was as far as I was tasked to take this simple data set though, and instead, began working with much more interesting and complex data in the core of the assignment.

# The Core

After working with my simple data set, it was time to move on to a large .csv file with tens of thousands of entries. My goal was to achieve over 70% test accuracy after training, and I began working towards this.

As the size of my training set grew, it became impractical to build the trees by hand. I needed a procedure to automatically construct the trees for me!

The algorithm I implemented looked something like this:

1. Check for base cases:

   1a. If all elements of a list are of the same class, return a leaf node with the appropriate class label.
   
   1b. If a specified depth limit is reached, return a leaf labeled with the most frequent class.
2. For each attribute alpha: evaluate the normalized gini gain gained by splitting on attribute `alpha`.
3. Let `alpha_best` be the attribute with the highest normalized gini gain.
4. Create a decision node that splits on `alpha_best`.
5. Repeat on the sublists obtained by splitting on `alpha_best`, and add those nodes as children of this node

Additionally, it's worth noting that I switched my evaluation metric to Gini impurity<sup><a href="https://en.wikipedia.org/wiki/Cross-validation_(statistics)#k-fold_cross-validation">[5]</a></sup>. I hoped to use this to assess overall information gain, and it seemed more promising than measuring entropy like some of my classmates chose to do.

After implementing this, I wrote a function to produce classifications for a list of features once the decision tree had been built.

# Validation

My performance quickly reached my goal, almost too quickly, and I became suspicious of my success. I realized it was very likely I was overfitting my training data, and performing well when analyzing data that had been used for training-- but not as well otherwise.

In general, reserving part of your data as a test set can lead to unpredictable performance. A serendipitous choice of your training or test split could give you a very inaccurate idea of how your classifier performs. I overcame this limitation by using k-fold cross validation<sup><a href="https://en.wikipedia.org/wiki/Cross-validation_(statistics)#k-fold_cross-validation">[6]</a></sup>.

In my fold generation implementation, I split the dataset into k equal subsections. I iterated on each of the k samples, and reserved that sample for testing, while using the other k-1 samples for training. Averaging the results of each fold gave me a MUCH more consistent idea for how the classifier was doing across the data as a whole.

# Leveraging Random Forests

The decision boundaries drawn by decision trees are very sharp, and fitting a decision tree of unbounded depth to a list of training examples almost inevitably leads to overfitting. In an attempt to decrease the variance of my classifier, I used a technique called 'Bootstrap Aggregating'<sup><a href="https://en.wikipedia.org/wiki/Bootstrap_aggregating">[7]</a></sup> (often abbreviated as 'bagging').

A Random Forest is a collection of decision trees, built as follows:
For every tree I'm going to build:
   1. Subsample the examples provided (with replacement) in accordance with a provided example subsampling rate.
   2. From the sample in the first step, choose attributes at random to learn on (in accordance with a provided attribute subsampling rate).
   3. Fit a decision tree to the subsample of data I've chosen (to a certain depth).
   
Classification for a random forest is then done by taking a majority vote of the classifications yielded by each tree in the forest after it classifies an example.

I implemented fitting for the decision tree as described above, and began using forests to classify a given list of examples. I began testing with a forest of 5 trees, and a depth limit of 5, as well as an example and attribute subsample rate of 0.5.

# Challenge Classification

After achieving the initial desired 70% success rate on unseen data, I was tasked with overcoming more challenging data. I now had to exceed 80% accuracy on an even larger and more complex dataset.

I previously mentioned that I was using GINI impurity as an evaluation metric. The biggest hangup I had in this project was relying too much on evaluating only one criteria, based on the similarity of attributes.

I realized that lots of information could be learned by analyzing the data I had in different ways!

For example, I could calculate:

* Standard deviation of attribute values from the mean
* Sum of all attribute values
* Mean of attribute values
* Similarity of absolute value of attribute values
* Etc.

This greatly enhanced the amount of information I was getting from analyzing my data, while only trivially increasing computation. (Basic arithmetic operations.)

Additionally, I implemented my own version of a technique known as Gradient Boosting<sup><a href="https://en.wikipedia.org/wiki/Gradient_boosting">[8]</a></sup>, which helped increase the speed and effectiveness of my decision trees.

I called upon my random forest with a variety of different paramaters, made many tweaks over countless iterations, and finally was able to surpass my goal! I achieved a final stable classification success rate of approximately 91%.

### Vectorization

As a small note, one of the most interesting parts of this assignment was improving the speed so I could use greater depth and quantity in my forests. I was able to implement vectorization using numpy in Python. Essentially, I outsourced a variety of my arithmetic operations to faster programming languages, and performed this in mass on huge quantities of numpy arrays. The performance gain from this was what pushed me from barely not passing to greatly exceeding my goal!

# Reflection

For me, the greatest lesson from this assignment was comprehending that there's always more than meets the eye in a data set. I spent well over 80% of my time working on this assignment only evaluating one metric of the data-- similarity of raw data attributes.

I simply did not consider that performing operations on the provided data would actually yield meaningful information gain. However, it did, and this is a core aspect of machine learning which opened my eyes to many challenges I would approach in the future.

It may not seem like "evaluating the standard deviation of attribute values in a sample" means *anything* to the naked eye. But to a classifier, this is extremely important, and it shows how abstract data sets like these mimic the myriad of complex and unseen correlations which appear in real-world classification problems.
