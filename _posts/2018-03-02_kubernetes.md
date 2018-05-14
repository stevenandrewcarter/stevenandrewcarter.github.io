---
layout: post
title:  "Kubernetes and NodeJS"
date:   2018-03-02 14:02:00 +0200
categories: kubernetes
published: false
---

In the current world of microservices we want to dynamically scale and introduce
services as needed. Kubernetes provides a platform for managing microservices,
and is currently the market leader in orchestration. Although you don't always
want to expose your kubernetes internals to everybody. So for the purpose of
this article is looking at starting up a set of web servers as kubernetes
containers. Using a simple NodeJS express API, I will show you how to
provision web sites on demand.

# Configure the API
