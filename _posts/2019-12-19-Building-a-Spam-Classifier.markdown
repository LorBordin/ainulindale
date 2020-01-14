---
layout: post
title:  "Building a Spam Classifier"
date:   2019-12-19
categories: jekyll update
---


We all face the problem of spam e-mails in our inboxes.
Data show that the total volume of spam mails has been consistently growing in time to the point that nowadays it overtakes the amount of ordinary or _ham_ emails. Estimates tell us that 97% of all emails sent over the Internet in 2008 were unwanted [^foot].
Most of the spam is already fitered out by our providers, but
still, quite a few spam emails are able to evade these filters and make their way to us.
 
To better detect the unwanted spam the best way is probably to train an _ad hoc_ filter, tuned on our own mails. 
Contrary to what you could think this isn't a hard task which can be realized with a little of __machine learning__.

In this post I will describe all the steps and analysis I've done, in order to build my personal spam-fitler.
Given that the important concepts are general and do not rely on any programming languag, here I do not show any code. 
If you're interested in the code - written in __Python__ - you can it you can take a look at the Jupyter notebook I've used to create this post. Here'e the link to my <a href="https://github.com/LorBordin/Spam_classifier">GitHub repo</a>. 

[^foot]: <a href="https://en.wikipedia.org/wiki/Email_spam#Statistics_and_estimates">Wikipedia/spam">Wikipedia/spam repo</a>



## 1. Getting the data

As previously said, to build a spam-filter __tuned for our (or our company) needs__  one should use her/his own mails. Getting the training data in this way is easy, in my case I built up a quite large database in few months.

However, instead of sharing my personal messages, to make this post I've opted for the <a href="https://spamassassin.apache.org/old/publiccorpus/">Apache SpamAssassin’s public datasets</a>. It's the perfect set to start with: no pre-processing at all has been made on it, the dataset consists in _raw_ emails, the  same we get from the mail-box.

The SpamAssasins's dataset consists in three different compressed files that contain:

- 2500 e-mails classified as __easy ham__, 
- 250 __hard ham__ e-mails, 
- 500 __spam__ e-mails. 

The first thing we have to do before starting any analysis is to split our dataset into a __train__ and __test__ set.
In practice we label the emails with their class (ham or spam), then we merge the sets together, shuffle and finally split the dataset in two: a bigger one (80% of the mail) and a smaller set for the final evaluation (the test set).



## 2. Data cleaning

Let's focus on the train set. We need to extract some features to train our spam-filter. A good thing to do is to look at the email content. 
Spam mails often consists in advertising messages, so they might contain some common word as _money_,  _opportunity_, _loan_ or _mortage_. 

So we want to extract the words content from each mail and build 
a list with the most common spam mails.

Let's start with the words extraction part. To avoid repetitions it's better to firstly parse the mails' content doing the following operations:
 
- remove all the html tags, 
- remove punctuation, 
- covert words in lower-case, 
- replace numbers with _NUMBER_ and urls with _URL_. 

If you use _python_, some packages like `email`, `regex` and  `beautifulsoup` are very useful for this purpose.



## 3. Data Analysis

We've extracted the words let's __count their frequency__.
It is convenient to build 2 different vocabularies that count the frequency hammy and spammy words.
In doing so don't forget to __remove the english common stopwords__ like _the_, _is_, _at_, ..., which are not useful to understand the content of the mail.
Here's the 20 most common ham and spam words in my train dataset:

<img src="https://raw.githubusercontent.com/LorBordin/Spam_classifier/master/images/words_freq.png" alt="">

Good! Notice Words like __free__, __money__, __click__ that appear often only in spam mails. They look like really spammy words!

Let's parse the __mails' subjcet__  as well: it will probably contain important info.
Let's build a vocabulary that counts the most frequent spam's subject words.
Here's the 20 most common words:

<img src="https://raw.githubusercontent.com/LorBordin/Spam_classifier/master/images/subj_spam.png" alt="">

We've already collected a lot of interesting features. Still many other  info can be extractred from database!
For instance, many emails consist in multipart and, a closer analysis reveals that spam and ham have in general a different structure: 

<img src="https://raw.githubusercontent.com/LorBordin/Spam_classifier/master/images/email_type.png" alt="">

There's a lot of _plain text_ in the ham dataset, while _html_ is more frequent in the spam one. Good to know.



## 4. Preprocessing

It's time to convert all the extracted features into frequency vectors. 
For each mail we create a long vector (507 entries!) that stores

 
- the frequency if the __most frequent 200 ham words__, 
- the frequency of the __most frequent 250 spam words__, 
- the frequency of the __most frequent 50 subject words__, 
- the __email's type__.

Better to __rescale the features vector__ in order to perform better on the  Support Vector Machine algoritims.

To get an a-priori idea of how the emails dispose in the features space, we use the _t-SNE_ algorithm that reduces the dimensionality of the features vector while trying to keep similar instances close and dissimilar instances apart. 

<img src="https://raw.githubusercontent.com/LorBordin/Spam_classifier/master/images/clustering.png" alt="">

Notice how ham clusters in several regions of the pain, while spam spreads mainly in the top-right part of the plot.

Before training any model, let's tackle the __overfitting__ problem. The features vector we've created has a large number of entries and many of them consist in words that appear both in the ham and the spam vocabularies, so there are many repeated entries. Traininig any algorithm with such features will, most probably, overfit the data causiung the algorithm will perform poorly on the new ones. 
To avoid this problem I've created a function that randomly selects only a  subset of features, taking care of removing all the repetitions.
The subset length is treated as a hyper-parameter that is tuned for each classifier during the training phase. 



## 5. Training the classifiers

Time to train our filter! 

But firstly.. What __score__ should the maximise? 
The simplest thing we can  think is the __accuracy__, that just counts the number of wrong predictions  


<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/ainulindale/gh-pages/_images/accuracy.png" width="225">
</div>   

<br>

But, this is __not a good score__: our dataset consists of  only ~ 15% spam mails, so a _dumb filter_ that says everything is ham would score ~ 0.85 out of 1! <br>
Moreover, accuracy considers _false positive_ (FP) and _false negative_ (FN) equally important. In other words it makes no distinction if ham is classified as spam (FP), or if spam is classified as ham (FN). But, of course, we care more that our spam filter does not filter out ham messages, better if it lets some spam pass instead.
So instead of accuracy it's better to use a combination of two additional scores, __precision__ and __recall__, 


<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/ainulindale/gh-pages/_images/prec_rec.png" width="380">
</div>

<br>

where TP and FP stand for _true positive_ and _false negative_ (respectively, spam and spam classified as ham). Ideally we want to tune our filter to have the maximum possible precision - 1.0 - while keeping recall the highest possible.
Therefore, we decide to maximise the __F1 score__, that is the _harmonic mean_ of precision and recall. 

We chose the score, what about the classifier?<br>
Shall we train Linear model? Support Vector Machine? Random Forest? Well let's train them all and __make a voting classifier__, most probably it'll perform better than any individual classifier.
Let's train the following algorithms:

 
- a __Logistic regressor__, - a Stochastic Gradient Descent or __SGD classifier__, 
- a __Random Forest classifier__, 
- a __K Neighbors classifier__, 
- a Support Vector Machine or __SVM classifer__. 

As previously discussed, each classifier is trained on a different feature sets. In this way not only we lower the risk of overfitting, but also the algorithms will make more uncorrelated errors and the voting classifier will perform better. 

To get an idea of how the individual classifiers will generalise to an independent data set, _cross-validation_ is a good option. It's a tequinique that consists in dividing the train set in some validation sets (5 is a good choice), and then evaluating the classifier on each validation set, after having trained it on the rest of the data.
Here's my results:

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/Spam_classifier/master/images/cv_performances.png" width="400">
</div> 

Since the Random Forest classifier performs the best among the individual ones, its vote count twice on the voting classifier.

A great quality of Random Forests is that they make it easy to measure the relative importance of each feature. The following plot shows the features that contributed more than 1% in the decision process.

<img src="https://raw.githubusercontent.com/LorBordin/Spam_classifier/master/images/feature_importances.png" alt="">

So the most important feature is _click_ (8%) followed by _please_ and the email types _text/html_ and _text/plain_ (all around 5%).



## 6. Final evaluation

We've fine-tuned and trained the algorithms, cross validated them on the train dataset and built a voting classifier. It only reamin to evaluate our filter on the test set in order to asses its performances.

Of course do not forget to preprocess the test set  in the same as we did for the training set before!   
Here's the perfomances of the voting classifier on an unknown dataset.

<div align="center">
 <img src="https://raw.githubusercontent.com/LorBordin/Spam_classifier/master/images/test_voting.png" width="200">
</div>

It performs very well, confirming the results of the cross-validation test!
Let's take a look at the __confusion matrix__ :

<div align="center">
 <img src="https://raw.githubusercontent.com/LorBordin/Spam_classifier/master/images/confusion_matrix.png" width="300">
</div> 

Wow! It misclassified only 2 ham mails; at the same time it let passed 13 spam mails out of 100. Not bad!

But let's take a closer look at the misclassified ham mails:

The first is an advertising email form Ryanair

> Ryanair in partnership with Primary Insurance  
> offer excellent value travel insurance from  
> £7.00GBP/9.00 Euro per person for 31 day cover.  
>
> Annual travel insurance* from £45.00GBP/63.00 Euro,  
> includes 24 days winter sports cover !
>
> Our travel insurance provides a high standard of cover.
>
> Summary of Cover
>	
> Medical Expenses up to £2 million  
> Personal Liability  up to £2 million   
> Personal Effects & Baggage up to £750  
> Personal Accident Maximum Benefit £15,000  
>
> ...


while the second is written in Japanese (according to Google translate)

> OpenText社  
> 伊東様
>
> いつもお世話になっております。 
> 安井@infocomです。
>
> あるタスクリストに、適当なマイルストーンを複数、タスクを複数用意し  
> それぞれのタスクについては、存在する適当なマイルストーンに割り当てたものと  
> します。  
> あるマイルストーンを見ると、添付の絵にあるような画面が表示されるのですが  
> ここで丸で囲った部分（ 期間 ）は、何を意味しているのでしょうか？  
>
> つまり、３７日 、２７日  
> のそれぞれの意味（算出のされ方について教えてください。） 
>
> また「週末を除く」とありますが、週末を除かない設定方法はあるのでしょうか？
> 
> 以上、ご回答の方よろしくお願いいたします。
>
>
> --
> +--------------------------------------+
> インフォコム(株)  
> ナレッジマネジメント本部  
> KMフロンティア部  
> コラボレーティブシステムグループ  
>
> 〒101-0062  
> 東京都千代田区神田駿河台3-11三井住友海上駿河台別館5F  
> 安井  剛  
> E-mail: go@infocom.co.jp  
> TEL: 03-3518-3299  
> FAX: 03-3518-3055  

Here, the distinction between ham and spam gets thin, personally I'd labelled both the emails as spam.

Before concluding, let's compare the performances of the individual classifiers on the test set: 

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/Spam_classifier/master/images/test_performances.png" width="400">
</div> 

As expected __the voting classifier scores better than any individual one!__ 

## 7. Conclusion

In this post we've exploited a public dataset of raw emails and in order to make a __simple yet powerful spam-filter__ that detects most of the spam that evade the controls of our providers.  

Again, take a look at my 
<a href="https://github.com/LorBordin/Spam_classifier">GitHub repo</a>
and in particular at the  __Jupyter Notebook__ in it, if you want to inspect the code I used to make my filter.

<!--- comment on periodically update the dictionaries and re-train the voting classifier in order to keep the filter up-to-date with the new spam e-mails. 
This is a simple task and it isn't time consuming as well. --->

Let's comment about some possibile generalisations.
Here we've build a spam-filter, however the same ideas can applied to other similar tasks; the code as well could be used with little changes. 

Obvious examples are a __Hate-speach detector__, that classifies profiles in a social network with little modifications, or a __Clickbait recognizer__, that detects false advertisement from a website.


