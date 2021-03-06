# Balagan, Clojure (Script) data transformation and querying library

A tiny library for data structure transformation inspired by [Enlive](https://github.com/cgrand/enlive).

It's often hard to come up with a sensible DSL for data structure transformation. You either get
too close to the data domain, and your functions get messy and difficult to write (or even maybe
too nested) or you're too far away from data, and your functions operate nested structures as if
they were plain.

Bałagan solves that problem for you by providing you with predicate-based selectors and arbitrary
transformation functions that suits any taste. It's mostly like `get-in` and `update-in` on steroids, 
that allows you to build predicate queries, not only concrete ones.

## Project Maturity

Balagan is a fairly young project, but it's been used in production for around a year by now, 
and has proven itself to be safe and stable.

## Artifacts

Bałagan artifacts are [released to Clojars](https://clojars.org/clojurewerkz/balagan). If you are using Maven, add the following repository definition to your `pom.xml`:

``` xml
<repository>
  <id>clojars.org</id>
  <url>http://clojars.org/repo</url>
</repository>
```

### Most recent release version

With Leiningen:

```clojure
[clojurewerkz/balagan "1.0.0"]
```

With Maven:

```xml
<dependency>
  <groupId>clojurewerkz</groupId>
  <artifactId>balagan</artifactId>
  <version>1.0.0</version>
</dependency>
```


## Documentation and Examples

Balagan builds on ideas from [Enlive](https://github.com/cgrand/enlive). It may help to get familiar with them
first.

Let's say you're working on a Data Access layer for the application. You have a user entry
represented as hash:

```clojure
(def user
  {:name "Alex"
   :birth-year 1990
   :nickname "ifesdjeen"})
```

### Transformation

Now, we can start transforming users the way we want: add, remove fields based on certain conditions.

```clojure
(update user
           []                  (add-field :cool-dude true) ;; adds a field :cool-dude with value true
           (new-path [:age])   #(- 2014 (:birth-year %))   ;; explicit adding of a new field, calculated from the existing data
           (new-path [:posts]) #(fetch-posts (:name %))    ;; fetching some related data from the DB
           [:posts :*]         #(update-posts %))       ;; apply some transformations to all the fetched posts, if there are any
```

### Queries

Queries are very similar to how you'd query your data with `filter` in Clojure:

```clj
(let [data {:a {:b [{:c 1} {:c 2} {:c 3}]
                :d [{:c 5} {:c 6} {:c 7}]}}]
  (select data  [:* :* even? :c]))
;; => (1 3 5 7)  
```

Results are returned in the order they've been seen in your data structure, however you should be aware of the 
fact that iterating over the hash in Clojure doesn't guarantee you order.

### Path Queries

Path queries are most useful when you'd like to fire a function against some part of your data (be it processing,
database initialization or anything else.

You can also run predicate queries based on your map, for example if you want to configure your database servers
from rather big and complex config:

```clj
(def conf {:db
           {:redis {:cache  [{:host "host01" :port 1234} {:host "host02" :port 1234}]
                    :pubsub [{:host "host01" :port 1234} {:host "host02" :port 1234}]}}
           :cassandra [{:host "host01"} {:host "host02"}]})

(with-paths conf
        [:db :redis :cache]  configure-redis-cache
        [:db :redis :pubsub] configure-redis-pubsub
        [:db :cassandra]     configure-cassandra)
```

In this example, `configure-redis-cache` funciton will receive two arguments: `value` and `path`:

```clj
(defn configure-redis-cache
  [value path]
  (println "Value: " value)
  (println "Path: " path))

;; => Value: [{:host host01, :port 1234} {:host host02, :port 1234}]
;; => Path: [:db :redis :cache]
```

You can also do wildcard-matching with `:*`, for example: 

```clj
(b/select {:a {:b {:c 1} :d {:c 2}}}
          [:a :* :c] (fn [val path]
                       (if (= path [:a :b :c])
                         (is (= val 1))
                         (is (= val 2)))))
```

## Community

To subscribe for announcements of releases, important changes and so on, please follow
[@ClojureWerkz](https://twitter.com/#!/clojurewerkz) on Twitter.



## Balagan Is a ClojureWerkz Project

Balagan is part of the [group of libraries known as ClojureWerkz](http://clojurewerkz.org), together with
[Monger](https://clojuremongodb.info), [Welle](https://clojureriak.info), [Neocons](https://clojureneo4j.info),
[Elastisch](https://clojureelasticsearch.info) and several others.


## Continuous Integration

[![Continuous Integration status](https://secure.travis-ci.org/clojurewerkz/balagan.png)](http://travis-ci.org/clojurewerkz/balagan)

CI is hosted by [travis-ci.org](http://travis-ci.org)


## Development

Balagan uses [Leiningen 2](https://github.com/technomancy/leiningen/blob/master/doc/TUTORIAL.md). Make
sure you have it installed and then run tests against all supported Clojure versions using

```
lein do clean, cljx once, all test, cljsbuild test
```

Then create a branch and make your changes on it. Once you are done with your changes and all
tests pass, submit a pull request on Github.


## Balagan?

`Hash` is a synonym of `mess`, Bałagan means `mess` in several Slavic
languages.


## License

Copyright © 2014 Alex P, Michael S. Klishin

Distributed under the Eclipse Public License, the same as Clojure.
