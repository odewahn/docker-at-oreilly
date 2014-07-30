# Docker Experiments at O'Reilly Media

<img src="images/docker-at-orm-cover.png"/>


Docker Boston Meetup

July 30, 2014

Andrew Odewahn (odewahn@oreilly.com)

# About O'Reilly Media

<img style="width: 50%" src="images/about-orm.png"/>

* [O'Reilly Media](http://www.oreilly.com/) produces technical books and events
* Both "Web 2.0" and "Open Source" were terms that came from O'Reilly events
* Founded in Cambridge, MA, but now headquartered in Sebastopol, CA (north of San Francisco)

# At heart, O'Reilly is a learning company

<p style="font-size:24px">"Spreading the knowledge of innovators"  -- O'Reilly exists to take the knowledge in an experts head and
	package it up so that other people can learn it.</p>  

# The way people want to learn is changing radically.

<img src="images/new-learning-tools.png"/>

* How do we create media products like these using our core capabilities (editorial, brand, community)
* How do we transform as media is increasingly becoming software
* Exploring Docker to help us make new kinds of media products


# Opportunity in iPython Notebooks

<video autoplay="true" loop="true" muted="true" width="720"><source src="https://s3.amazonaws.com/orm-atlas-media/ipynb.webm" type="video/webm"> <source src="https://s3.amazonaws.com/orm-atlas-media/ipynb.mp4" type="video/mp4"> Your browser does not support the video tag.</video>
	
* [iPython Notebooks](http://ipython.org/notebook.html) are becoming the defacto tool in the scientific and big data science communities
* Provides authoring and execution environment for text, math, and arbitrary code (Python, Julia, R, and Ruby)
* Strong demand among our authors to support this format
* Plus, it's awesome

# How can we deliver iPython Notebooks as a new kind of learning experience?

* How do authors create them?
* How do we produce them (edit, copyedit, illustrate, index, etc)?
* How do we get that content out in other formats (print, MOBI, Safari, etc)?
* How do we distribute them to make a compelling experience?

We're exploring Docker as a key solution for each of these questions.

# Case Study 1: Packaging the examples for _Python for Data Analysis_ as a Docker image

<img style="width: 60%" src="images/python-data-analysis.jpg"/>

* Successful book in the "Data Science Area" published in 2012
* This is a rapidly changing area
* Create a companion product as an iPython Notebook

# DEMO

* Install [boot2docker](https://github.com/boot2docker/boot2docker) (*NB: if you have an older version of Boot2Docker, here's [a great article on how to upgrade](http://blog.javabien.net/2014/03/17/upgrade-docker-and-boot2docker-on-osx/)*)
* Set up an account on [docker.com](https://hub.docker.com/)
* Expose port 8888 in Virtualbox (do this just once)

```
VBoxManage controlvm boot2docker-vm natpf1 "ipython-notebook,tcp,127.0.0.1,8888,,8888"
```

* Start boot2docker and ssh into the box
* Pull [odewahn/python-data-analysis](https://registry.hub.docker.com/u/odewahn/python-data-analysis/).  (*NB: This is a big image -- 3GB+*)

```
sudo docker pull odewahn/python-data-analysis
```

* Start the container, and be sure to expose port 8888

```
sudo docker run -i -t -p 8888:8888 odewahn/python-data-analysis /bin/bash
```

* Once the container starts and your at the bash prompt, start the server with this command:

```
./start.sh
```

* Go to localhost:8888 on your local browser

## Here's a quick video that shows this in action

<video autoplay="true" loop="true" muted="true" width="720"><source src="https://s3.amazonaws.com/orm-atlas-media/boot2docker-demo.mp4" type="video/mp4"> Your browser does not support the video tag.</video>


# How do we go beyond companion pieces and make actual products?

<img src="images/atlas-splashpage.png">

* Companion products are great, but how do we make actual products themselves?
* We use an internally developed tool call [O'Reilly Atlas](https://atlas.oreilly.com/) for 80% of our content.

# Atlas Key Concepts

<img src="images/atlas-key-ideas.png">

# A single source for semantic content

<video autoplay="true" loop="true" muted="true" width="720"><source src="https://s3.amazonaws.com/orm-atlas-media/introducingatlas/visual_editor.webm" type="video/webm"> <source src="https://s3.amazonaws.com/orm-atlas-media/introducingatlas/visual_editor.mp4" type="video/mp4"> Your browser does not support the video tag.</video>
	
	
* [HTMLBook](http://oreillymedia.github.io/HTMLBook/)
* [Markdown](http://daringfireball.net/projects/markdown/)
* [AsciiDoc](http://www.methods.co.nz/asciidoc/)
* [DocBook XML](http://www.docbook.org/)

# Version Control

<img src="images/atlas-github.png">

* All Atlas content is stored in Git.
* This presentation was created in Atlas and posted to [Github](https://github.com/odewahn/docker-at-oreilly)

	
# Transformation engines

<video autoplay="true" loop="true" muted="true" width="720"><source src="https://s3.amazonaws.com/orm-atlas-media/introducingatlas/make_a_book.webm" type="video/webm"> <source src="https://s3.amazonaws.com/orm-atlas-media/introducingatlas/make_a_book.mp4" type="video/mp4"> Your browser does not support the video tag.</video>
	
* Print books (80% of titles published through ORM)
* EPUB
* MOBI
* Web Sites
  * [Raspberry Pi Cookbook](http://razzpisampler.oreilly.com/)
  * [Interactive Data Visualization for the Web](http://chimera.labs.oreilly.com/books/1230000000345)
  * [Etudes for Erlang](http://chimera.labs.oreilly.com/books/1234000000726)

# A Docker toolchain for transforming Atlas Projects to ipynb format

<img src="images/atlas2ipynb.png">

* The [atlas2ipynb gem](https://github.com/odewahn/atlas2ipynb) gem transform HTMLBook into iPython Notebook's JSON-based format


# A Dockerfile for a base Docker image

```
FROM ubuntu
MAINTAINER Andrew Odewahn "odewahn@oreilly.com"

RUN apt-get update
RUN apt-get install -y ruby1.9.3
RUN apt-get install -y python-software-properties python-dev python-pip
RUN apt-get install -y libfreetype6-dev libpng-dev libncurses5-dev vim git-core build-essential curl unzip wget

# Install Atlas-specific gems
RUN gem install bundler atlas-api atlas2ipynb

# Install ipython notebook requirements
RUN pip install --upgrade pip
ADD requirements.txt /tmp/requirements.txt
RUN pip install numpy==1.7.1
RUN pip install -r /tmp/requirements.txt --allow-unverified matplotlib --allow-all-external

#
# Create the command to actually run the ipython notebook
#
RUN adduser --disabled-password --home=/home/atlas --gecos "" atlas
USER atlas
WORKDIR /home/atlas
RUN echo '#!/bin/sh' > start.sh
RUN echo 'ipython notebook --ip=0.0.0.0 --port=8888 --pylab=inline --no-browser'  >> start.sh
RUN chmod +x start.sh

#
# Set us back to the root user
#
USER root
```

* An [atlas-base](https://github.com/odewahn/ipython-docker/blob/master/base/Dockerfile) Docker image


# A Dockerfile for each book

```
FROM odewahn/atlas-base
MAINTAINER Andrew Odewahn "odewahn@oreilly.com" 

#
# Author can add any dependencies he or she requires
#

#
# Use atlas-api to build the project and install it on the docker image
#
USER atlas
WORKDIR /home/atlas
RUN atlas2ipynb $ATLAS_KEY odewahn/atlas2ipynb-sample
```

* An author can include a Dockerfile in his or her Atlas project and use it to create an interactive iPython Notebook

# Case Study 2: Just Enough Math

<img src="images/jem-formats.png"/>

* A combination book, video series, and tutorial
* Delivered as an iPython Notebook created in Atlas

# Authoring a notebook in Atlas

<img src="images/jem-docker-atlas.png"/>

# A Dockerfile for Just Enough Math

```
FROM odewahn/atlas-base
MAINTAINER Andrew Odewahn "odewahn@oreilly.com" 

#
# Install systemwide requirements
#

RUN apt-get install -y libatlas-base-dev 
RUN apt-get install -y gfortran 
RUN apt-get install -y gcc-multilib
RUN apt-get install -y lynx 
RUN apt-get install -y emacs23-nox 
RUN apt-get install -y glpk 
RUN apt-get install -y python-glpk


#
# Install python packages using pip
#
RUN pip install scipy
RUN pip install neurolab
RUN pip install hyperloglog                                      
RUN pip install countminsketch
RUN pip install pybloom               
RUN pip install lshash


#
# Install content using atlas-api to build the project
# Be sure to set ATLAS_KEY as an environment variable!
#   export ATLAS_KEY=<your atlas API key>
#
USER atlas
WORKDIR /home/atlas
RUN atlas2ipynb $ATLAS_KEY odewahn/jem-docker

```

# DEMO


* Start boot2docker and ssh into the box
* Pull [odewahn/python-data-analysis](https://registry.hub.docker.com/u/odewahn/python-data-analysis/).  (*NB: This is a big image -- 3GB+*)

```
sudo docker pull odewahn/jem-tutorial
```

* Start the container, and be sure to expose port 8888

```
sudo docker run -i -t -p 8888:8888 odewahn/jem-tutorial /bin/bash
```

* Once the container starts and your at the bash prompt, start the server with this command:

```
./start.sh
```

* Go to localhost:8888 on your local browser


# This experience leaves a lot to be desired

* One of the first projects was "Kids Code," which teaches kids about Python
* "OK kids, let's fire up an Ubuntu Virtual Machine and do some coding!" doesn't work well
* Even for pros, this is a bit intimidating
  * VMs and Vagrant are unfamiliar
  * Windows does not include an SSH client...

# Towards a more seamless experience

<video autoplay="true" loop="true" muted="true" width="720">
   <source src="https://s3.amazonaws.com/orm-atlas-media/pyxie-poc-video.mp4" type="video/mp4"> Your browser does not support the video tag.</video>

* [O'Reilly Pyxie](www.pyxie.io) is a place where authors can put Docker images for distribution
* Super-duper pre-alpha proof of concept
* Inspired by Nick Stinemates [any-sass](http://github.com/keeb/any-saas) project


# DEMO -- Just enough Math

* Go to [www.pyxie.io](http://www.pyxie.io)
* Log in
* Launch "Just Enough Math"
* Boom

# Lots of caveats

* Scalability is a *HUGE* issue
* Exploring many solutions for hosting images
  * [Docker on Mesos](https://github.com/mesosphere/mesos-docker)
  * [DEIS](http://deis.io/)
  * [Kubernetes](https://github.com/GoogleCloudPlatform/kubernetes)
* Security issues in running untrusted code
* Persistence and state
* Skills -- finding people who are familiar with these tools is challenging


# For more Info

* This presentation is at [http://sites.oreilly.com/odewahn/docker-at-oreilly/ch01.html](http://sites.oreilly.com/odewahn/docker-at-oreilly/ch01.html)
* The source is at [https://github.com/odewahn/docker-at-oreilly](https://github.com/odewahn/docker-at-oreilly)
* Email me directly at odewahn@oreilly.com
  * While you're there, check out my [Distributed Development Field Guide](http://sites.oreilly.com/odewahn/dds-field-guide/)

# Questions / Comments

<img src="images/questions.png"/>
