---
layout: post
title:  Elasticsearch and Watches
date:   2018-06-06 14:02:00 +0200
categories: General
---

An important part of the DevOps lifecycle is monitoring and alerting. Often developers can overlook this aspect, since 
they traditionally did not always support the products they developed. Today, with so many products being deployed
to the cloud, it is much more likely that you will be the person responsible for your product. Of course, many tools
already exist to help monitor and manage your application, from [deeply integrated](https://www.appdynamics.com/) to
something a little [less intrusive](https://kubernetes.io/docs/concepts/cluster-administration/logging/). Regardless of
what applications or software you use, sooner or later you will want your monitoring to alert you to certain conditions.
This becomes more apparent when you realise that dashboards do not help much, when you do not have time to always look
at them.

Recently I took a detailed look at the different options available for alerting, and in particular focussed on what used
to be called [watcher](https://www.elastic.co/guide/en/x-pack/current/xpack-alerting.html). Now just called watches in
X-Pack, it provides a means to create alerts from an elastic cluster. Since it is not uncommon to see more organizations
adopt elastic, especially for collecting logs. The watch we will build in this article will alert on memory usage.

# ElastAlert

Previously we used [ElastAlert](https://github.com/Yelp/elastalert) to generate alerts from our cluster, and I may do a
future post about my experiences with it. But for now, I can say that if you don't have the money to pay for X-Pack, this
offering from Yelp is fully functional and great option to do alerts from elasticsearch data. Depending on the type of
alerts you may have to do a little more work to get this working, also if your team or yourself has limited experience
with python you may also find the solution to be a little frustrating to work with. But it is open source, and it can
be fun to step up to a new challenge...

# X-Pack Watches

Watches are the official solution provided by Elastic. This solution does require a X-Pack license in order to utilize
it. You can see that Elastic themselves are beginning to use Watcher extensively for other X-Pack features (Such as
monitoring), and have started providing a huge set of built in Watches for things like cluster status all the way to
incorrect versions in the Elastic Stack. Currently you can add a watch, either manually or via Kibana. The Kibana
solution is functional, but you will quickly find that it does not provide enough customization if you want to do more
complicated watches. The only watch that Kibana can provide at the moment is a threshold watch, and anything more complex
requires you to use the Watch API instead.

While the official [documentation](https://www.elastic.co/guide/en/x-pack/current/xpack-alerting.html) provides extensive
and detailed examples of how to configure and run watches. I thought I would instead create a watch using a real world
example. The example is to use [Metric Beat](https://www.elastic.co/products/beats/metricbeat) metrics to create a
alerts based on some of the more common alerts (disk usage, CPU usage and Memory Usage). I expect that eventually 
Elastic will provide these alerts as standard watches from metric beats, but until then I will walk you through creating
these watches.

## Prerequisites

To simplify the process I recommend that you get the [All in one stack](https://github.com/elastic/stack-docker). This will
provide you with a docker compose file that will start a elastic cluster with Kibana, Elastic and Beats already running.
Just clone the project to your machine, and run `docker-compose up`. What we will need for the rest of this example is
really just elasticsearch and metric beats, so if you want to build the solution a different way then just make sure 
you have those components working.

## Overview of a Watch

Watches are made up of a few components, a trigger, a input, a transform, a condition and a action. The trigger describes 
how often a watch should be run, for now it only supports cron like scheduling, but eventually it might also 
support other types of triggers. The input describes what sort of data should be fed into a watch, which comes in the 
form of an elastic query for the most part. The transform allows the results to be modified or cleaned before running
the condition check on the results, this is useful if you want to run scripts to update or change the data. A condition
incidates when a watch should perform an action, and can as simple as a comparision between two values, to running a script
to determine the condition. Finally, a set of actions is given for when the condition is triggered, each action will be
invoked on each true condition.

As indicated above, we want to try and build a watch that alerts us on the basic metrics from metric beat. So lets 
look at what data metricbeat sends to elastic...

## Output from Metricbeat

Metricbeat is an agent that reads and sends metric data to a couple of different destinations, the standard one is
elasticsearch, but you could also send to logstash, kafka or a file. Configuring and managing metric beat is not really
covered in this article, but elasticsearch does provide excellent documentation around metric beat. When you start 
looking at the results from metric beat, you could feel a little overwhelmed. The field data is pretty well
described and matches what you would expect. For our alerts we are interested in the _metricset.module: system_ events.
In particular, memory data is available under _metricset.name: memory_. 

Here is an example of the memory json, as you can see it contains a set of fields that describes the memory usage at a 
particular time.

```json
"system": {
  "memory": {
    "free": 20283392,
    "actual": {
      "free": 7004405760,
      "used": {
        "pct": 0.5923,
        "bytes": 10175463424
      }
    },
    "swap": {
      "total": 1073741824,
      "used": {
        "bytes": 477364224,
        "pct": 0.4446
      },
      "free": 596377600
    },
    "total": 17179869184,
    "used": {
      "bytes": 17159585792,
      "pct": 0.9988
    }
  }
}
```

The next section describes the queries to get the metric sets from the index...

## Elastic Queries

The query that will retrieve all of the memory metrics from the metric beat index is given below. 

```json
GET metricbeat-*/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": {"metricset.module": "system"} },
        { "match": {"metricset.name": "memory"} }
      ]
    }
  }
}
```

We could be fancier about it, but for the most part this will provide the simpliest options and help us use the watch for
the more complicated stuff. You can replace _"memory"_ with _"cpu"_ or _"filesystem"_ to get different metrics. Also, you
could get a lot of details around processes if you want from the _"process"_ metricsets, but this is beyond what we are
trying to achieve in this example.

## The First Watch

Our first watch will be pretty simple, and really it will tell us when our free memory is less than 2 gigs. You
can create a watch either in kibana via the watches in the kibana options, or via the API endpoint. Either way, you will
need to provide the watch as a json configuration given below

```json
{
  "trigger": {
    "schedule": {
      "interval": "30m"
    }
  },
  "input": {
    "search": {
      "request": {
        "search_type": "query_then_fetch",
        "indices": [
          "metricbeat-*"
        ],
        "types": [],
        "body": {
          "query": {
            "bool": {
              "must": [
                {
                  "match": {
                    "metricset.module": "system"
                  }
                },
                {
                  "match": {
                    "metricset.name": "memory"
                  }
                },
                {
                  "range": {
                    "system.memory.free": {
                      "lte": "2000000000"
                    }
                  }
                },
                {
                  "range": {
                    "@timestamp": {
                      "gte": "now-30m"
                    }
                  }
                }
              ]
            }
          }
        }
      }
    }
  },
  "condition": {
    "compare": {
      "ctx.payload.hits.total": {
        "gte": 1
      }
    }
  },
  "actions": {
    "my-logging-action": {
      "logging": {
        "level": "info",
        "text": "Free memory was less than 2 gigabytes."
      }
    }
  }
}
```

Breaking this down, the first part indicates that this watch is expected to run once every 30 minutes. The input is
a query on the indices looking for any memory metrics in the last 30 minutes that had less than 2 gigabytes of free 
memory. The condition simply looks at the payload and reports if any documents exists with that condition. Finally the 
action is to just log out that the condition was matched. Which is great for telling us if at any point in the last 
30 minutes, our free memory was less than 2 gigs. It does not however tell you when that actually occurred or what the
free memory actually was. So we could improve this watch and shall do in the next section...

## Where to next?

The simple watch is not really all that useful, you would probably want to know about free memory a little more often than
once every 30 minutes, so the first change is to reduce the window, to lets say a minute.

```json
"trigger": {
  "schedule": {
    "interval": "1m"
  }
},
"throttle_period": "10m"
```

You will also have to change the range input to 1 minute to be synconised. Else you will get 30 minute rolling windows,
which is probably not what you want.

But, we also don't really want to be spammed everytime the alert is triggered (until we have amazing automation systems)
in place. So lets also add a backoff, so that the message can only be sent once every ten minutes if it occurs. In the 
above example it is given on the entire alert, but you can also do it at the action level, so you could have a alert
being fired to send an email once every 10 minutes, but log it every time it occurs. You could also just set the schedule
to 10 minutes...

The next thing is that we would probably want the watch to be triggered off percentages instead of absolute values, in
the case of memory at least. So, remove the range constraint from the input query and instead add a transform that 
looks like this at the watch.

```json
"transform": {
  "script": {
    "source": """
      ctx.vars.result=new ArrayList();
      def hits = ctx.payload.hits.hits;
      for (int j = 0; j < hits.size(); j++) {
        ctx.vars.result.add(((double) hits[j]._source.system.memory.free / hits[j]._source.system.memory.total) * 100);
      }
    """
  }
},
```

This takes the query results and builds an array with each entry being the percentage of free memory for that record. The
above JSON might need to be flattened in order to work correctly, and is only spaced out like this to give readibility. 
It also uses *painless* scripting by default, which is provided by elastic as a scripting language. Refer to the elastic
documentation around the over scripting options.

Finally, we want to only alert when the memory usage is less than 20%, so we will need to change the condition clause as
well to something like the following...

```json
"condition": {
  "script" : {
    "source" : """
    def results = ctx.vars.results;
    for (int i=0; i < results.size(); i++) {
      if (result[i] > 20) {
        return true;
      }
    }
    return false;
  """
  }
}
```

Anytime a result has a value greater than 20 it will tell the watch to trigger. While this is better than our first watch
it still has problems, firstly it does not take into account the entire time period which is a minute. A lot could have
happened in a minute, such as going below 20 percent usage and then back above. Which means that the alert will not
be required anymore. It also does not take into account where that alert came from, in the case when you send multiple
hosts to the same metricbeat index. Finally, it might be normal for your memory usage to spike every now and then, which
is why elastic provide a machine learning component to X-Pack.

# Conclusions

The watches that can be built with elastic provide a comprehensive solution for triggering watches on data inside elastic
indices. While it is a powerful tool, it is also awkward to work with and can be confusing. Some of the built in watches 
can provide good examples of how to build more complicated watches. Hopefully elastic will eventually provide a cleaner
solution for managing and creating watches.