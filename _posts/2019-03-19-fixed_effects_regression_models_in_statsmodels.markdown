---
layout: post
title:      "Fixed Effects Regression Models in Statsmodels"
date:       2019-03-19 17:01:57 +0000
permalink:  fixed_effects_regression_models_in_statsmodels
---


In this blog post, I describe how I used pandas and statsmodels to implement a fixed effects regression model: a useful but counterintuitive type of regression model. I will also walk through the proper interpretation of the main coefficient of interest from this model. Hopefully, you will finish this post feeling comfortable with implementing such a model yourself, and understanding when/why you might want to do so.

## Background

I have just finished my second data science project as part of the Flatiron Data Science course. The project uses the Northwind Traders database to test various hypotheses related to the company's sales. It was a good exercise to learn how to use Python to extract data from relational databases and use them for hypothesis testing.

Coming from a social science background, I have used regression models to test a fair number of hypotheses. These models are very flexible, and can be used to conduct a wide variety of statistical tests. I tend to use regression as my default framework for hypothesis testing, which I also did for this project.

Using regressions for hypothesis testing does have some challenges. It can be difficult to correctly specify regression models when there are complex relationships among variables in the dataset. Moreover, as you add more features to your regression model, it becomes more challenging to provide accurate and understandable interpretations to coefficients in the model.

## What Are Fixed Effects Models?

Before going into the specifics, it's worth clarifying a bit of terminology. Sadly, the term "fixed effects" has been used to describe two different types of regression models. Among statisticians, it describes all models where parameters are fixed, i.e. not treated as random. Among economists, it describes a specific type of heirarchical model (i.e. observations belong to multiple groups) and means for each group are fixed. Here I use the term according to this latter meaning.

What does it mean to say that the group means are fixed? Let's take an example from education. Say we want to use students' 4th grade test scores to predict their 5th grade test scores. I could run a simple linear regression with these two variables to describe this relationship. The coefficient on 4th grade scores would tell me what difference in 5th grade scores is associated with a single point difference in 4th grade scores. 

However, if I know what schools the students are enrolled in, I can use that information to answer a more nuanced question with a fixed effects model. Using school ID as a categorical variable, I would create dummies for each school (leaving one out to avoid perfect multicollinearity). I woud include these dummies in my model. What this would do is "soak up" the variation in 5th grade scores that exists among schools. Some schools are higher performing on average, while others are lower performing - this process would control for those differences.

In this context, the coefficient on students' 4th grade scores gets a different interpretation. Rather than describing an overall relationship for all students in your dataset, the coefficient describes the relationship *controlling for differences in mean 5th grade scores across schools*. This means that the coefficient *represents a weighted average of school-level coefficients*, where the school-level coefficients capture the difference in 5th grade scores associated with a single point difference in 4th grade scores *for students in the same school*.

What is the benefit of this interpretation? It allows you to distinguish between relationships that exist within groups. It could be the case that having a high 4th grade score predicts having a high 5th grade score only because the student attends a good school, but that all students in that school perform well on the 5th grade test regardless of their 4th grade score. We would not notice this if we just used a simple linear regression.

Note that I have omitted an important discussion about centering. How you center the variables has a big impact on the interpretation of the coefficients. See a nice overview here: [Centering in Multilevel Regression](http://web.pdx.edu/~newsomj/mlrclass/ho_centering.pdf)

## Region Match Analysis with Northwind Traders Data

I will now take this out of the theoretical and into the applied using my project for Flatiron. In the dataset, we have information about orders submitted by customers and the employees who processed the orders. My question is the following: do employees who come from the same geographical region as their customers tend to get more orders from those customers than from customers in other regions? The theory is that when there is a match in customer and emplyee region, the employee could be better able to serve the customer because of familiarity with the region, and that this would translate into customers being more satisfied and submitting more orders.

### Simple Linear Regression

Here is the result of a simple linear regression, using a dummy variable indicating employee-customer region match to predict the number of orders received from customers.

```
from statsmodels.formula.api import ols

#Simple Linear Regression

f = 'Cust_Ord_Count ~ Region_Match'
model = ols(formula=f, data=df3).fit()
model.summary()
```

![Simple Linear Regression Results](https://i.imgur.com/1dr2eSJ.png)

The coefficient on the Region_Match dummy variable is 1.3763 with a p-value of 0.041, which is significant at the 0.05 level. This means that we can reject the null hypothesis - that there is no relationship between region match and order count - in favor of the alternative hypothesis - that there is such a relationship. Specifically, I estimate that when employees and customers come from the same region, customers tend to submit 1 or 2 more orders, on average.

### Fixed Effects Regression

This finding leaves me with a question, though. Could it be that some employees just get more orders than others, regardless of region match? If they do, and if those employees also happen to have more region matches, then we might observe the relationship above, even if those employees don't do any better with customers from their region. This is exaclt the sort of question fixed effects models help to resolve: Do employees see greater numbers of orders from their customers in their region than their customers in other regions? I implement a fixed effects regression to answer this below.

```
#Fixed Effects Regression

#Create dataset with dummies for each employee, leaving one out
fe_df = pd.get_dummies(df3, columns=['Employee_Id'], drop_first=True)

#Create a string with the names of each Employee_Id column
emp_id_col_list = fe_df.columns[6:]
emp_id_string = '+'.join(emp_id_col_list)

#Run regression, including Employee_Id dummies
f = 'Cust_Ord_Count ~ Region_Match + ' + emp_id_string
model = ols(formula=f, data=fe_df).fit()
model.summary()
```

![Fixed Effects Regression Results](https://i.imgur.com/v6fJvoe.png)

The coefficient on the Region_Match dummy variable is 0.4501 with a p-value of 0.576, which is not significant at the 0.05 level. This means that we cannot reject the null hypothesis - that there is no relationship between region match and order count within employees - in favor of the alternative hypothesis - that there is such a relationship. Thus, we would not expect that employees serving customers from their region would get more orders from those customers

## Conclusion

It may seem counterintuitive that the simple linear regression produced a significant result for Region_Match while the fixed effects regression did not. The two models answer similar questions, after all. However, this demonstrates the value of fixed effects regressions. The relationships between and across groups can be quite different - sometimes they are even in opposite directions! You can see an example of this in the graphic below.

![Within vs Between Group Relationships](https://www.researchgate.net/profile/Jennifer_Feitosa/publication/282557082/figure/fig1/AS:283875679981568@1444692644245/Illustration-that-the-Between-Group-X-Y-Relationship-Can-Differ-from-the-Within-Group-X_W640.jpg)

Fixed effects regression models can be challenging to implement and interpret, but they are very helpful for disentangling within versus between group relationships. There are many extensions of this model that wone can use to further disaggregate relationships this way. To dig in deeper, I'd recommend reading any of the books out there about multilevel modeling or heirarchical models, such as this one: [Heirarchical Linear Models](https://us.sagepub.com/en-us/nam/hierarchical-linear-models/book9230)
