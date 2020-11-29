+++
title = "CGSpace DSpace 6 Upgrade"
date = 2020-11-15T13:27:35+02:00
description = "Documenting the DSpace 6 upgrade."
categories = ["Notes"]
tags = ["Migration"]
url = "cgspace-dspace6-upgrade"
+++

Notes about the DSpace 6 upgrade on CGSpace in 2020-11.

<!--more-->

- [Re-import OAI with clean index](#re-import-oai-with-clean-index)
- [Processing Solr statistics with solr-upgrade-statistics-6x](#processing-solr-statistics-with-solr-upgrade-statistics-6x)
  - [Current year's statistics core](#statistics)
  - [statistics-2019 core](#statistics-2019)
  - [statistics-2018 core](#statistics-2018)
  - [statistics-2017 core](#statistics-2017)
  - [statistics-2016 core](#statistics-2016)
  - [statistics-2015 core](#statistics-2015)
  - [statistics-2014 core](#statistics-2014)
  - [statistics-2013 core](#statistics-2013)
  - [statistics-2013 core](#statistics-2012)
  - [statistics-2013 core](#statistics-2011)
  - [statistics-2013 core](#statistics-2010)
- [Processing Solr statistics with AtomicStatisticsUpdateCLI](processing-solr-statistics-with-atomicstatisticsupdatecli)


### Re-import OAI with clean index

After the upgrade is complete, re-index all items into OAI with a clean index:

```console
$ export JAVA_OPTS="-Dfile.encoding=UTF-8 -Xmx2048m"
$ dspace oai -c import
```

The process ran out of memory several times so I had to keep trying again with more JVM heap memory.


### Processing Solr Statistics With solr-upgrade-statistics-6x
After the main upgrade process was finished and DSpace was running I started processing the Solr statistics with `solr-upgrade-statistics-6x` to migrate all IDs to UUIDs.

## statistics
First process the current year's statistics core:

```console
$ export JAVA_OPTS='-Dfile.encoding=UTF-8 -Xmx2048m'
$ chrt -b 0 dspace solr-upgrade-statistics-6x -n 2500000 -i statistics
...
=================================================================
        *** Statistics Records with Legacy Id ***

           3,817,407    Bistream View
           1,693,443    Item View
             105,974    Collection View
              62,383    Community View
             163,192    Community Search
             162,581    Collection Search
             470,288    Unexpected Type & Full Site
        --------------------------------------
           6,475,268    TOTAL
=================================================================
```

After several rounds of processing it finished. Here are some statistics about unmigrated documents:

- 227,000: `(*:* NOT id:/.{36}/) AND (*:* NOT id:/.+-unmigrated/)`
- 471,000: `id:/.+-unmigrated/`
- 698,000: `*:* NOT id:/.{36}/`
- Majority are `type: 5` (aka SITE, according to `Constants.java`) so we can purge them:

```console
$ curl -s "http://localhost:8081/solr/statistics/update?softCommit=true" -H "Content-Type: text/xml" --data-binary "<delete><query>*:* NOT id:/.{36}/</query></delete>"
```

## statistics-2019
Processing the statistics-2019 core:

```console
$ chrt -b 0 dspace solr-upgrade-statistics-6x -n 2500000 -i statistics
...
=================================================================
        *** Statistics Records with Legacy Id ***

           5,569,344    Bistream View
           2,179,105    Item View
             117,194    Community View
             104,091    Collection View
             774,138    Community Search
             568,347    Collection Search
           1,482,620    Unexpected Type & Full Site
        --------------------------------------
          10,794,839    TOTAL
=================================================================
```

After several rounds of processing it finished. Here are some statistics about unmigrated documents:

- 2,690,309: `(*:* NOT id:/.{36}/) AND (*:* NOT id:/.+-unmigrated/)`
- 1,494,587: `id:/.+-unmigrated/`
- 4,184,896: `*:* NOT id:/.{36}/`
- 4,172,929 are `type: 5` (aka SITE) so we can purge them:

```console
$ curl -s "http://localhost:8081/solr/statistics-2019/update?softCommit=true" -H "Content-Type: text/xml" --data-binary "<delete><query>*:* NOT id:/.{36}/</query></delete>"
```

## statistics-2018
Processing the statistics-2018 core:

```console
$ chrt -b 0 dspace solr-upgrade-statistics-6x -n 2500000 -i statistics-2018
...
=================================================================
        *** Statistics Records with Legacy Id ***

           3,561,532    Bistream View
           1,129,326    Item View
              97,401    Community View
              63,508    Collection View
             207,827    Community Search
              43,752    Collection Search
             457,820    Unexpected Type & Full Site
        --------------------------------------
           5,561,166    TOTAL
=================================================================
```

After some time I got an error about Java heap space so I increased the JVM memory and restarted processing:

```console
$ export JAVA_OPTS='-Dfile.encoding=UTF-8 -Xmx4096m'
$ chrt -b 0 dspace solr-upgrade-statistics-6x -n 2500000 -i statistics-2018
```

Eventually the processing finished. Here are some statistics about unmigrated documents:

- 365,473: `(*:* NOT id:/.{36}/) AND (*:* NOT id:/.+-unmigrated/)`
- 546,955: `id:/.+-unmigrated/`
- 923,158: `*:* NOT id:/.{36}/`
- 823,293: are `type: 5` so we can purge them:

```console
$ curl -s "http://localhost:8081/solr/statistics-2018/update?softCommit=true" -H "Content-Type: text/xml" --data-binary "<delete><query>*:* NOT id:/.{36}/</query></delete>"
```

## statistics-2017

Processing the statistics-2017 core:

```console
$ chrt -b 0 dspace solr-upgrade-statistics-6x -n 2500000 -i statistics-2017
...
=================================================================
        *** Statistics Records with Legacy Id ***

           2,529,208    Bistream View
           1,618,717    Item View
             144,945    Community View
              74,249    Collection View
             479,647    Community Search
             114,658    Collection Search
             852,215    Unexpected Type & Full Site
        --------------------------------------
           5,813,639    TOTAL
=================================================================
```

Eventually the processing finished. Here are some statistics about unmigrated documents:

- 808,309: `(*:* NOT id:/.{36}/) AND (*:* NOT id:/.+-unmigrated/)`
- 893,868: `id:/.+-unmigrated/`
- 1,702,177: `*:* NOT id:/.{36}/`
- 1,660,524 are `type: 5` (SITE) so we can purge them:

```console
$ curl -s "http://localhost:8081/solr/statistics-2017/update?softCommit=true" -H "Content-Type: text/xml" --data-binary "<delete><query>*:* NOT id:/.{36}/</query></delete>"
```

## statistics-2016

Processing the statistics-2016 core:

```console
$ chrt -b 0 dspace solr-upgrade-statistics-6x -n 2500000 -i statistics-2016
...
=================================================================
        *** Statistics Records with Legacy Id ***

           1,765,924    Bistream View
           1,151,575    Item View
             187,110    Community View
              51,204    Collection View
             347,382    Community Search
              66,605    Collection Search
             620,298    Unexpected Type & Full Site
        --------------------------------------
           4,190,098    TOTAL
=================================================================
```

- 849,408: `(*:* NOT id:/.{36}/) AND (*:* NOT id:/.+-unmigrated/)`
- 627,747: `id:/.+-unmigrated/`
- 1,477,155: `*:* NOT id:/.{36}/`
- 1,469,706 are `type: 5` (SITE) so we can purge them:

```console
$ curl -s "http://localhost:8081/solr/statistics-2016/update?softCommit=true" -H "Content-Type: text/xml" --data-binary "<delete><query>*:* NOT id:/.{36}/</query></delete>"
```


## statistics-2015

Processing the statistics-2015 core:

```console
$ chrt -b 0 dspace solr-upgrade-statistics-6x -n 2500000 -i statistics-2015
...
=================================================================
        *** Statistics Records with Legacy Id ***

             990,916    Bistream View
             506,070    Item View
             116,153    Community View
              33,282    Collection View
              21,062    Community Search
              10,788    Collection Search
              52,107    Unexpected Type & Full Site
        --------------------------------------
           1,730,378    TOTAL
=================================================================
```

Summary of stats after processing:

- 195,293: `(*:* NOT id:/.{36}/) AND (*:* NOT id:/.+-unmigrated/)`
- 67,146: `id:/.+-unmigrated/`
- 262,439: `*:* NOT id:/.{36}/`
- 247,400 are `type: 5` (SITE) so we can purge them:

```console
$ curl -s "http://localhost:8081/solr/statistics-2015/update?softCommit=true" -H "Content-Type: text/xml" --data-binary "<delete><query>*:* NOT id:/.{36}/</query></delete>"
```


## statistics-2014

Processing the statistics-2014 core:

```console
$ chrt -b 0 dspace solr-upgrade-statistics-6x -n 2500000 -i statistics-2014
...
=================================================================
        *** Statistics Records with Legacy Id ***

           2,381,603    Item View
           1,323,357    Bistream View
             501,545    Community View
             247,805    Collection View
                 250    Collection Search
                 188    Community Search
                  50    Item Search
              10,918    Unexpected Type & Full Site
        --------------------------------------
           4,465,716    TOTAL
=================================================================
```

Summary of unmigrated documents after processing:

- 182,131: `(*:* NOT id:/.{36}/) AND (*:* NOT id:/.+-unmigrated/)`
- 39,947: `id:/.+-unmigrated/`
- 222,078: `*:* NOT id:/.{36}/`
- 188,791 are `type: 5` (SITE) so we can purge them:

```console
$ curl -s "http://localhost:8081/solr/statistics-2014/update?softCommit=true" -H "Content-Type: text/xml" --data-binary "<delete><query>*:* NOT id:/.{36}/</query></delete>"
```


## statistics-2013

Processing the statistics-2013 core:

```console
$ chrt -b 0 dspace solr-upgrade-statistics-6x -n 2500000 -i statistics-2013
...
=================================================================
        *** Statistics Records with Legacy Id ***

           2,352,124    Item View
           1,117,676    Bistream View
             575,711    Community View
             171,639    Collection View
                 248    Item Search
                   7    Collection Search
                   5    Community Search
               1,452    Unexpected Type & Full Site
        --------------------------------------
           4,218,862    TOTAL
=================================================================
```

Summary of unmigrated docs after processing:

- 2,548 : `(*:* NOT id:/.{36}/) AND (*:* NOT id:/.+-unmigrated/)`
- 29,772: `id:/.+-unmigrated/`
- 32,320: `*:* NOT id:/.{36}/`
- 15,691 are `type: 5` (SITE) so we can purge them:

```console
$ curl -s "http://localhost:8081/solr/statistics-2013/update?softCommit=true" -H "Content-Type: text/xml" --data-binary "<delete><query>*:* NOT id:/.{36}/</query></delete>"
```


## statistics-2012

Processing the statistics-2012 core:

```console
$ chrt -b 0 dspace solr-upgrade-statistics-6x -n 2500000 -i statistics-2012
...
=================================================================
        *** Statistics Records with Legacy Id ***

           2,229,332    Item View
             913,577    Bistream View
             215,577    Collection View
             104,734    Community View
        --------------------------------------
           3,463,220    TOTAL
=================================================================
```

Summary of unmigrated docs after processing:

- 0: `(*:* NOT id:/.{36}/) AND (*:* NOT id:/.+-unmigrated/)`
- 33,161: `id:/.+-unmigrated/`
- 33,161: `*:* NOT id:/.{36}/`
- 33,161 are `type: 3` (COLLECTION), which is different than I've seen previously... but I suppose I still have to purge them because there will be errors in the Atmire modules otherwise:

```console
$ curl -s "http://localhost:8081/solr/statistics-2012/update?softCommit=true" -H "Content-Type: text/xml" --data-binary "<delete><query>*:* NOT id:/.{36}/</query></delete>"
```


## statistics-2011

Processing the statistics-2011 core:

```console
$ chrt -b 0 dspace solr-upgrade-statistics-6x -n 2500000 -i statistics-2011
...
=================================================================
        *** Statistics Records with Legacy Id ***

             904,896    Item View
             385,789    Bistream View
             154,356    Collection View
              62,978    Community View
        --------------------------------------
           1,508,019    TOTAL
=================================================================
```

Summary of unmigrated docs after processing:

- 0: `(*:* NOT id:/.{36}/) AND (*:* NOT id:/.+-unmigrated/)`
- 17,551: `id:/.+-unmigrated/`
- 17,551: `*:* NOT id:/.{36}/`
- 12,116 are `type: 3` (COLLECTION), which is different than I've seen previously... but I suppose I still have to purge them because there will be errors in the Atmire modules otherwise:

```console
$ curl -s "http://localhost:8081/solr/statistics-2011/update?softCommit=true" -H "Content-Type: text/xml" --data-binary "<delete><query>*:* NOT id:/.{36}/</query></delete>"
```


## statistics-2010

Processing the statistics-2010 core:

```console
$ chrt -b 0 dspace solr-upgrade-statistics-6x -n 2500000 -i statistics-2010
...
=================================================================
        *** Statistics Records with Legacy Id ***

              26,067    Item View
              15,615    Bistream View
               4,116    Collection View
               1,094    Community View
        --------------------------------------
              46,892    TOTAL
=================================================================
```

Summary of unmigrated docs after processing:

- 0: `(*:* NOT id:/.{36}/) AND (*:* NOT id:/.+-unmigrated/)`
- 1,012: `id:/.+-unmigrated/`
- 1,012: `*:* NOT id:/.{36}/`
- 654 are `type: 3` (COLLECTION), which is different than I've seen previously... but I suppose I still have to purge them because there will be errors in the Atmire modules otherwise:

```console
$ curl -s "http://localhost:8081/solr/statistics-2010/update?softCommit=true" -H "Content-Type: text/xml" --data-binary "<delete><query>*:* NOT id:/.{36}/</query></delete>"
```


### Processing Solr statistics with AtomicStatisticsUpdateCLI

On 2020-11-18 I finished processing the Solr statistics with solr-upgrade-statistics-6x and I started processing them with AtomicStatisticsUpdateCLI.

## statistics

First the current year's statistics core, in 12-hour batches:

```
$ chrt -b 0 dspace dsrun com.atmire.statistics.util.update.atomic.AtomicStatisticsUpdateCLI -t 12 -c statistics
```

It took ~38 hours to finish processing this core.

## statistics-2019

The statistics-2019 core, in 12-hour batches:

```
$ chrt -b 0 dspace dsrun com.atmire.statistics.util.update.atomic.AtomicStatisticsUpdateCLI -t 12 -c statistics-2019
```

It took ~32 hours to finish processing this core.

## statistics-2018

The statistics-2018 core, in 12-hour batches:

```
$ chrt -b 0 dspace dsrun com.atmire.statistics.util.update.atomic.AtomicStatisticsUpdateCLI -t 12 -c statistics-2018
```

It took ~28 hours to finish processing this core.

## statistics-2017

The statistics-2017 core, in 12-hour batches:

```
$ chrt -b 0 dspace dsrun com.atmire.statistics.util.update.atomic.AtomicStatisticsUpdateCLI -t 12 -c statistics-2017
```

It took ~24 hours to finish processing this core.

## statistics-2016

The statistics-2016 core, in 12-hour batches:

```
$ chrt -b 0 dspace dsrun com.atmire.statistics.util.update.atomic.AtomicStatisticsUpdateCLI -t 12 -c statistics-2016
```

It took ~20 hours to finish processing this core.

## statistics-2015

The statistics-2015 core, in 12-hour batches:

```
$ chrt -b 0 dspace dsrun com.atmire.statistics.util.update.atomic.AtomicStatisticsUpdateCLI -t 12 -c statistics-2015
```

It took ~17 hours to finish processing this core.

## statistics-2014

The statistics-2014 core, in 12-hour batches:

```
$ chrt -b 0 dspace dsrun com.atmire.statistics.util.update.atomic.AtomicStatisticsUpdateCLI -t 12 -c statistics-2014
```

It took ~12 hours to finish processing this core.

## statistics-2013

The statistics-2013 core, in 12-hour batches:

```
$ chrt -b 0 dspace dsrun com.atmire.statistics.util.update.atomic.AtomicStatisticsUpdateCLI -t 12 -c statistics-2013
```

It took ~3 hours to finish processing this core.

## statistics-2012

The statistics-2012 core, in 12-hour batches:

```
$ chrt -b 0 dspace dsrun com.atmire.statistics.util.update.atomic.AtomicStatisticsUpdateCLI -t 12 -c statistics-2012
```

It took ~2 hours to finish processing this core.

## statistics-2011

The statistics-2011 core, in 12-hour batches:

```
$ chrt -b 0 dspace dsrun com.atmire.statistics.util.update.atomic.AtomicStatisticsUpdateCLI -t 12 -c statistics-2011
```

It took 1 hour to finish processing this core.

## statistics-2010

The statistics-2010 core, in 12-hour batches:

```
$ chrt -b 0 dspace dsrun com.atmire.statistics.util.update.atomic.AtomicStatisticsUpdateCLI -t 12 -c statistics-2010
```

It took five minutes to finish processing this core.
