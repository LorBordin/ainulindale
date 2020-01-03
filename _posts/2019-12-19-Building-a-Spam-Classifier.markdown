---
layout: post
title:  "Building a Spam Classifier"
date:   2019-12-19
categories: jekyll update
---


We all face the problem of spam e-mails in our inboxes.
Data show that the total volume of spam mails has been consistently growing in time to the point that it overtakes the amount of ordinary or _ham_ emails. Estimates tell us that 97% of all emails sent over the Internet in 2008 were unwanted [^foot].
Fortunately, we don't see all these messages since the providers we use already contain spam filters that simply delete or reject _obvious spam_.
However, many unwanted messages are still able to avoid these filters and make their way to us.

To better filter out all the unwanted spam one can build and tune an _ad hoc_ filter using his own mails. This is a simple task that can be realized using some __machine learning__ technique.
As for any machine learning project the hardest part is the preprocessing that one needs to do before training the algorithm. 
In this post I will describe in words how analyse an email dataset in order to select and extract the best features that will be be used to build a simple yet powerful spam filter. 

My filter is written in __python__ however the ideas I describe are very general .
For this reason I do not share any snippet in this post.
If you are interested in the code, you take a look at the Jupyter notebook in my [GitHub repo] (https://github.com/LorBordin/Spam-Classifier).

[^foot]: [Wikipedia/spam] (https://en.wikipedia.org/wiki/Email_spam#Statistics_and_estimates">Wikipedia/spam)


## 1. Getting the data

The best data we can use to build a spam filter __tuned for our (or our company) needs__  are our own mails. Getting the data in this way is easy, in my case I built up a quite large database in few months.

For privacy reasons, I will not share my private dataset, but we will work on the <a href="https://spamassassin.apache.org/old/publiccorpus/">Apache SpamAssassin’s public datasets</a>. It's the perfect set to start with: no pre-processing at all has been made the dataset that consists in _raw_ emails, the we get from our mail-box.

The SpamAssasins's dataset consists in three different compressed files that contain:

- 2500 e-mails classified as __easy ham__, 
- 250 __hard ham__ e-mails, 
- 500 __spam__ e-mails. 

After having downloaded the content, the first thing we have to do before starting any analysis is to split our dataset into a __train__ and __test__ set.
In practice we label the emails with their class (ham or spam), then we merge the sets together, shuffle and finally split the dataset in two: a bigger one (80% of the mail) and a smaller set for the final evaluation (the test set).



## 2. Data cleaning

Let's focus on the train set. We need to extract some features to train our spam-filter. A good thing to do is to look at the email content. 
Spam mails often consists in advertising messages, so they might contain some common word as _money_,  _opportunity_, _loan_ or _mortage_. 

It is therefore important to extract the words content from each mail and build 
a list with the most common spam mails.

Let's start with the words extraction. We need to implement the following parsing operations:

 
- remove all the html tags, 
- remove punctuation, 
- covert words in lower-case, 
- replace numbers with _NUMBER_ and urls with _URL_. 

In python, there are some packages which are very useful for this purpose. Personally, I used  `email ` to extract the body's content from each email,  `regex ` and  `beautifulsoup ` to parse it. 



## 3. Data Analysis

Once we've extracted the words we have to __count their frequency__.
For this purpose let's build 2 vocabularies that count the frequency hammy and spammy words.
We do so taking care of __removing the english common stopwords__, i.e. words like _the_, _is_, _at_, ..., which are not useful to understand the purpose of an email.
Here's the 20 most common ham and spam words in my train dataset:

<img src="https://raw.githubusercontent.com/LorBordin/Spam_classifier/master/images/words_freq.png" alt="">

Good! Words like __free__, __money__, __click__ looks really spammy words!

Not only the mail's content but also the __mail's subjcet__ will probably contain important info.
Better make a vocabulary for spam's subject words then.
Here's the most common spam subject 20 words:

<img src="https://raw.githubusercontent.com/LorBordin/Spam_classifier/master/images/subj_spam.png" alt="">

We've already collected many interesting features, however our mails still contain a lot of info that can be useful for our filter.
For instance, let's take a look at __how emails, are structured__. Notice that many of them consist in multipart and, a detailed analysis reveals that spam and ham have in general a different structure: 

<img src="https://raw.githubusercontent.com/LorBordin/Spam_classifier/master/images/email_type.png" alt="">

There's a lot of _plain text_ in the ham dataset, while _html_ is more frequent in the spam one. Good to know.



## 4. Preprocessing

Now that We've chosen the features to feed our spam-filter with we need to convert them into vectors. 
For each mail we define a vector of length 507 that stores

 
- the frequency if the __most frequent 200 ham words__, 
- the frequency of the __most frequent 250 spam words__, 
- the frequency of the __most frequent 50 subject words__, 
- the __email's type__.

Do not forget to __rescale the features vector__ if we are going to use a Support Vector Machine or a Linear model (as I did) to fit the data.

We can get an idea of how the emails dispose in the features space by using the _t-SNE_ algorithm that reduces the dimensionality of the features vector while trying to keep similar instances close and dissimilar instances apart. 

<img src="https://raw.githubusercontent.com/LorBordin/Spam_classifier/master/images/clustering.png" alt="">

Notice how ham clusters in several regions, while spam clusters a bit less but lies mainly right part of the plot.

Before proceeding to the training part. let's tackle the problem of __overfitting__. Our features vector has a large number of entries (507), moreover many of them are words that appear both in the ham and the spam vocabularies, so there are many repeated entries. If we train an algorithm with this vector most probably we are going to overfit it! To avoid this problem we define a function that randomly selects only a subset of entries, taking care of avoiding repeated features. In this way each classifier we're going to use, will be trained on a different set of features.

## 5. Training some algorithm

Time to train our filter! 

But firstly we need to choose a __training score__. 
The simplest thing we could be tempted to use is the __accuracy score__ that just counts the number of wrong predictions  


<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/ainulindale/gh-pages/_images/accuracy.png" width="225">
</div>   

<br>

This is __not a good score__, though. 
In fact, in our dataset only ~ 15% of mails are spam. So, a _dumb filter_ that classifies everything as ham would score an accuracy of ~ 85% which is already high.<br>
Moreover, accuracy considers _false positive_ (FP) and _false negative_ (FN) equally important. In other words it makes no distinction if ham is classified as spam (FP), or if spam is classified as ham (FN). But of course we care more that our spam filter does not filter out ham messages, better if it lets some spam pass instead!
This is to say that instead of accuracy it's better to use a combination of two additional scores, __precision__ and __recall__, 


<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/ainulindale/gh-pages/_images/prec_rec.png" width="380">
</div>

<br>

where TP and FP stand respectively for _true positive_ and _false negative_, which are respectively, spam and spam classified as ham. Ideally we want to tune our filter to have the maximum possible precision, i.e. 1, and the highest recall.
The _harmonic mean_ of precision and recall is called __F1 score__ and works better than accuracy. This is the score we are going to maximize in training the spam filter. 

We chose the score, let's train the classifier! But..<br>
What model shall we use? Linear model? Support Vector Machine? Random Forest? Well let's train them all and __make a voting classifier__, most probably it'll perform better than any individual classifier.
Personally, I've trained the following algorithms:

 
- a __Logistic regressor__, - a Stochastic Gradient Descent or __SGD classifier__, 
- a __Random Forest classifier__, 
- a __K Neighbors classifier__, 
- a Support Vector Machine or __SVM classifer__. 

As discussed in the previous section, it's better to train each classifier with different features sets, smaller than the total features set. This will help not only for lowering the risk of overfitting, but in this way the algorithms will make uncorrelated errors and the voting classifier will perform better. 

To get an idea of how the individual classifiers will generalize to an independent data set we use _cross-validation_. In practice, we  divide the train dataset in 5 validation sets, then the classifier is evaluated on each validation set, after it is trained on the rest of the data.
Here's the results of each model:

<div align="center">
<img src="https://raw.githubusercontent.com/LorBordin/Spam_classifier/master/images/cv_performances.png" width="400">
</div> 

Since the Random Forest classifier performs the best among the individual ones, its vote count twice on the voting classifier.

A great quality of Random Forests is that they make it easy to measure the relative importance of each feature. The following plot shows the features that contributed more than 1% in the decision process.

<img src="https://raw.githubusercontent.com/LorBordin/Spam_classifier/master/images/feature_importances.png" alt="">

It seems that the most important feature is the word _click_ (8 %) followed by _please_ and the email types _text/html_ and _text/plain_ (all around 5%).



## 6. Final evaluation

We've fine-tuned and trained the algorithms, cross validated them on the train dataset and built a voting classifier. It's time to check the performances on the test dataset to asses the performances of our spam filter!

Of course we firstly need to preprocess the test set generating the features vectors as we did for the training set. Then we can finally asses how the voting classifier behaves on an unknown dataset.

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

This confirms that __the voting classifier scores better than any individual one!__ 

## 7. Conclusion

In this post we've exploited a public dataset of raw emails and in order to make a __simple yet powerful spam-filter__ that detects most of the spam that evade the controls of our providers.  

Again, take a look at my GitHub repo (MAKE LINK) and in particular at the  __Jupyter Notebook__ if you wanna inspect the code I used to analyze and preprocess the mails and to train the filter.

<!--- comment on periodically update the dictionaries and re-train the voting classifier in order to keep the filter up-to-date with the new spam e-mails. 
This is a simple task and it isn't time consuming as well. --->

Finally, I'd like to comment about possible extensions. 
We've developed a spam filter, however the same ideas can applied to other similar tasks; the code as well could be used with little changes. 

Obvious examples are a __Hate-speach detector__, that classifies profiles in a social network with little modifications, or a __Clickbait recognizer__, that detects false advertisement from a website.


