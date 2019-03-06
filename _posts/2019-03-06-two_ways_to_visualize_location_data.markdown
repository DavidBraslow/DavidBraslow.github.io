---
layout: post
title:      "Two Ways to Visualize Location Data"
date:       2019-03-06 20:24:20 +0000
permalink:  two_ways_to_visualize_location_data
---


As part of the Flatiron Data Science course, I have just finished my first start-to-finish data science project. The project looks at predicting house sale prices using various features of houses for sale. I thought this was a great way to practice data cleaning, data exploration, and modeling.

## Location Data Overview

One set of variables included in the feature set were latitude and longitude. Together, these variables tell you the location of houses on a map. Very cool! I figure that these will be important for modeling house sale prices, since every real estate agent knows the famous saying about what matters when selling or buying a house: "Location, Location, Location!"

Of course, there's a lot more to a house's location than its latitude and longitude. Is it on a busy street? Is it near transportation? Does it have a waterfront view? Latitude and longitude alone won't tell us any of these things.

The dataset does fortunately include a feature indicating whether houses are on the waterfront. The dataset also includes a variable for zip code. So, these are not the only data that give information about location. It is not clear whether or not the latitude and longitude will be important for this analysis.

There are two questions that I had about the latitude and longitude data that I wanted to explore with visualizations. First, how are the houses spread out around King County? Second, are more valuable houses concentrated in certain areas? 

To answer these questions, I created two different visualizations using the latitude and longitude data. I created them in Seaborn, which provides nice looking visualizations without too much work. By comparing them side by side, you can see how they highlight two different findings, and how they both have strengths and weaknesses.

## Plot #1: Concentration of House Sales

The first I created using Seaborn's jointplot, which provides both a scatter plot and marginal distributions. I thought the marginal distributions would be helpful for understanding the density of the points (i.e. the density of the houses being sold), since overlapping points mask this density. It also helps to highlight sparser areas of the scatterplot, where there are a small but non-negligible numbers of houses.

```
sns.set(style="white")
ax = sns.jointplot(x = df['long'], y = df['lat'], s = 2, height = 8)
ax.set_axis_labels(ylabel='Latitude', xlabel='Longitude')
plt.subplots_adjust(top=0.95)
ax.fig.suptitle('Locations of House Sales')
plt.show()
```

![](https://imgur.com/HDbG4YG.png)

First, you can see quite a bit of interesting clustering in this graph. There seem to be a number of boundaries where houses are clustered - presumably these are defined by bodies of water, roads, or parks. We can see some of this in this image taken from Google Maps of the same county:

![](https://i.imgur.com/iDkgRwG.png)

Notice the island to the west. You can see some house sales from there in the plot to the left - pretty neat!

Second, we can see that most of the houses are clustered in certain denser neighborhoods. Presumably, these are the more densely populated areas. Due to the overlapping points it is hard to know exactly how many houses are there, but the visualization gives a good sense of the area covered by this dataset. The histograms on the sides are helpful for understanding just how dense the neighborhoods are.

Lastly, there are large portions of the plot with very few points. Presumably these are more rural areas. The few pooints that are on the map may be anomalies in the data, but more likely they represent the small number of properties in those areas that sold. The dataset only includes one year's worth of sales, so it is plausible that not many properties sold. during this time.

## Plot 2: Value of House Sales

Next, I wanted to understand how important location was for understanding the value of a house. Since the dataset includes sale prices, I decided to plat a similar graph, but use hues to show where more expensive houses were being sold. I noticed that the original sale price distribution was positively skewed, so I decided to use a log transformation before using it in the visualization.

Original Prices:

![](https://i.imgur.com/soekZdY.png)

Log Prices:

![](https://i.imgur.com/t9nZ5lQ.png)

The log prices look more normal, which should make for a more useful gradient on a linear scale. I chose to use a blue-to-green gradient, with the green highlighting more expensive houses. Why? Because green represents $$$! I also used the alpha parameter to add some transparency to the dots as a way to address the density problem.

```
fig, ax = plt.subplots(figsize=(9,9)) 
sns.scatterplot(x = 'long', y = 'lat', data = df,
                hue = 'log_price', palette='BuGn', alpha = .7,
                s = 6, ax = ax )
ax.set_ylabel('Latitude')
ax.set_xlabel('Longitude')
plt.subplots_adjust(top=0.95)
ax.set_title('House Sale Prices by Location')
plt.show()
```

![](https://i.imgur.com/sS22xSk.png)

You can see that this graph provides new information about the houses, in addition to their clustering. For example, you can see that there is a pocket of more expensive houses located toward the center. Presumably this is a more affluent neighborhood. You can also see that there are boundaries with more expensive houses. Some of these adjoin bodies of water, while others are next to parks. Understandable that properties at those locations would be worth more.

## Conclusion

There are many ways to represent data visually, and sometimes it can be valuable to use more than one approach to describe the same data. There are plenty of opportunities for improvement in both of these visualizations, but they should give you a useful starting off point for your own visualization efforts.



