---
layout: post
title:  "Airbnb vs Booking"
date:   2020-03-16
categories: jekyll update
---

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/airbnb_vs_booking/master/images/holy_island.jpg" height="200" width="1000">
</div> 
<br>
<br>


Dreaming a holiday? A weekend escape? 
Nothing have been easier to plan nowadays, thanks to many online platforms that offer wonderful stayings and great experiences!

Few weeks ago, together with some friends, we decided to do a trekking along the wonderful [Northumberland coast](https://en.wikipedia.org/wiki/Northumberland_Coast) and explore some its magnificent castles.
Planning the holiday we were in doubt where to look for our stayings. 
Personally, I've always used _Airbnb_ but other friends were claiming _Booking.com_ is better, being more popular and thus having better options.
Personally, I had the opposite opinion.. So we made a bet!
But how to figure out which platform is more popular? 

Google provides a great tool for this: _Google Trends_.
Not only it knows about what's popular and what's not in different countries, but it can tell al lot about people's habit.
Besides looking for a answer for my bet, I've also decided to play a bit with is and figure out some people's habits on travels and holydays.



# The data

The Google trends website analyses the popularity of top search queries in Google Search across various countries and allows to compare different searches providing the historical data on the popularity of each query. 
To answer my question I've gathered the popularity data of the Airbnb and Booking.com platforms since 2008.
To easily download, access and analyse the data, I've used its [Unofficial API](https://pypi.org/project/pytrends/1.1.3/). 

__Important.__  Google Trends is a great tool for data analysis but in order to make reliable predictions it's crucial to understand what kind of data it provides. 
Google Trends  data are not a direct representation of the query’s search volume over time.
What it displays is the __relative popularity__ of a search query, normalised by the total searches of the geography and time range it represents.
This allows to compare the popularity of a query in different areas with different search volumes, but this also means that one cannot compare two different time series. 
Comparisons can only be made for the same time interval and only using the specific tool provided by Google Trends. 


# The Airbnb liftoff

The following plot shows the relative popularity between Airbnb and Booking.com. 
The first thing that stands out is a high correlation between the popularity of the two queries: both trends show, especially in the last years, a clear seasonality.

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/airbnb_vs_booking/master/images/hist_trend_UK.png" width="700">
</div> 

This oscillatory behaviour partially masks the underlying trends, nonetheless the plot undoubtedly answers my question. 
While at the begin of 2008 Airbnb - that had just started operating - couldn't compete with Booking then, from 2014 its popularity started to grow very fast and eventually surpassed Booking.com in 2017-2018. 

The plot, however tell us way more things than this. 
For instance we see that since 2015, there are two main peaks in the yearly popularity of both the queries. 
The highest happens in summer, but there is a second one, centred at the begin of the year that raises every year. 
Looks like that before 2015 a winter holiday was not so common as now.

Now, let's take a closer look at all the possible seasonalities.


# Yearly, Weekly and daily seasonalities

The previous plot is coarse grained,  data are sampled monthly. 
Selecting smaller time intervals we finer samplings, in this way also weekly and daily seasonalities can be detected. 
I have plotted all the seasonalities I found in the series of graphs below. 
Again there is a strong correlation between Airbnb and Booking.com queries: people google them in similar time slots.

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/airbnb_vs_booking/master/images/by_month.png" width="700">
</div> 

The first plot is about the popularity per month of the year. 
There are two peaks in the popularity distribution, the first centred in January and a second one that reaches its maximum in July. 
These are, of course the two main periods when people go on holiday.
During Autumn the both trends drop and reach their minimum in December.
Assuming the trends are directly correlated with how much the people travel, we can say that Autumn is the period of the year where people travel the less.

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/airbnb_vs_booking/master/images/by_week.png" width="700">
</div> 

The second plot is similar to the first but now popularity is by week of the year. 
This allows to resolve additional peaks related to specific periods of holidays.
For instance we can notice a small increase (especially on the Airbnb query) on the 7th week of the year - late February - is it due to Carnival?
Another local maximum happens to be on the 20th-21st week, this is surely due to the Bank Holiday of the 25th of May. 
Finally, there's a huge spike on the penultime week of the year: lingerers are making the last arrangements for New Years's Eve!

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/airbnb_vs_booking/master/images/by_day.png" width="700">
</div>

Let's explore now the popularity of the two queery by day of the week. 
This is the plot in which the two queries show the less correlation. 
While the Booking's popularity is almost constant during the whole week, the Airbnb's one is biggest on Sunday and constantly decreases to reach its minimum on Friday. 
This makes me wonder that perhaps Booking.com is more used to plan work travels than Airbnb.

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/airbnb_vs_booking/master/images/by_hour.png" width="700">
</div>


My previous hypothesis finds further evidence by looking at hourly popularity.
In fact, while the Airbnb query distribution peaks around 13 - lunch break - and after 18, the Booking.com distribution stays more or less constant during the whole working hours.



# The Whole World 

In Great Britain _Airbnb_ is more popular than _Booking.com_. 
What about the rest of the world?
Let's ask Google Trends! 
Here's the answer:

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/airbnb_vs_booking/master/images/hist_trend_world.png" width="700">
</div> 

Booking.com is winning, even if Airbnb looks very close to surpass it. 
Will it eventually happen? 
We answer this analysing the time series and making some forecast. 
The package `fbprophet` is the perfect tool for this task: it is designed to analyse time series, decompose them into trends plus seasonalities and make predictions.

In the picture below I've plotted the results of `fbprophet`'s analisys.

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/airbnb_vs_booking/master/images/decomposition.png" width="700">
</div> 

The first two plots from top show, respectively, the trend and the yearly seasonality of both the platforms. 
The bottom one is the sum of the two. 
The black vertical line corresponds to the 15th of March, the date from which the forecasts begin.

The analysis reveals that while Airbnb's trend is increasing, the Booking.com one is dropping. 
According to the forecasts, Booking's trend is going to drop below Airbnb's one in the first half of 2020. 
To these, we must take into account the seasonalities as well, that shift the date of the surpass towards the second half of the year.

But here's the crucial question: __how much are these forecasts reliable?__ 

There is confidence they can be trusted if two conditions are met:

- __The market conditions don't change__.
The forecasts have been done only analysing the numerical data, without any external input. 
In our case, the sudden spread of the _Corona Virus_ (Covid19) disease, have suddenly affected the travel sector. 
One can try to take this into account in `fbprophet`, tuning hyper-parameters 
related to `changepoints`, however, given that Covid19 appeared very recently - no effect can be seen before the Feb 15 - but at the same time they strongly affecting the travel sector, it is too early to draw any reliable prediction -after all there are only 4 samples after Feb 15!.  

- __The fit is good__. 
Poor fits lead to poor predictions. 
To asses the goodness of the fit I've studied the residuals distribution. 
If the fit is a good it should be gaussian distributed.

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/airbnb_vs_booking/master/images/residuals_plot.png" width="700">
</div> 

As it is evident from the plot, there is a clear deviation from gaussianity at the end of the time series where there are few huge outliers.
In fact, neglecting the last 4 samples of our time series, both the Shapiro and Normal tests (normally used to asses if a distribution is gaussian) say that the distributions of both residual's query are gaussian at a good confidence level.
However, the inclusion of the latest values completely spoils this results.
This is again a strong confirmation of the fact that Covid19 has have strongly impacted the travel sector. 

In conclusion, we can say that, around the world, Booking's query is still more popular than Airbnb's one, even if it seems that this latter is going to get more popular in the future.
In absence of sudden events the forecasts say that this would have happened around the second half of the year, but the sudden spread of Covid19 have made this conclusion unreliable. 
It still too early to model the impact of Covid19, but keeping collecting data will make the situation clearer.  
In the meanwhile you can take a look at the [Jupyter notebook](https://github.com/LorBordin/airbnb_vs_booking/blob/master/analysis.ipynb) I've used to make the analysis and play with it!