---
layout: post
title: Quicksort in Clojure
tag: Clojure
feature-img: "img/sample_feature_img_2.png"
---

最近在学习*Robert Sedgewick*老头子的《算法》红宝书，刚好看到`Sorting`那一部分。各种有趣的排序算法，真是大开眼界，其中最`NB`的要算*Quicksort*了。书里的算法都是用`Java`实现的，但是`Java`写多了，真的有点无聊（而且手累），所以我就尝试用`Clojure`来实现一下*Quicksort*。

*Quicksort* 的基本思路就是：

1. 在序列中找一个数作为基准(*pivot*)（暂时取第一个数吧）
2. 对序列进行分区操作，将比这个基准大的数都放到它的右边，将小于或等于它的数都放到<br>
　它的左边（这个过程实际上就已经将作为基准的数放到了其排序后的正确位置）
3. 再对左右区间重复上一步，直到各区只有一个数，这样整个序列就排好啰~~~

在`Clojure`里面，函数`filter`和`remove`可以用来筛选出一个序列里面我们想要的数（大于基准的数和不大于基准的数）。所以*Quicksort*在`Clojure`里可以这样写：

{% highlight clojure linenos %}
(defn qsort [[pivot & xs]]
  (when pivot
    (let [smaller? #(< % pivot)]
      (lazy-cat (qsort (filter smaller? xs))
                [pivot]
                (qsort (remove smaller? xs))))))
{% endhighlight clojure %}

* `[[pivot & xs]]`对参数进行解构(destructuring)，即将序列的第一个数bind给*pivot*，<br> 
　剩余部分bind给*xs*
* `smaller?`是由匿名函数`#(< % pivot)`定义的一个判断参数是否小于基准的函数
* `lazy-cat`是用来惰性地拼接两个筛选出来并排好序的子序列
<br>
<br>

同样也可以用`for`宏来实现`filter`和`remove`的功能：

{% highlight clojure linenos %}
(defn qsort [[pivot & xs]]
  (when pivot
    (lazy-cat (qsort (for [x xs :when (< x pivot)] x))
              [pivot]
              (qsort (for [x xs :when (>= x pivot)] x)))))
{% endhighlight clojure %}
<br>

也可以用``quasiquote(`)``来拼接两个子序列：

{% highlight clojure linenos %}
(defn qsort [[pivot & xs]]
  (when pivot
    `(~@(qsort (filter #(> % pivot) xs))
      ~pivot
      ~@(qsort (remove #(> % pivot) xs)))))
{% endhighlight clojure %}

* 这样写感觉有点逼格，但是呢，速度实在不如前面两种写法→_→
<br>

*Quicksort*有一种升级版叫*3-way-Quicksort*，它在序列中重复的数比较多时食用更佳哦！它的思想和普通*Quicksort*差不多，只是它分区的时候不止分左右两个区，而是同时筛选出与基准相等的数，并放在中间，这样就可以一次过将所有与基准相等的数放到它们排序后的正确位置，而不是每次只能排好一个数。接下来就是用同样的方法递归地去排列左右两个区啦。实现起来也很简单，只需在原来的基础上加多一行代码来筛选出与基准相等的数：<br>

{% highlight clojure linenos %}
(defn three-way-qsort [[pivot :as xs]]
  (when pivot
    (lazy-cat (three-way-qsort (filter #(< % pivot) xs))
              (filter #{pivot} xs)
              (three-way-qsort (filter #(> % pivot) xs)))))
{% endhighlight clojure %}

* 在`Clojure`里面，`set`是可以当函数的哦！(๑•̀ㅂ•́)و✧<br>
* 所以可以直接用一个set，即`#{pivot}`作为判断一个数是否与`pivot`相等的函数
<br>
<br>

最后，可以将`qsort`定义为接收一个`pred`参数，返回一个以`pred`作为比较函数的`Quicksort`函数，从而实现不同的排序方式。
{% highlight clojure linenos %}
;; Quicksort
(defn qsort [pred]
  (fn qsort-help [[pivot & xs]]
    (when pivot
    (lazy-cat (qsort-help (filter #(pred % pivot) xs))
              [pivot]
              (qsort-help (remove #(pred % pivot) xs))))))
              
;; ascending order
(def ascending-quicksort (qsort <))
;; descending order
(def descending-quicksort (qsort >))
(def coll [9 6 5 4 9 0 4 8 6 2])
(ascending-quicksort coll)
;=> (0 2 4 4 5 6 6 8 9 9)
(descending-quicksort coll)
;=> (9 9 8 6 6 5 4 4 2 0)
{% endhighlight clojure %}
<br>

也可以给`qsort`函数再增加一个参数`pred`作为比较的函数，当没有传入`pred`时，默认按升序排列。
{% highlight clojure linenos %}
;; Quicksort
(defn qsort
  ([coll] (qsort coll <))
  ([[pivot & xs] pred]
    (when pivot
      (lazy-cat (qsort (filter #(pred % pivot) xs) pred)
                [pivot]
                (qsort (remove #(pred % pivot) xs) pred)))))
                
;; ascending order
(qsort coll)
;=> (0 2 4 4 5 6 6 8 9 9)
;; descending order
(qsort coll >)
;=> (9 9 8 6 6 5 4 4 2 0)
{% endhighlight clojure %}
<br>

好了，就这样，就这样水了一篇啦<(‾︶‾)>

