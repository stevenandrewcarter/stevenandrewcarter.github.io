---
layout: 	  post
title:  	  Elasticsearch Architecture
date:   	  2020-12-03 18:00:00 +0200
categories: General
published:  false
---

Over the last couple years I have built a few clusters and have made some observations around how to design and plan when
building a new cluster. Some of the considerations described here would also apply to other systems that have a similar
approach to scaling and redundancy. Also the designs discussed in this article should work on any version of elasticsearch
and the examples are built using Terraform.

# Single Node Elasticsearch Cluster

The first solution that almost everybody will build with elasticsearch would be the Single Node Elasticsearch Cluster.
Obviously this type of cluster is potentially not ideal in a production environment and would not be able to manage large
amounts of data either. 

[Single Node Elasticsearch Cluster](https://stevenandrewcarter.github.io/assets/SNEC.png "Single Node Elasticsearch Cluster")