---
layout: post
title:      "Working with Genetic Variant Classification Data"
date:       2019-04-18 21:44:18 +0000
permalink:  working_with_genetic_variant_classification_data
---


For Module 3, we were asked to solve a classification problem using a publicly available dataset. I wanted to see if I could apply the techniques that I have been learning to datasets that I was unfamiliar with. I figured that this would stretch me to think about ways to select and tune models without domain knowledge to inform my decisions. 

I chose to work on the Genetic Variant Classification dataset from Kaggle. The dataset was not involved in many competitions, so there is little available material for me to look at how other data scientists have approached it. On the plus side, I knew that I'd be able to make a solid contribution toward it with my project. You can find the dataset here: [Gene Variant Classification on Kaggle](https://www.kaggle.com/kevinarvai/clinvar-conflicting/version/3).

I have published a public kernel with my Jupyter notebook on Kaggle here: [Predicting Conflicting Gene Variant Classification](https://www.kaggle.com/iguanaonastick/predicting-conflicting-gene-variant-classification?scriptVersionId=13092023)
## Overview

In this dataset, we are given multiple genetic variants and various properties of each. Geneticists at different laboratories rated these variants based on their perceived clinical significance, with ratings ranging from Benign to Pathogenic. The target variable is whether the raters have clinical classifications that are concordant, meaning that they are in the same clinical category. About a quarter of the classifications are conflicting, while about three quarters are concurrent. 

![Genetic Variant Classification Overview](https://i.imgur.com/vO3XoxT.png)

## Scrubbing the Data

The dataset has about 65,000 observations with 45 features. Given that I understand little about what the features represent (even though they are properly documented in the Kaggle dataset), I knew that I had my work cut out for me cleaning the dataset. This became even more apparent when I realized that most of the variables were categorical, and I would need to make a number of difficult decisions about how to process them. I leaned on the little I remembered about genetics from freshman Biology and did the best I could, relying more on data science principles than on domain knowledge for most decisions. For example, I dichotomized some variables that had one dominant class and multiple infrequent classes, whereas someone with more domain knowledge could have made more fine-grained categories.

I have often heard that 80% of a data scientist's work is cleaning data. I thought that was hyperbole until I started on this project. I had to strategically take breaks so that I didn't go crazy as I worked methodically through each feature in the dataset. However, in the end, I was happy with the squeaky clean dataset I produced for analysis. This exercise gave me great sympathy for geneticists, who must deal with far messier data.

## Exploring the Data

One challenge that I encountered while exploring the dataset was that I was unfamiliar with how to describe the relationship between two categorical features. A Spearman correlation works fine for count variables or ordinal variables, but not for nominal/categorical ones. With a bit of Googling, I found a statistic that turned out to be quite useful: Cramer's V. It measures the association between categorical variables on a 0 to 1 scale by looking at how often different pairs of values co-occur. You can find more info about it here: [Cramer's V](https://en.wikipedia.org/wiki/Cram%C3%A9r%27s_V)

This blog post provided the inspiration for using Cramer's V with this dataset and for the visualization below: [The Search for Categorical Correlation](https://towardsdatascience.com/the-search-for-categorical-correlation-a1cf7f1888c9)

Here is the Python code from that blog post for Cramer's V:

```
def cramers_v(x, y):
    confusion_matrix = pd.crosstab(x,y)
    chi2 = ss.chi2_contingency(confusion_matrix)[0]
    n = confusion_matrix.sum().sum()
    phi2 = chi2/n
    r,k = confusion_matrix.shape
    phi2corr = max(0, phi2-((k-1)*(r-1))/(n-1))
    rcorr = r-((r-1)**2)/(n-1)
    kcorr = k-((k-1)**2)/(n-1)
    return np.sqrt(phi2corr/min((kcorr-1),(rcorr-1)))
```

And here is the Python codeI wrote for the heatmap visualization, using a list of categorical features orig_feat_cat:

```
# Number of categorical features
num_feat = len(orig_feat_cat)

# Empty matrix to populate
cat_corr_arr = np.empty((num_feat, num_feat))
for i, row in enumerate(orig_feat_cat):
    for j, col in enumerate(orig_feat_cat):
        cat_corr_arr[i, j] = cramers_v(features[row], features[col])

# Plot heatmap with seaborn
plt.figure(figsize=(16, 14))
sns.heatmap(cat_corr_arr,
            vmin=0,
            vmax=1,
            cmap='YlGnBu',
            xticklabels = orig_feat_cat,
            yticklabels = orig_feat_cat,
            annot=np.round(cat_corr_arr, 2))
						
```

And here was the result:

![Categorical Variable Association Heatmap](https://i.imgur.com/I4Oudy8.png)

Using this, I was able to identify some variables that were highly associated with other variables in the dataset, so I dropped them before starting with modeling.

## Modeling

For this project, I chose to use the F1 metric for evaluating my models. I did so of the unbalanced nature of the outcome, and the importance of identifying gene misclassifications even though they are uncommon. The F1 score allows me to ensure that recall is not sacrificed for accuracy.

I tried using a number of modeling approaches: Naive Bayes, Random Forest, AdaBoost with Decision Trees, and XGBoost. Surprisingly, Naive Bayes turned out to be the best approach. Given the large number of categorical variables, which I one-hot encoded, I found Bernoulli Naive Bayes to be the most effective. The F1 score for this model (after tuning) was 0.44, compared to 0.25 for random guessing. This isn't great, but it is an improvement.

## Interpretation

I generated a classification report to better understand how the model was working. While the model had decent recall, it had a high chance of classifying concurrent classifications as conflicting ones.

[Classification Report](https://i.imgur.com/miRfGFH.png)

The feature probabilities from the model yeilded three insights into the features that have the biggest discrepancies between classes:

* Gene variant is present in other genetic databases
* Gene variant is associated with an unnamed disease
* Gene variant is associated with hereditary cancer risk

These insights could be useful for directing research on gene variants that are likely to be difficult to classify.

## Conclusion

This project gave me a good taste for what it is like to come in as a data scientist and work an a wholly unfamiliar problem. Ultimately, I believe that there is more room for improving the model, possibly by changing the evaluation metric or by cleaning the data differently. That said, I am not surprised that the messiness in our genes might make it hard for us to make any predictions about them.



