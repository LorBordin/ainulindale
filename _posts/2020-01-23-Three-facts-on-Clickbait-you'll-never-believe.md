---
layout: post
title:  "3 Things on Clickbait You Won't Believe"
date:   2020-01-23
categories: jekyll update
---

  
__Clickbaiting__ is the intentional act of over-promising or otherwise misrepresenting what you’re going to find when you read a story on the web.

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/clickbait_analysis/master/images/clickbt_ex_1.png" width="200">
<img src="https://raw.githubusercontent.com/LorBordin/clickbait_analysis/master/images/clickbt_ex_2.png" width="180">
<img src="https://raw.githubusercontent.com/LorBordin/clickbait_analysis/master/images/clickbt_ex_3.png" width="190">
</div> 
 
Headlines, like the one above, affect our brain in a very subtle way. Their aim is to exploit the _curiosity gap_, providing just enough information to make the reader curious, but not enough to satisfy her/his curiosity without clicking through to the linked content.

Over time, clickbaiting can get potentially dangerous. 
Studies show that not only readers get addicted to captivating headlines, but they  prevent people to come up with their own mental models.
In addition to these, clickbait articles increasingly often do not deliver any substance and because of their catchy titles, usually very emotional, they tend to make readers less objective.

So if they are so disruptive in nature, why are they so used? Of course, there's only one reason for that: __money__.  

News websites are competing for our attention because they need it to thrive in today’s economy. Our interactions on these platforms generate invaluable data without which the machine intelligence that makes them so intuitive and personalised would fail to exist.
All this comes at the expense of the depletion of real journalism. Important and renowned newspapers have started to produce clickbait contents as well.

To better understand this phenomenon I have carried out an analysis, quantifying the amount of clickbait contents in modern journalism. 
I have collected headlines from some of the most famous US and UK newspapers.
Data show the raising of the clickbait phenomenon over time.

All the details of my analysis are available in my [GitHub repo] (https://github.com/LorBordin/clickbait_analysis). 



## A Clickbait Detector

Classifying thousands of headlines would not be possible without the help of the artificial intelligence. 
Therefore, before any analysis, I have firstly developed a __Clickbait detector__ that uses __machine learning__  to categorise headlines.
 
I trained the algorithm on the __AOSSIE Click Bait Dataset__ (publicly available on [Kaggle](https://www.kaggle.com/ad6398/aossie-click-bait-dataset)): it consists in a huge set of labelled news titles, collected from various websites like _Buzzfeed_ or The _New York Times_.

Inspired by the study published on this paper, [Stop Clickbait: Detecting and Preventing Clickbait in Online News Media](https://arxiv.org/abs/1610.09786), I have used some NLP tools to extract the following training features from each headline:

- lemmatised words count,
- length of title,
- stopwords ratio,
- contractions ratio.

Then, I've trained a Random Forest Classifier. 
Take a look at my [repo](https://github.com/LorBordin/clickbait_analysis) to inspect the code .


## The Raising of Clickbait Over Time

I begun my analysis studying the evolution of clickbait headlines over time.  
The __New York Times__ (NYT) Archive API is the perfect tool that allow to easily collect the big amount of data needed.
It provides data of all the articles that appeared in the NYT since 1851. 
 
I've collected all the those entries with the attribute `type_of_material` labelled as `News`, then I've used the clickbait detector to quantify the ratio of clickbait headlines.
As a check that the detector works, here's some titles classified as clickbait with the highest probability:

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/clickbait_analysis/master/images/clckbt_headlines.png" width="600">
</div>
<br>

Notice their typical structure like __numbers at the begin of the title__, or __catchy  expressions as  _that will make/keep you_ , and so on.  
These are, instead, some of the titles with the least probability of being clickbait:

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/clickbait_analysis/master/images/no_clckbt_headlines.png" width="600">
</div>
<br>

Notice the second headline: it starts with a number, a features usually associated to clickbait. 
Nonetheless this is clearly not a clickbait headline as the classifier correctly claims.

Having verified the detector works, I have used it to evaluate the amount of clickbait headlines published each month, since 2008.  

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/clickbait_analysis/master/images/clckbt_ratio_VS_time.png" width="780">
</div>
<br>

The plot shows the percentage of clickbait headlines over time. 
It is interesting to notice that is stays constant until 2016, then it suddenly raises doubling its value in about 3 years.
In 2019 about 25% of news headlines were clickbait! 


### Web vs Printed Version

Clickbait headlines is mainly a web phenomenon. 
A confirmation of this can be found by comparing the printed version of the same headline that appears in the website.
This is made possible thanks to the attribute `print_headline` that many headlines in the NYT API posses.

The analysis shows that quite often __headlines are clickbait in the web version but  not in the printed one__. For instance,

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/clickbait_analysis/master/images/web_vs_print.png" width="750">
</div>
<br>

## A Common Trend?

The analysis on the NYT shows a sudden increase of the clickbait content in recent times.
Is this trend common in other newspapers as well, or is it specific of the NYT?
To answer this question I have collected headlines from 3 British news websites: __The Guardian__,  __BBC news__ and __The telegraph__, I have also included in my analysis the famous tabloid __Dailymail__.

Unfortunately, I'm not aware of any API, therefore I had to gently scrap the websites for about two weeks. 
This short period does not allow me to quantify the rising of the phenomenon over time, but I could just estimate the current amount of clickbait contents.

Overall, the  __percentage of clickbait headlines is about 20%__, comparable with the NYT. It seems that also in the Old Continent, newspapers use similar strategies to attract readers. 

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/clickbait_analysis/master/images/clckbt_ex_uk_journals.png" width="600">
</div>
<br>

Here's some example of some headlines classified as clickbait with the highest probability. 
The fourth entry does not look like clickbait, probably the "six" at the begin of the phrase is one of the causes why the classifier got wrong.

Even if on average the percentage of clickbaits is similar to the NYT one, it varies considerably among the individual websites, as it is highlighted in the following.

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/clickbait_analysis/master/images/clckbt_ratio_uk_j.png" width="600">
</div>
<br>

This is a very interesting plot: it shows that individual websites adopted different policies regarding clickbaits. Both _The Guardian_ and the _Dailymail_ seem to have a poor clickbait content if compared with the _BBC news_ and especially at _The Telegraph_, where the percentage grows up to 30%. 
However, it is important to mention that the Detector I trained and used, is well suited for "standard clickbait" (that is short titles, catchy phrases, presence of many abbreviations ...), but it can get in trouble if headlines become very long and verbose as in Dailymail.


## Conclusion

The analysis carried out points out the existence of a real emergency in the modern journalism. 
Bad journalism does not only affect hoax websites, whose only purpose is to attract people to get economic reward, but also famous and trusted news publications. 

To fight against this crisis, the developing of new tools like news filters that hide unwanted contents is
probably one of the most effective weapons we have at our disposal.
Together with new tools, however, educating people about the negative effects that clickbait have, not only on the media but on themselves as well, will be crucial. 
