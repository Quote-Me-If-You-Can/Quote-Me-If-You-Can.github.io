---
layout: page
title: Quote me if you can !
cover-img: "img/background2.jpg"
subtitle: An ADAventure based on quotations
classes: wide
---

Ever heard of [Catch Me If You Can](https://en.wikipedia.org/wiki/Catch_Me_If_You_Can) ? In this movie, the main character impersonates many professions mainly by appropriating their uniforms, specific behaviours and lingos, and everybody fall for it !  

<figure>
    <p align="center">
    <img title="Do you concur ?" width="400px" src="img/tenor.gif">
    </p>
</figure>

For the uniform it is quite obvious, but what can we say about the lingo ? One will agree that each occupation has its « proper words », but let's think further, a person of a specific profession will also express himself in a certain manner. Think of a politician, he does not only speaks about economy and education, but also uses some figure of speech, and his discourse is fluid and well conducted; rather of a sportsman or a scientist, for exemple.  

Based on this statement: give us a quote, and we will tell you which profession said it ! 


## Table of Contents

- [Datasets](#datasets)
- [Filtering](#filtering)
- [Occupation clustering](#occupation-clustering)
- [BERT](#bert)
- [Results](#results)
- [Room for improvement](#room-for-improvement)
- [References](#references)



## Datasets   

We present here the two main datasets used for this project. You can find them following the link in the upper menu. 

#### Quotebank
We do we have here ? The full dataset is made of 178 million quotations together with a list of possible speaker ranked by probability, the name of the most probable speaker and its [Wikidata](https://www.wikidata.org/wiki/Wikidata:Main_Page) Qid, when it has been published, and where it has been published. The later is important because this information has been exctracted out of 162 million English news articles published between 2008 and 2020 included, so one might want to keep records of it.

162 million, 178 million, but who did that ? Well it's the nice library assistant Quobert, and it does it freely. Here is a nice picture (taken from [this publication](https://dlab.epfl.ch/people/west/pub/Vaucher-Spitz-Catasta-West_WSDM-21.pdf)[^1] explaining in full detail how the dataset has been collected) that could help you to grasp Quobert's workflow.

<figure>
    <p align="center">
    <img title="Quobert.PNG" width="400px" src="img/Quobert.PNG">
    </p>
</figure>

Super nice, except that it correctly attributes only about 85% of the quotations. But could you except better from un unpaid assistant ?  
Here is some of its failures:

<figure>
    <p align="center">
    <img title="Fails" width="800px" src="img/fail.png">
    </p>
</figure>

* The Harry Potter he refers to is an [Australian journalist](https://en.wikipedia.org/wiki/Harry_Potter_(journalist)).
* Joe Bidden does not like himself so much.

But this will be good enough for our purpose. Just keep in mind that Quobert is an unpaid assistant.  
Furthermore, for this project, we did not use the full Quotebank but only the years 2015-2020 included.

#### External sources
We said we want to predict the occupations but did not said a word about it so far... Quotebank is augmented using metadata about the speakers. It an external dataset we will find for ~9M Wikidata entities some additional information as occupation, gender, religion, etc.


## Filtering

In the first place, we had to deal with the "homonym issue". This is because a quotation is linked to a list of names, ranked by probability, and further linked to a single name in a "winner takes it all" fashion. Once a name is elected, the name is linked to its corresponding Qid; BUT if a name has a namesake, then several Qids will be added. So how do we know that we are facing Harry Potter the magician or Harry Potter the journalist (it does not really apply in this case because there is no fictional character in Quotebank, but you got the point) ? Furthermore, if Quobert is uncertain, it just assigns "None" as speaker with an empty list of Qid.To cope with those issues we simply filtered out all speakers, whose list of Qid contained more or less than 1 Qid.  
By doing so we dealt with both the "homonym issue" and the "empty speaker issue" and get rid of 50% of the remaining data.

<figure>
    <p align="center">
    <img title="i love magic" width="600px" src="img/tumblr_l982reZrDD1qzcmp3o1_500.gif">
    </p>
</figure>

Fine, but now suppose that Harry Potter the magician was tired of figthing evil, and after all, You-Know-Who has been defeated in 2011 so... He decided to start a career as a journalist at the renown "The Daily Prophet" newspaper. We now have Harry Potter the magician and Harry Potter the journalist (it is the same person now). So how do we know that his quotes are those of a magician or those of a journalist ? 
This is why we remove all speaker that have more than one occupation. By doing so we remove 12% of the remaining data.

## Occupation clustering

By applying our filtering criterions on the external dataset, we find that there is 6800 different occupations in it. This is a lot. Here is a look at the distribution of occurencies of occupations:

<figure>
    <p align="center">
    <img title="occuroccup" width="400px" src="img/nb_occur_occup.png">
    </p>
</figure>

We selectected an amount we can sort by hand = 280 (even though it is still a piece of work), this correspond to a treshold of occurencies > 1000. The downside is that we loose a lot of occupation just because we can't sort them by hand. We would need another unpaid assistant to do it for us... and we will come back later to that. Those "unclustered" occupations are gathered in `Other`. There was some "nan" occupation that survived the df.dropna() because they were of type string. We store those in `NoOcc`. The later class won't be use on the next step. It can be seen as a garbage class. 

To define some clusters we looked at those publications [here](https://repository.library.georgetown.edu/handle/10822/559298)[^2] and [there](https://www.leyden212.org/Page/4244) about Career Clusters. After that, we classified by hand our occupations with occurencies > 1000 in similar clusters and assign our quotations to unique occupation. (recall: 1 quotation >> a unique Qid speaker >> a sole occupation). We build two new datasets:  
One consists of 4 classes, it will be used  for the proof-of-concept of the -later explained- BERT-based classifier:

| Cluster | Label | Meaning | # of quote|
|-------|--------|---------|---------|
| 0 | Research | Research and science related careers | 2'372'142 |
| 1 | Politics | Government related careers | 6'928'256 |
| 2 | Sports | Sport related careers | 13'237'461 |
| 3 | Arts | Artists and creator related careers | 2'421'718 |

<figure>
    <p align="center">
    <img title="distribution4" width="600px" src="img/distribution4.PNG">
    </p>
</figure>

The second one consists of the 20 classes:

| Cluster | Label | Meaning | # of quote|
|-------|--------|---------|---------|
| 0 | Other| NaN careers | 2'167'753|
| 1 | AFNR | Agriculture, Food and Natural Resources careers | 9'891 |
| 3 | AAVTC | Arts, Audio/Video Technology and Communications careers | 2'649'277 |
| 4 | BMA | Business Management and Administration careers | 961'273 |
| 5 | ET | Education and Training careers | 14'009'216 |
| 6 | F | Finance careers | 275'615 |
| 7 | GPA | Government and Public Administration careers | 6'491'038 |
| 8 | HS | Health Science careers | 46'010 |
| 9 | HumS | Human Services careers | 111,'82 |
| 10 | IT | Information Technology careers | 14'356 |
| 11 | LPSCS | Law, Public Safety, Corrections, and Security careers | 639'833 |
| 12 | M | Manufacturing careers | 129'037 |
| 13 | MSS | Marketing, Sales, and Service careers | 97'653 |
| 14 | STEM | Science, Technology, Engineering, and Mathematics careers | 2'152'432 |
| 15 | R | Religion related careers | 173'688 |
| 16 | AT | Academic and Teacher related careers | 177'524 |
| 17 | J | Journalism related careers | 839'391 |
| 18 | MW | Military and War related careers | 179'833 |
| 19 | AS | Aircraft and Space careers | 24'947 |
| 20 | NoOcc | Not clustered careers | 1365 |

<figure>
    <p align="center">
    <img title="distribution20" width="600px" src="img/distribution20.PNG">
    </p>
</figure>

If you noticed that the cluster #2 does not exist in the last table it is just because we created a cluster with no quotes in it by mistake... if yu didn't notice it, this is not so important.

## BERT

Where we find it  
Did we modify it ?  
Write about internal preprocessing  


## Results
We present here the different results: the "proof-of-concept" classification, followed by the 20 classes classification, and an extra step ;)

#### Proof-of-concept
We trained and tested with unbalanced datasets, but make use of weighted loss.   
After having fed the classifier with the quotes classified following the 4-classes table, and trained it for 20 minutes, we get the following results:

<figure>
    <p align="center">
    <img title="4class_loss_graph" width="400px" src="img/results/4classes_loss_graph.png">
    </p>
</figure>

The behaviour of the curves has nothing to do with the swiss mountains; it just come from the fact that we used a scheduler to vary the lerning rate of our AdamW optimizer. But its initialization was not well set and it initialized repeatidly instead of only once, so we find the strange behaviour. But it is fixed in the last use of the classifier.  
The important point is that it learns well and fast ! Here is the roc_auc for each class:

<figure>
    <p align="center">
    <img title="4results" width="400px" src="img/results/4classes_rocauc_values.png">
    </p>
</figure>

"Amount of class*" states the number of quote belonging to this class in the test set. As said before, it is unbalanced.  
The results seems convincing. Next step: just feed the classifier with the quotes classified following the 20-classes table, run it for 8-10h and that's it! 

#### 20 classes
We trained and tested with unbalanced datasets, but make use of weighted loss. After having trained for about 9h, we get the following results:

<figure>
    <p align="center">
    <img title="20class_loss_graph" width="400px" src="img/results/20classes_loss_graph.png">
    </p>
</figure>

(Same problem with scheduler, wait for it). Ok, we can't assess if it is good or bad by looking at the evolution of the loss but what about the roc_auc values ?

<figure>
    <p align="center">
    <img title="20results" width="400px" src="img/results/20classes_rocauc_values.png">
    </p>
</figure>

They are close to 0.5, which means that the classifier just classify randomly. This is super bad !

#### Can we do better ?
We assumed that the problem came from the fact that we have to many classes. So we decided to move from 20 to only 10 classes. We merge the classes according their similarity.  Here is the new classification table:

| Cluster | Label | Meaning | # of quote |
|-------|--------|---------|---------|
| 0 | AAVTCM | Arts, Audio/Video Technology and Communications careers | 2'778'314 |
| 1 | BMAxF | Business Management and Administration careers | 1'236'888 |
| 2 | GPAxLPSCS | Government, Law, Security careers | 7'072'268 |
| 3 | MSSxHumS | Marketing, Sales and Service careers | 209'135 |
| 4 | ATE | Academic and Teacher related careers | 177'524 |
| 5 | SPORTS | Sport careers | 14'009'216 |
| 6 | STEMxIT | Science, Technology, Mathematics and Health science careers | 2'316'481 |
| 7 | R | Religion related careers | 173'688 |
| 8 | J | Journalism related careers | 839'391 |
| 9 | MW | Military and War related careers | 188'863 |

Furthermore, after some reflexions, it seemed that the unbalanced testing set was fooling us in some ways for the interpretation of the results. Thus, this step is done with unbalaced training set but balanced testing set.  

<figure>
    <p align="center">
    <img title="4class_loss_graph" width="500px" src="img/results/10classes_unbalanced_loss_graph.png">
    </p>
</figure>

Well, even though the scheduler thing is still not fixed (waaiiiiiit for iiiit), those loss curves are kind of impossible to interpret in any ways. We can look at extended performance metrics:

<figure>
    <p align="center">
    <img title="10results" width="500px" src="img/results/Unbalanced_merged.png">
    </p>
</figure>

Here, "support" states that there is 10'000 quotes/class in the test set.  
We can see that the f1 score is low for some classes. We then decided to plot the confusion matrix of classification. And here is what we got:

<figure>
    <p align="center">
    <img title="10results" width="600px" src="img/results/confusion_matrix_unbalanced.png">
    </p>
</figure>

Even though the diagonal term seems to by the higher column-wise, some columns behaves as attractors. That is the classes 3,6 (GPAxLPSCS, SPORTS) and 1,2,7 (AAVTCM, BMAxF, STEMxIT). Unsurprisingly, this is correlated with the number of quote per class in the training set. The weighted loss does not correct the problem ! We have to try to pass it a balanced training set 

<figure>
    <p align="center">
    <img title="distribution10h" width="600px" src="img/distribution10h.PNG">
    </p>
</figure>

## Results and limitations 
blalala

## Room for improvement
blalala

## Game embeddings

 train a Sent2vec[^2] 

[**this dedicated page**]({% link embeddings.md %})

## References<br>


[^1]: [Vaucher, T., Spitz, A., Catasta, M., & West, R. (2021, March). Quotebank: A Corpus of Quotations from a Decade of News. In Proceedings of the 14th ACM International Conference on Web Search and Data Mining (pp. 328-336).](https://dlab.epfl.ch/people/west/pub/Vaucher-Spitz-Catasta-West_WSDM-21.pdf)
[^2]: [Carnevale, A. P., Smith, N., Stone III, J. R., Kotamraju, P., Steuernagel, B., & Green, K. A. (2013). Career clusters: Forecasting demand for high school through college jobs, 2008-2018.](https://repository.library.georgetown.edu/handle/10822/559298)



<details>
<summary><b>Images</b></summary>
<ul>
    
<li>2017-11-11-170105.jpg. (2021, December 16). https://wallpapercave.com/w/wp3396925</li>
<li>tenor.gif. (2021, December 16). https://media1.tenor.com/images/24eba459fc0a6e19c4d2d60ed678e2f9/tenor.gif?itemid=7219821</li>
<li>[Quobert.PNG (2021, December 17). https://dlab.epfl.ch/people/west/pub/Vaucher-Spitz-Catasta-West_WSDM-21.pdf<\li>
<li>tumblr_l982reZrDD1qzcmp3o1_500.gif (2021, December 17). http://1.bp.blogspot.com/-kQcj14MDfdA/Tl7fyQQ_hnI/AAAAAAAAAE8/KI-pUSWaHF8/s1600/tumblr_l982reZrDD1qzcmp3o1_500.gif <\li>
    
 </ul>
<br>
</details>
