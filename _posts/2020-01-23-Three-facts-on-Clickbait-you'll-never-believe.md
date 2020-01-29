---
layout: post
title:  "3 Things on Clickbait You'll Never Believe"
date:   2020-01-23
categories: jekyll update
---

  
__Clickbaiting__ is the intenctional act of over-promising or otherwise misrepresenting what you’re going to find when you read a story on the web.

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/clickbait_analysis/master/images/clickbt_ex_1.png" width="200">
<img src="https://raw.githubusercontent.com/LorBordin/clickbait_analysis/master/images/clickbt_ex_2.png" width="180">
<img src="https://raw.githubusercontent.com/LorBordin/clickbait_analysis/master/images/clickbt_ex_3.png" width="190">
</div> 
 
Headlines, like the one above, work on our brain in a very subtle way. Their aim is to exploit the _curiosity gap_, providing just enough information to make the reader curious, but not enough to satisfy her/his curiosity without clicking through to the linked content.

Over time, clickbaiting can get potentially dangerous. 
Studies show that not only readers get addicited to captivating headlines, but they  prevent people to come up with their own mental models.
In addition to these, clickbait articles increseangly often do not deliver any substance and because of their catchy titles, usually very emotionals, they tend to make readers less objective.

So if they are so distruptive in nature, why are they so used? Of course, there's only one reason for that: __money__.  

On-line news websites are competing for your attention because they need it to thrive in today’s economy. Our interactions on these platforms generate invaluable data without which the machine intelligence that makes them so intuitive and personalized would fail to exist.

All this comes at the expense of the depletion of real journalism. Important and renowned newspapers have starded to produce clickbait contents as well.

To better understand this phenomenon in journalism I have decided to carry out a quantitative analysis to see how much clickbait affects ordinary journalism. 
I've selected headlines from some of the most famous US and UK newspapers.
Data show the raising of the clickbait phenomenon over time.

Stay tuned, the results I found will shock you!



## A Clickbait Detector

Classifing tousands of headlines would not be possible without the help of the artificial intelligence. 
Therefore, I've firstly developed a __Clickbait detector__ that uses __machine learning__  to classify headlines.
 
As training data I've used the __AOSSIE Click Bait Dataset__, publicly available on [Kaggle](https://www.kaggle.com/ad6398/aossie-click-bait-dataset): it consists in a huge set of labelled news titles, collected from variuos websites like _Buzzfeed_ or The _New York Times_.

Inspired by this research paper, [Stop Clickbait: Detecting and Preventing Clickbaits in Online News Media](https://arxiv.org/abs/1610.09786), I've used some NLP tools to extract the following training features from each headline:

- lemmatised words count,
- lenght of title,
- stopwords ratio,
- contractions ratio.

Then, I've trained a Random Forest Classifier. 
If you're courious to see how the detector works, take a look at my code! Here's the [GitHub repo](https://github.com/LorBordin/clickbait_analysis).


## The Raising of Clickbait Over Time

I begun my analysis studing the trend of clickbait headlines over time. I've used as database the one provided by the _New York Times_ (NYT) Archive API. 
It provides data of all the articles that appeard in the NYT since 1851. 

The API makes collecting headlines relatively easy. 
I've collected all the those entries with the attribute `type_of_material` labelled as `News`, then I've classified them with my clickbait detector. 
Here's some titles classified as clickbait with the highest probability:

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/clickbait_analysis/master/images/clckbt_headlines.png" width="600">
</div>
<br>

Very good! They all have the typical structure of the clickbait headlines like or _numbers at the begin_, they contain catchy phrases as _that will make/keep you_,  and so on. 
These are instead some of the titles with the lowest probability of being clickbait:

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/clickbait_analysis/master/images/no_clckbt_headlines.png" width="600">
</div>
<br>

Interestingly, also some headlines with the lowest probability start with a number, a feature usually associated to clickbait titles.
However it is clear these titles are not clickbaits.

Ok, let's see now how the number of clickbait titles varies in time 
This plot shows the ratio of clickbait over the whole number of articles published each month since 2008.

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/clickbait_analysis/master/images/clckbt_ratio_VS_time.png" width="780">
</div>
<br>

The proportion of clickbait articles stays constant around 10% up to 2016, then it suddenly raises and in just 3 years it doubles.
In 2019 about 25% of news headlines were clickbait!


### Web vs Printed Version

A confirmation of the fact that clickbait is mainly a web phenomenon can be found in looking at the printed version of the headlines that appear on the web page. It can be found, in the NYT Archive API under the attribute `print_headline.

Web and print versions are often different, and the analysis shows that __
__many times headlines in the printed version are not clickbait, while on the web they are__. Here some examples:

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/clickbait_analysis/master/images/web_vs_print.png" width="750">
</div>
<br>

Take a look at the notebook [NYT_analysis](https://github.com/LorBordin/clickbait_analysis/blob/master/NYT_analysis.ipynb) in my GitHub repo for an explanation on how I did the analysis and play with it.

## Clickbait on UK journals

To be finished ...

<!--- ### A closer look into The Guardian --->
