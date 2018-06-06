---
layout: post
title:  "Remote Docker API"
date:   2018-05-14 14:02:00 +0200
categories: General
---

Managing and enabling the Docker engine can be accomplished in many ways. Initially
I treated Docker much like I would treat a local installation. Which means I would
use Docker in a pretty inefficient method, and would rely on many other tools to
deploy containers to my environment. Then I looked into the Docker remote API and
suddenly realized how much effort I wasted...

# Docker Remote API

The Docker API by default points to the TCP port on the local host. This means that
only the local machine will have access to the Docker engine, but it can also
be configured to listen on the HTTP ports. What this means is that any Docker
host can be accessed remotely, which helps manage automation and orchestration.

## How to Enable

The Docker [website](https://docs.docker.com/engine/reference/commandline/dockerd/#examples)
provides a fairly comprehensive guide on enabling the remote API. Instead of
providing the same guide I instead want to talk about some of the advantages of
using the remote API.

## Remove Management

Once configured, any Docker engine can access the remote API by adding the `-H`
flag to the docker CLI. `docker -H <IP_ADDR> <command>`. If you prefer to not
have to add the flag to each command you can also set the `DOCKER_HOST` environment
variable.

## Automation

You can use orchestration tools such as Kubernetes or Swarm if you want a comprehensive
orchestration solution. For the situations that such a environment is not required
the Remote API is suitable, maybe just with plain old docker-compose. You can do
something like the following for example

```bash
export DOCKER_HOST=<IP_ADDR>
docker run -d nginx
```

which will start a nginx container on the remote host.
