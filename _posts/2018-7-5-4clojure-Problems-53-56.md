---
layout: post
title: 4clojure-Problems-53-56
---

## 53 - Longest Increasing Subseq

Given a vector of integers, find the longest consecutive sub-sequence of 2 or more increasing numbers.

<pre><code class="language-klipse">(ns live.test
  (:require [cljs.test :refer-macros [deftest is testing run-tests]]))

(defn longest-subseq [s]
  (or (first (for [l (reverse (range 2 (count s)))
                   f (filter #(apply < %) (partition l 1 s))]
               f)) []))
  
(deftest test-numbers
  (is (= (longest-subseq [1 0 1 2 3 0 4 5]) [0 1 2 3]))
  (is (= (longest-subseq [5 6 1 3 2 7]) [5 6]))
  (is (= (longest-subseq [2 3 3 4 5]) [3 4 5]))
  (is (= (longest-subseq [7 6 5 4]) [])))
  
(run-tests)
</code></pre>

## 54 - Partition

Returns a sequence of lists of no less than x items each.

Source for ```partition``` function:

    user=> (source partition)
    (defn partition                                
    "Returns a lazy sequence of lists of n items each, at offsets step
    apart. If step is not supplied, defaults to n, i.e. the partitions
    do not overlap. If a pad collection is supplied, use its elements as
    necessary to complete last partition upto n items. In case there are
    not enough padding elements, return a partition with less than n items."
    {:added "1.0"
     :static true}
    ([n coll]
      (partition n n coll))
    ([n step coll]
     (lazy-seq
       (when-let [s (seq coll)]
         (let [p (doall (take n s))]
           (when (= n (count p))
             (cons p (partition n step (nthrest s step))))))))
    ([n step pad coll]
     (lazy-seq
       (when-let [s (seq coll)]
         (let [p (doall (take n s))]
           (if (= n (count p))
             (cons p (partition n step pad (nthrest s step)))
             (list (take n (concat p pad)))))))))
    nil

<pre><code class="language-klipse">(ns live.test
  (:require [cljs.test :refer-macros [deftest is testing run-tests]]))

(defn partition-seq [n s]
  (loop [c s partitioned []] (if (< (count c) n) partitioned (recur (drop n c) (conj partitioned (take n c))))))
  
(deftest partition-test
  (is (= (partition-seq 3 (range 9)) '((0 1 2) (3 4 5) (6 7 8))))
  (is (= (partition-seq 2 (range 8)) '((0 1) (2 3) (4 5) (6 7))))
  (is (= (partition-seq 3 (range 8)) '((0 1 2) (3 4 5)))))

(run-tests)
</code></pre>

## 55 - Frequencies

Returns a map containing the number of occurences of each distinct item in a sequence.

Here is the source for the ```frequencies``` function:

    user=> (source frequencies)
    (defn frequencies
    "Returns a map from distinct items in coll
    to the number of times they appear."
    {:added "1.2"
    :static true}
    [coll]
    (persistent!
      (reduce (fn [counts x]
             (assoc! counts x (inc (get counts x 0))))
           (transient {}) coll)))
    nil

<pre><code class="language-klipse">(ns live.test
  (:require [cljs.test :refer-macros [deftest is testing run-tests]]))

(defn freqs [s]
  (apply merge-with + (for [i s] {i 1})))
  
(deftest partition-test
  (is (= (freqs [1 1 2 3 2 1 1]) {1 4, 2 2, 3 1}))
  (is (= (freqs [:b :a :b :a :b]) {:a 2, :b 3}))
  (is (= (freqs '([1 2] [1 3] [1 3])) {[1 2] 1, [1 3] 2})))

(run-tests)
</code></pre>

### 56 - Distinct

Remove duplicate items from a sequence. Order must be maintained.

While we are not allowed to use the ```distinct``` function (because that would be too easy), we can still examine its source code to inform our answer:

    user=> (source distinct)
    (defn distinct
    "Returns a lazy sequence of the elements of coll with duplicates removed.
    Returns a stateful transducer when no collection is provided."
    {:added "1.0"
     :static true}
    ([]
    (fn [rf]
     (let [seen (volatile! #{})]
       (fn
         ([] (rf))
         ([result] (rf result))
         ([result input]
          (if (contains? @seen input)
            result
            (do (vswap! seen conj input)
                (rf result input))))))))
    ([coll]
    (let [step (fn step [xs seen]
                (lazy-seq
                  ((fn [[f :as xs] seen]
                     (when-let [s (seq xs)]
                       (if (contains? seen f)
                         (recur (rest s) seen)
                         (cons f (step (rest s) (conj seen f))))))
                   xs seen)))]
     (step coll #{}))))
    nil


This exercise continues the theme of taking a built-in function and asking us to re-implement it another way. In this case, it turns out we can get it way more concise:

<pre><code class="language-klipse">(ns live.test
  (:require [cljs.test :refer-macros [deftest is testing run-tests]]))
  
(defn deduper [s]
    (reduce #(if ((set %) %2) % (conj % %2)) [] s))
  
(deftest deduper-test
  (is (= (deduper [1 2 1 3 1 2 4]) [1 2 3 4]))
  (is (= (deduper [:a :a :b :b :c :c]) [:a :b :c]))
  (is (= (deduper '([2 4] [1 2] [1 3] [1 3])) '([2 4] [1 2] [1 3])))
  (is (= (deduper (range 50)) (range 50))))

(run-tests)
</code></pre>

