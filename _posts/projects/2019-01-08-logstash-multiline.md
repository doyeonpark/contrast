---
title:  "Logstash의 멀티라인 세팅하기"
date:   2019-10-28 22:00:00
tags: [logstash, filebeat, elk, logger]
categories: posts
---

hostsedit
Value type is uri
Default value is [//127.0.0.1]
Sets the host(s) of the remote instance. If given an array it will load balance requests across the hosts specified in the hosts parameter. Remember the http protocol uses the http address (eg. 9200, not 9300). "127.0.0.1" ["127.0.0.1:9200","127.0.0.2:9200"] ["http://127.0.0.1"] ["https://127.0.0.1:9200"] ["https://127.0.0.1:9200/mypath"] (If using a proxy on a subpath) It is important to exclude dedicated master nodes from the hosts list to prevent LS from sending bulk requests to the master nodes. So this parameter should only reference either data or client nodes in Elasticsearch.

Any special characters present in the URLs here MUST be URL escaped! This means # should be put in as %23 for instance.