---
layout: post
title: "Puppet, Gatling, And Jenkins Together"
modified: 2013-09-02
tags: [puppet]
image:
  feature: 
  credit:
  creditlink:
comments: true
share:
---

Puppet Gatling is a Jenkins-CI plugin that post-processes Gatling simulation data to generate useful reports for load-testing Puppet. Using this tool, users are able to discover a clear difference in performance between various versions of open source Puppet and Puppet Enterprise.

#### Quick Note

This is a cross post from the [Puppet Labs Blog](http://puppetlabs.com/blog/puppet-gatling-and-jenkins-together). I originally intended it to be the first post on here, but was given an opportunity to post on the official blog and took it.

# Load Testing

The concept of load testing is pretty simple. Mostly it involves coming up with a few different ways to stress-test the code to understand how robust a given project is. This becomes very important when trying to show customers improvements

One of my major projects as a summer intern at Puppet Labs was to find a simple way to show developers, marketing and customers that each version of open source Puppet and Puppet Enterprise improves on the prior version.

How did we accomplish this? Well there are a few main tools that we used, along with a little plugin development.

# Jenkins CI

![jenkins](/images/jenkins_logo.png)

Jenkins CI (or just Jenkins) is an open source application that helps you execute repeated jobs like running different unit or acceptance tests, building packages, or other load testing. This style of development is known as Continuous Integration. Organizations will have either public or private Jenkins instances, or sometimes both. Some public examples include [Apache](https://builds.apache.org/), [jruby](http://ci.jruby.org/), and even [open source Puppet](https://jenkins.puppetlabs.com/)! Using Jenkins for continuous integration helps improve quality assurance and gives developers a way to keep a continuous check on the state of their application through development.

Jenkins also provides a huge variety of [plugins](https://wiki.jenkins-ci.org/display/JENKINS/Plugins) to choose from to improve your experience. Because Jenkins is open source, it is pretty easy to write [your own plugin](https://wiki.jenkins-ci.org/display/JENKINS/Plugin+tutorial) too!

# Gatling Project

![gatling](/images/gatling_logo.png)

The [Gatling Project](http://gatling-tool.org/) has provided a [plugin](https://wiki.jenkins-ci.org/display/JENKINS/Gatling+Plugin) that you can hook into Jenkins which keeps track of different simulation information and a detailed view of each build that runs on Jenkins. By simply adding the plugin to a Jenkins job, it will display a few different trend graphs across builds including:

- Mean response time trend
- 95th percentiles response time trend
- Percentage of failed requests 

These graphs provide some great information about the job, however understandably so do not account for how Puppet runs work between a master and its agents. This means that the data is calculated incorrectly. Thus the need for a plugin extension, Puppet-Gatling.

## Implementation of Puppet-Gatling

The main tools used in this process beyond Jenkins and Gatling include Maven with Java for plugin development. Maven proves to be a useful tool while developing Jenkins plugins for the simple reason that when you are ready to test, running the plugin through Maven will generate a new Jenkins instance. Without this feature, you would have to package your plugin every time and deploy it on a live Jenkins server.

[<img src="/images/jenkins/puppet_gatling.png" alt="web console" style="width: 800px; height: 700px;"/>](/images/jenkins/puppet_gatling.png)

The [src](https://github.com/puppetlabs/puppet-gatling/tree/master/src/main) folder in Puppet-Gatling contains three main folders: java, resources, and webapp. The java folder is what you would expect: This is where all the java code for the project lives. The resources folder is for all of the view/GUI files that manipulate the Jenkins web console. These files are known as .jelly files. The webapp directory contains the plugin icon that will be displayed on a given Jenkins build.

The plugin itself is broken down into a few main classes that are important to Jenkins:

### PuppetGatlingBuildAction.java

This is what controls the code behind specific build pages on the web console. This page can be reached by clicking on some of the builds on the left-hand side of the main page for a job, as seen below. When visiting the specific build page for a given run, you will find some OS data that the project gatling-puppet-load-test gathered during the simulation (discussed below under the heading Additional Open Source Contributions), along with a breakdown of the data calculated by Puppet-Gatling. All the data you see on the build page is generated initially through the index.jelly file in the resources folder. Some of the code in index.jelly calls the related java file here as well - that is, to obtain some of the data collected and stored within the plugin. Think of jelly files like View in an MVC framework.

[<img src="/images/jenkins/builds.png" alt="builds" style="border: 1px solid grey;"/>](/images/jenkins/builds.png)

### PuppetGatlingProjectAction.java

This is what controls the plugin main page for each Jenkins job. The icon on the left of the job dashboard provides a link to the project page. This page shows all the different trend graphs that Puppet-Gatling calculates for each run. This page also provides links to builds at the bottom, although you can also click on any data point within the graph to load the build page, too. This data point feature was added with my plugin, and was later merged into the official plugin.

[<img src="/images/jenkins/project_view.png" alt="web console" style="width: 800px; height: 700px;"/>](/images/jenkins/project_view.png)

### PuppetGatlingPublisher.java

This is the main brains behind the project. The publisher part of that file name is important. For plugins, publisher tells Jenkins that it should be a post build action. Simply, this means that the plugin should run after everything else in the job has executed. This is important because if we run the plugin too soon, the Gatling data will not have been generated yet. This file is in charge of gathering all the data generated by gatling-puppet-load-test and the Gatling plugin, then crunching the numbers and providing the user with a pretty trend graph that has data related to Puppet runs.

[<img src="/images/jenkins/postBuild.png" alt="web console" style="border: 1px solid grey;"/>](/images/jenkins/postBuild.png)

Other java files in the project include: a class that stores the Simulation configuration; some data about the simulation; a report about the data collected from Facter for the simulation; and some other classes that help run the display of different trend graphs.

## Testing Setup

A simple example job configuration is shown below. It will install Puppet Enterprise 2.8, and run two simulations. Each simulation has a unique id, and each simulation can have a different number of nodes to test against, along with the number of times it executes the node config. This style of configruation allows for ease in configuring a given Jenkins job. Before this, you had to go through a complicated process of setting env variables and adding a lot of shell scripts to execute. Thanks to the gatling-puppet-load-test project, you can now configure the job like this json below.

<script src="https://gist.github.com/briancain/6247039.js"></script>

Our end product will test the same simulation against two different versions of Puppet. This can be done by having a new simulation step where you add an install step before the simulation. It will then uninstall whichever version of Puppet is present, add the new installation, and go about the simulation.

# Gatling Puppet Load Test

This project is what allows for easy configuration in Jenkins. When the job configuration is in this json format, the gatling-puppet-load-test project is able to parse the simulation configuration and set up the Jenkins job for it to run with the given description. My part in this project was fairly small, and I contributed only a couple of features to the project itself. 

One important feature of the Puppet-Gatling plugin is the ability to display different information about the OS and the simulation to the user. For that to happen, there needed to be a step in the setup of gatling-puppet-load-test using [Facter](http://docs.puppetlabs.com/facter/1.6/core_facts.html) to gather data. This was all done in ruby. Once the data required was collected, the setup step would save that data to the Jenkins workspace for the plugin to use later.

Another small contribution I added was the ability to make comments in the json configuration. This is important when you don’t want to install Puppet Enterprise on every job, or if there’s any other simulation step that you want to leave out. Otherwise you would have to move it out of the json configuration and save it as a comment in the execute shell portion of the Jenkins job configuration. This can become messy and annoying, especially when there are a lot of steps that you don’t need more than once.

# Additional Open Source Contributions

As mentioned above, a feature of the Puppet-Gatling plugin was merged into the working source for the official Gatling Jenkins plugin. The main idea behind it was the ability to click on some of the graph data points and be taken directly to a build screen that Gatling generates. Without this feature, you had to navigate to the bottom and look through all of the builds in list order. This becomes a problem when you are trying to remember the specific build number and simulation id that you want to look at from the graph. Now you don’t have to: Just click on the data point, and it will open a new tab and take you there!

# Final product

The plugin now lives on [Github](https://github.com/puppetlabs/puppet-gatling) and will be maintained by Puppet Labs. However, it’s open source! So pull requests and contributions are welcome.

tags: [puppet](/puppetlabs)
