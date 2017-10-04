### TF-IDF In Scikit-Learn

#### 2017年9月30日补充

&emsp; 其实在算下面TF-IDF的步骤之前，还有一步，就是计算Term Frequency 也就是词频。当然，scikit-learn 中也提供了计算词频的包。

`CountVectorizer` 位于 `sklearn.feature_extraction.text `中

下面以一个小Demo 来演示计算

```python
>>> from sklearn.feature_extraction.text import CountVectorizer   
>>> countVectorizer = CountVectorizer()
>>> countVectorizer
CountVectorizer(analyzer='word', binary=False, decode_error='strict',
        dtype=<class 'numpy.int64'>, encoding='utf-8', input='content',
        lowercase=True, max_df=1.0, max_features=None, min_df=1,
        ngram_range=(1, 1), preprocessor=None, stop_words=None,
        strip_accents=None, token_pattern='(?u)\\b\\w\\w+\\b',
        tokenizer=None, vocabulary=None)
>>> corpus
['This is the first document.', 'This is the second second document.', 'And the third one.', 'Is this the first document?']
>>> X = countVectorizer.fit_transform(corpus)
>>> X
<4x9 sparse matrix of type '<class 'numpy.int64'>'
	with 19 stored elements in Compressed Sparse Row format>
>>> X.toarray()
array([[0, 1, 1, 1, 0, 0, 1, 0, 1],
       [0, 1, 0, 1, 0, 2, 1, 0, 1],
       [1, 0, 0, 0, 1, 0, 1, 1, 0],
       [0, 1, 1, 1, 0, 0, 1, 0, 1]], dtype=int64)
```

上面呢就计算好了，接下来我们需要知道的是该文本集中所有的 tokenizing ，

```python
>>> countVectorizer.get_feature_names()
['and', 'document', 'first', 'is', 'one', 'second', 'the', 'third', 'this']
```

下面解释一下上面那个X矩阵表示的意思，

`[0, 1, 1, 1, 0, 0, 1, 0, 1] `表示

第一行文本

`This is the first document.` 中 `and` 出现的次数为 0 `document` 出现的次数为 1 ，以此类推。

算出来这个后，后面就可以继续计算 TF-IDF 了。

```python
>>> from sklearn.feature_extraction.text import TfidfTransformer
>>> transformer = TfidfTransformer()
>>> tfidf = transformer.fit_transform(X)
>>> tfidf
<4x9 sparse matrix of type '<class 'numpy.float64'>'
	with 19 stored elements in Compressed Sparse Row format>
>>> tfidf.toarray()
array([[ 0.        ,  0.43877674,  0.54197657,  0.43877674,  0.        ,
         0.        ,  0.35872874,  0.        ,  0.43877674],
       [ 0.        ,  0.27230147,  0.        ,  0.27230147,  0.        ,
         0.85322574,  0.22262429,  0.        ,  0.27230147],
       [ 0.55280532,  0.        ,  0.        ,  0.        ,  0.55280532,
         0.        ,  0.28847675,  0.55280532,  0.        ],
       [ 0.        ,  0.43877674,  0.54197657,  0.43877674,  0.        ,
         0.        ,  0.35872874,  0.        ,  0.43877674]])
```

还有一个包叫做TfidfVectorizer 囊括了包括 CountVectorizer 和 TfidfTransformer 所以呢，我们可以用下面更加简便的方法。

```python
>>>
$$
E = mc^2
$$
 vectorizer
TfidfVectorizer(analyzer='word', binary=False, decode_error='strict',
        dtype=<class 'numpy.int64'>, encoding='utf-8', input='content',
        lowercase=True, max_df=1.0, max_features=None, min_df=1,
        ngram_range=(1, 1), norm='l2', preprocessor=None, smooth_idf=True,
        stop_words=None, strip_accents=None, sublinear_tf=False,
        token_pattern='(?u)\\b\\w\\w+\\b', tokenizer=None, use_idf=True,
        vocabulary=None)
>>> vectorizer.fit_transform(corpus)
<4x9 sparse matrix of type '<class 'numpy.float64'>'
	with 19 stored elements in Compressed Sparse Row format>
>>> vectorizer.fit_transform(corpus).toarray()
array([[ 0.        ,  0.43877674,  0.54197657,  0.43877674,  0.        ,
         0.        ,  0.35872874,  0.        ,  0.43877674],
       [ 0.        ,  0.27230147,  0.        ,  0.27230147,  0.        ,
         0.85322574,  0.22262429,  0.        ,  0.27230147],
       [ 0.55280532,  0.        ,  0.        ,  0.        ,  0.55280532,
         0.        ,  0.28847675,  0.55280532,  0.        ],
       [ 0.        ,  0.43877674,  0.54197657,  0.43877674,  0.        ,
         0.        ,  0.35872874,  0.        ,  0.43877674]])
>>> 
```

结果和上面的是一样的。

#### Text Feature Extraction

&emsp;首先是`Scikit-learn`特征提取中的文本特征提取的一段介绍。

&emsp;In order to address this, scikit-learn provides utilities for the most common ways to extract numerical features from text content, namely:

- **tokenizing** strings and giving an integer id for each possible token, for instance by using white-spaces and punctuation as token separators.
- **counting** the occurrences of tokens in each document.
- **normalizing** and weighting with diminishing importance tokens that occur in the majority of samples / documents.

下面这句话很关键，对于理解下面`scikit-learn`官方文档中的一个`TF-IDF`的例子，起到了决定性的作用。当时没太大意看，导致怎么看那个例子都理解不了。终于又一次翻回来看时豁然开朗。

> **A corpus of documents can thus be represented by a matrix with one row per document and <u>one column per toke</u>n (e.g. word) occurring in the corpus.**



#### TF-IDF Introduce

&emsp;Term Frequency-Inverse Document Frequency，从字面上来理解呢，就是 词频-逆文档频率。从[官方文档](http://www.tfidf.com)上我们可知这个统计方法是用来干啥的。Information Retrieval and Text mining 信息检索以及文本挖掘中。

> The tf-idf weight is a weight often used in information retrieval and text mining. This weight is a statistical measure used to evaluate how important a word is to a document in a collection or corpus. The importance increases proportionally to the number of times a word appears in the document but is offset by the frequency of the word in the corpus.

它的计算方法也很简便，TF-IDF(term,doc) = TF(term,doc) * IDF(term) 

- **TF: Term Frequency**, which measures how frequently a term occurs in a document. Since every document is different in length, it is possible that a term would appear much more times in long documents than shorter ones. Thus, the term frequency is often divided by the document length (aka. the total number of terms in the document) as a way of normalization: 

  > TF(t) = (Number of times term t appears in a document) / (Total number of terms in the document).

- **IDF: Inverse Document Frequency**, which measures how important a term is. While computing TF, all terms are considered equally important. However it is known that certain terms, such as "is", "of", and "that", may appear a lot of times but have little importance. Thus we need to weigh down the frequent terms while scale up the rare ones, by computing the following: 

  > IDF(t) = log_e(Total number of documents / Number of documents with term t in it).

#### TF-IDF Implements In Scikit-learn

&emsp;接下来解释一下TF-IDF在scikit-learn 使用计算上的一些不同处。首先是 idf(term) 的计算，计算tf-idf的包在 sklearn.feature_extraction.text 的 TfidfTransformer (关于该方法详情，参考[这里](http://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfTransformer.html#sklearn.feature_extraction.text.TfidfTransformer))中。根据 smooth_idf =True or False 的不同针对 idf 有两种不同的计算方式。分别是

* smooth_idf = true (Default)

`idf(t) = log(n(d)/(1+df(d,t)) +1`

* smooth_idf = false

`idf(t) = log(n(d)/df(d,t))+1`

> Where `n(d)` is the total number of documents, and `df(d,t)` is the number of documents that contain term `t`. The resulting tf-idf 

接下来主要解释一下官方中计算 tf-idf的一个Demo。

先贴出计算的代码，

```python
from sklearn.feature_extraction.text import TfidfTransformer
transformer = TfidfTransformer(smooth_idf=False)
counts = [[3, 0, 1],
          [2, 0, 0],
          [3, 0, 0],
          [4, 0, 0],
          [3, 2, 0],
          [3, 0, 2]]
tfidf = transformer.fit_transform(counts)
tfidf.toarray() 
//result:
array([[ 0.81940995,  0.        ,  0.57320793],
       [ 1.        ,  0.        ,  0.        ],
       [ 1.        ,  0.        ,  0.        ],
       [ 1.        ,  0.        ,  0.        ],
       [ 0.47330339,  0.88089948,  0.        ],
       [ 0.58149261,  0.        ,  0.81355169]])
```

这代码就不解释了，主要解释的是我们如何用手工来计算。

在该例子的前面有这样一句话

> The first term is present 100% of the time hence not very interesting. The two other features only in less than 50% of the time hence probably more representative of the content of the documents

我当时怎么看都不明白为什么 3 在文本中没有出现100%，但是 文档却说 100% 。如果你也这样想，恭喜你，你是无法理解下去的。我想在网上找找关于这个解释的资料，无奈找不到，只好自己再去看了一遍原文文档。

接下来就是解释：为何是 100%？

算一算第一行的`[3,0,1] `是怎么得到最后的结果`[0.8194,0,0.57320793]` 的。其实说到这里，如果你们和我一样，把这个[3,0,1]想成了正常数字中的3，0和1 的话，那么你想一周也不知道这到底怎么算的。**这里的数字代表的是该token出现的次数。**要理解这个，就回到了我们之前最初说的：

> **A corpus of documents can thus be represented by a matrix with one row per document and <u>one column per toke</u>n (e.g. word) occurring in the corpus.**

其实呢，上面那个6x3 的矩阵是这样理解的。

说的通俗点， 其实上面那个其实已经是把一堆文本处理过一遍之后得到的结果。6x3 有三列，说明有3个token。假如这三个token 是 my name robert。 那么我们可以模拟出上面第一行的文本['my robert my my'] 这样算出来的结果就知道啦，my 在该行文本中出现了3次，robert 出现了1次。所以用矩阵表示就是[3,0,1]。反观刚才的问题，是不是就解释的通了，为什么是100% ，因为这一列每个都不为0 就是说每一行都含有这个token，所以为100%。

这上面就是最难理解的，其他的就是计算了。其中涉及了欧几里得范数

接下来演示一下计算过程：（仅计算 第一行的）

```mathematica
n(d,term1) = 6

df(d,t)term1 = 6

idf(d,t)term1 = log(n(d,term1)/df(d,t)) +1 = log(1) +1 =1

tf-idf(term1) = tf x idf = 3 *1 =3

tf-idf(term2) = tf * idf = 0 *(log(6/1)+1) = 0

tf-idf(term3) = tf * idf = 1 *(log(6/2)+1) ～ 2.0986

算出来的 tf-idf 行是 
[3,0,2.0986]

然后算欧几里得范数的结果为
[3,0,2.0986]/sqrt(3^2 + 0^2 + 2.0986^2) = [0.819,0,0.573]

0.819 = 3 / sqrt(3^2 + 0^2 + 2.0986^2


// 欧几里得范数 
V(norm) = v / sqrt(v(1) x v(1) + …. + v(n) x v(n))  

Sqrt 表示根号的意思。

```

我只能说大学的线性代数已经还给老师了。

#### 后记

希望给初次学习 scikit-learn 中 Tf-Idf 的可以通过这个，少走点弯路，特别在理解文中提到的那个矩阵的时候。

####使用Python 中使用的一些工具

1. Anaconda

常用命令：

>conda create -n env_name  --file conda-requirement.txt
>
>conda create - -name python36 python=3.6
>
>source activate env_name 激活环境
>
>source deactivate env_name
>
>conda info  - -env/-e  查看当前所有环境
>
>conda remove - - name env_name - - all 删除当前名为 env_name 的环境
>
>

入门指南：http://www.jianshu.com/p/2f3be7781451

2. Redis

`redis-cli shutdown`

3. Cosine Similarity

&emsp;余弦定理： 

4. Oh My Zsh Error About Battery

* https://aaron.blog/2014/11/17/oh-my-zsh-error-about-battery/


### Reference

* http://blog.csdn.net/eastmount/article/details/50323063
* http://blog.csdn.net/liuxuejiang158blog/article/details/31360765
* http://www.cnblogs.com/chenbjin/p/3851165.html
* http://www.cnblogs.com/chenbjin/p/3843800.html
* http://scikit-learn.org/stable/index.html
* https://github.com/fxsjy/jieba