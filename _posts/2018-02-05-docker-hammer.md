---
layout: post
title:  "When Docker is the tool, everything is a container"
date:   2018-02-19 14:02:00 +0200
categories: General
---

I recently had an interesting situation occur with my experiences with containers
and Docker. What had happened is that we are building a application using the meteor
framework, and in order to make the process as containerized as possible, we ended
up putting most of the development environment into containers. For those who don't
know, meteor is a Javascript framework that is about a gig in size. And it is not
needed for the final production application, instead it is used to build the application.

So, nothing really weird at the moment. We could just create a base Docker container with
meteor, and with a multi-stage build end up with something like this...

```
FROM base-meteor:latest
ADD ./src /meteor
RUN meteor build --production

from node:latest
COPY --from=0 /meteor/output /application
RUN node /application/server.js
```

The first part of the Dockerfile is building the build container, and the second part is taking the result of the first 
as the application to run. This basically means we end up with a pretty small node based container (~600mb) instead
of the base meteor container (~1gb). Of course, we didn't stop there...

# The Hammer

So we wanted to make everything in the build process also a container, it makes sense right?
The devs can test the build process locally and know that it will use the same process on the
build server. It sounds great...until we tried it. The first problem we ran into is that
the testing pipeline for meteor does not look anything like the above two images. In fact
it does not even match anything the multi-stage build, so that means we would need another
Dockerfile for the testing, something like this...

```
FROM base-meteor:latest
ADD ./src /meteor
CMD npm test
```

That doesn't seem too bad right? Except that it actually doesn't work as it turns out. The
first problem we encountered is that the container doesn't correctly run the tests, you see
meteor also has client tests that require a browser to execute against. Lets fix the Dockerfile
to do that...

```
FROM base-meteor:latest
ADD ./src /meteor
CMD npm run-script meteor-full-test
```

Of course, the assumption here is that the base-meteor image has a browser. Which it turns out
it does not. In fact, without running the image as a container, its not even clear what
the base-meteor uses as its base Linux libraries either. So we check the base-meteor to find
its distro. Which turns out to be Debian in this case. Now all we need to do is install a
headless browser to run the meteor tests. The first choice, chrome headless, turns out to
be a real pain to get running in a docker container, with little to no feedback during failures
and no help to be found. The next choice is to run it with Electron/Nightmare, which
seems to at least run (but fails) with something to work with.

Turns out, that the container also needs some sort of windows manager to actually render the
client tests in. Once again, luckily for us, there is a tool called Xvfb that allows us to
run a virtualized window manager. Of course, it does make the Dockerfile a little more complicated

```
FROM base-meteor:latest
ADD ./src /meteor
RUN apt-get install -y \
  xvfb \
  x11-xkb-utils \
  xfonts-100dpi \
  xfonts-75dpi \
  xfonts-scalable \
  xfonts-cyrillic \
  x11-apps \
  clang \
  libdbus-1-dev \
  libgtk2.0-dev \
  libnotify-dev \
  libgnome-keyring-dev \
  libgconf2-dev \
  libasound2-dev \
  libcap-dev \
  libcups2-dev \
  libxtst-dev \
  libxss1 \
  libnss3-dev \
  gcc-multilib \
  g++-multilib
CMD npm run-script meteor-full-test
```

Wonderful, our initial container now installs a whole load of packages so that the client tests
can run in a container too. Which on the build server, will happen everytime we make a build.
Which increases the test step to nearly 20 minutes from less than a minute if you just run the tests.

# Everything is a Container

Of course, something needs to be done to make this solution even worth considering. A couple
of suggestions I would make, the first is that the base-meteor could be edited to include the
xvfb libraries as well. This would at least move the build time to the base container, which
is unlikely to be built with every build. Another option, is to create a base test container
if the base-meteor container should not be meddled with.

Finally, the last suggestion, is that maybe this solution does not need a container. Containers
are about making the production and development environment identical for the application,
not for making the development environment into a containerized mess. Developers are fairly
unlikely to run tests exclusively inside a container. And since all it really achieves is making
the tests really hard to work with, it might be worth considering installing the required
components on the build server to run the tests.

Sometimes, we can use other things besides hammers...
