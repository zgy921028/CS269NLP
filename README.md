<!--#CS269 NLP-->

Paper summarization: Continuous Representation of Location for Geolocation and Lexical Dialectology using Mixture Density Networks
------------
by Guangyu Zhou

Reference Paper: [EMNLP2017 Paper](https://drive.google.com/open?id=0B9ZfPKPvp-JickpYa0drZWQxcHc)

Code: [``GEOMDN``](https://github.com/afshinrahimi/geomdn)

Background
------------

Many services such as web search, recommender systems, targeted advertising, and rapid disaster response rely on the location of users to personalize information and extract actionable knowledge. Text-based geolocation task of identifying the home location of users or documents using linguistic variation. This paper focus on studying of the relationships between geolocation and lexical dialectology of text in social medias. 

It is obviously at the language level that there is strong bias towards geo-location (e.g. Chinese vs. English). However, such bias is more subtly at the dialect level (e.g. English used in California vs. Texas). Language usage in social media services such as Twitter, has been used extensively either for geolocation of users or dialectology. In traditional methods, a user is often represented by the concatenation of their tweets, and the geolocation model is trained on a very small percentage of explicitly geotagged tweets, noting the potential biases implicit in geotagged tweets. These methods have their own limitations and are not suitable for solving the dialectology problem. 

Key contributions of this paper
------------------
In this paper, they use bivariate Gaussian mixtures over geotagged Twitter data in two different settings, and demonstrate their use for geolocation and lexical dialectology. Their contributions can be summarized as below:
(a)	They propose a continuous representation of location using bivariate Gaussian mixtures;
(b)	They show that the geolocation model outperforms regression-based models and achieves comparable results with classification models, but with added uncertainty over the continuous output space;
(c)	They show that their lexical dialectology model can predict geographical dialect terms from latitude/longitude input with state-of-the-art accuracy;
(d)	They show that the automatically learned Gaussian regions match expert-generated dialect regions of the U.S


Traditional Approaches
---------------------
Work on Twitter user geolocation falls into two categories: text-based and network-based methods. Text-based methods make use of the geographical biases of language use, and network-based methods rely on the geospatial homophile of user–user interactions. In both cases, the assumption is that users who live in the same geographic area share similar features (linguistic or interactional). 

Regression models, as a consequence of minimising squared loss for a unimodal distribution, predict inputs with multiple targets to lie between the targets.

Classification models, while eliminating this problem by predicting a more granular target, don’t provide fine-grained predictions and also require heuristic discretisation of locations into regions (e.g. using clustering).
Regression model: Errors assumed to be normally distributed. 
![aaaa](https://lh3.googleusercontent.com/lSb5rz3q7H1DfIBcky1AQSec-ff3FWu8343clx0_zTOoMeJB1gbqRMUZnNTbMoIEA0d4UJzLYqD2YCD2IygtuIA6yQEKPNSUR1wcYjKi3PGQAB2E1dSZwIJnANVIFg2Lw0FsQeyJvf0)KD-Tree model: Very coarse result. 

![](./img/kdtree.png)


Model & Novelty
------------------
A bivariate Gaussian distribution is a probability distribution over 2d space. It is used to generate the location coordinate pair.Then they leverage Mixture Density Network (MDN), which is a latent variable model where the conditional probability of p(y|x) is modelled as a mixture of K Gaussians where the mixing coefficients π and the parameters of Gaussian distributions μ and Σ are computed as a function of input using a neural network. 
![](./img/f1.png)

However, in the case of geolocation or lexical dialectology, the relationship between inputs and outputs is not so obvious. Instead of using learn all the parameters, they proposed Mixture Density Network with Shared Parameters (MDN-SHARED) to find the latent relationship between geo-location and lexical dialectology within each sample.Finally, they extend the Gaussian mixtures in neural networks (as in MDN) for inverse regression problems by using them as an input representation when the input is a multidimensional continuous variable. Below are the neural networks they use for the MDN of two different tasks respectively.
![](./img/NN.png?raw=true)


Here is the key function of their MDN-SHARED. 

```

 def nll_loss_sharedparams(self, mus, sigmas, corxy, pis, y_true):
        negative log likelihood loss of a 2d y_true coordinate in
        each of the Gaussians with parameters mus, sigmas, corxy, pis.
        Note that the mus, sigmas and corxy are shared between all samples
        and only pis are different for each sample.
        mus_ex = mus[np.newaxis, :, :]
        X = y_true[:, np.newaxis, :]
        diff = X - mus_ex
        diffprod = T.prod(diff, axis=-1)
        corxy2 = corxy ** 2
        diff2 = diff ** 2
        sigmas2 = sigmas ** 2
        sigmainvs = 1.0 / sigmas
        sigmainvprods = sigmainvs[:, 0] * sigmainvs[:, 1]
        diffsigma = diff2 / sigmas2
        diffsigmanorm = T.sum(diffsigma, axis=-1)
        z = diffsigmanorm - 2 * corxy * diffprod * sigmainvprods
        oneminuscorxy2inv = 1.0 / (1.0 - corxy2)
        expterm = -0.5 * z * oneminuscorxy2inv
        new_exponent = T.log(0.5 / np.pi) + T.log(sigmainvprods) + T.log(np.sqrt(oneminuscorxy2inv)) + expterm + T.log(pis)
        max_exponent = T.max(new_exponent , axis=1, keepdims=True)
        mod_exponent = new_exponent - max_exponent
        gauss_mix = T.sum(T.exp(mod_exponent), axis=1)
        log_gauss = max_exponent + T.log(gauss_mix)
        loss = -T.mean(log_gauss)
        return loss

```
Evaluation of their approaches
------------------They apply the two described MDN models on two geotagged Twitter datasets for geolocation, and compare the results with state-of-the-art classification and regression baselines. In addition, they use the mixture of Gaussian representation of location to predict dialect terms from coordinates.Thus, their code contains 3 main modules, where the first two are used to predict location from text and the last one is to predict language from location:1) lang2loc.py implements mixture density networks to predict location from text input.2) lang2loc_mdnshared.py implements shared mixture density networks to predict location from text input. This improved the model as the global mixture of Gaussian structure exists and can be learned from all the samples rather than predicted for each individual sample.3) loc2lang.py implements a lexical dialectology model where given 2d coordinate inputs predicts a unigram probability distribution over vocabulary.The evaluation of the predictions of the geolocation models is based on three measures: Acc@161, Mean, and Median. And the evaluation of the lexical dialectology model is by using perplexity of the predicted unigram distribution to compare with a baseline where the Gaussian mixture layer is replaced with a tanh hidden layer.Below is a heat map of log probabilities of the term: “hella”, which is used mostly in north CA. They generate such [maps](https://drive.google.com/open?id=0B9ZfPKPvp-JiWlhoZ01HMk9GY3c) for a list of [local dialect words](https://drive.google.com/open?id=0B9ZfPKPvp-JiTW1yWlF2ZG56SUE) . 

![](img/heatmap.png)



Geolocation Datasets
--------------------
Datasets are GEOTEXT a.k.a CMU (a small Twitter geolocation dataset)
and TwitterUS a.k.a NA (a bigger Twitter geolocation dataset) both
covering continental U.S. which can be downloaded from [here](https://www.amazon.com/clouddrive/share/kfl0TTPDkXuFqTZ17WJSnhXT0q6fGkTlOTOLZ9VVPNu)


References
----------

Christopher Bishop. 1994. Mixture density networks. Technical report, Aston University. 

Zsolt Bitvai and Trevor Cohn. 2015. Predicting peer- to-peer loan rates using bayesian non-linear regression. In Proceedings of the Twenty-Ninth AAAI Conference on Artificial Intelligence (AAAI-15), pages 2203–2209, Austin, USA. 

Zhiyuan Cheng, James Caverlee, and Kyumin Lee. 2010. You are where you tweet: a content-based approach to geo-locating Twitter users. In Proceedings of the 19th ACM International Conference Information and Knowledge Management (CIKM 2010), pages 759–768, Toronto, Canada. 

Paul Cook, Bo Han, and Timothy Baldwin. 2014. Statistical methods for identifying local dialectal terms from gps-tagged documents. Dictionaries: Journal of the Dictionary Society of North America, 35(35):248–271. 

Gabriel Doyle. 2014. Mapping dialectal variation by querying social media. In Proceedings of the 14th Conference of the EACL (EACL 2014), pages 98– 106, Gothenburg, Sweden. 

Jacob Eisenstein, Brendan O’Connor, Noah A Smith, and Eric P Xing. 2010. A latent variable model for geographic lexical variation. In Proceedings of the 2010 Conference on Empirical Methods in Natural Language Processing (EMNLP 2010), pages 1277– 1287, Boston, USA. 

Noora Al Emadi, Sofiane Abbar, Javier Borge-Holthoefer, Francisco Guzman, and Fabrizio Sebastiani. 2017. QT2S: A system for monitoring road traffic via fine grounding of tweets. In Proceedings of the 11th International Conference on Weblogs and Social Media (ICWSM 2017), pages 456–459, Montreal, Canada. 

Bruno Gonc ̧alves and David Sa ́nchez. 2014. Crowd- sourcing dialect characterization through Twitter. PloS One, 9(11). 

Bo Han, Paul Cook, and Timothy Baldwin. 2012. Geolocation prediction in social media data by finding location indicative words. In Proceedings of the 24th International Conference on Computational Linguistics (COLING 2012), pages 1045– 1062, Mumbai, India. 

Bo Han, Paul Cook, and Timothy Baldwin. 2014. Text- based Twitter user geolocation prediction. Journal of Artificial Intelligence Research, 49:451–500. 

Shen-Shyang Ho, Mike Lieberman, Pu Wang, and Hanan Samet. 2012. Mining future spatiotemporal events and their sentiment from online news articles for location-aware recommendation system. In Proceedings of the First ACM SIGSPATIAL Interna- tional Workshop on Mobile Geographic Information Systems, pages 25–32, Redondo Beach, USA. 

Yuan Huang, Diansheng Guo, Alice Kasakoff, and Jack Grieve. 2015. Understanding US regional linguistic variation with Twitter data analysis. Computers, En- vironment and Urban Systems, 59:244–255. 

Hayate Iso, Shoko Wakamiya, and Eiji Aramaki. 2017. Density estimation for geolocation via convolutional mixture density network. arXiv preprint arXiv:1705.02750. 
Sheila Kinsella, Vanessa Murdock, and Neil O’Hare. 2011. “I’m eating a sandwich in Glasgow”: Modeling locations with tweets. In Proceedings of the 3rd International Workshop on Search and Mining User- generated Contents, pages 61–68, Glasgow, UK.


Contact
-------
Guangyu Zhou <guangyuzhou@ucla.edu>