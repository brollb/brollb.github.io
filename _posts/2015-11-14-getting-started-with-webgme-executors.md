---
layout: post
title:  "Getting Started with WebGME executors"
date:   2015-11-14 15:06:00
categories: webgme
---

In the past, I have played with the example WebGME executor but I keep forgetting exactly how they work (so I figured posting about it should solve this problem!). For anyone unfamiliar with the WebGME, information about it can be found on the [home page](http://webgme.org) or you can just check out the source on [github](github.com/webgme/webgme). At the time of this writing, the latest WebGME is version 1.1.0.

First, what are _executors_ in the WebGME?

Executors, in the WebGME, allow a WebGME server to post jobs to remote machines which have access to system resources. This allows the server to delegate computation to remote hosts (rather than performing the computation itself).

<!-- TODO: Add a diagram or something -->

Before we create our own custom executor, we should probably get the tutorial (included in the WebGME) working. The steps to get it working are:

### Install basic dependencies
The WebGME is a NodeJS app so it is best if you have npm installed on your machine. I recommend using a version manager (such as [nvm](https://github.com/creationix/nvm)) so you can easily transition between node versions.

Also, the WebGME is software so you should probably use version control. In this case, we will be using git.

In this example, we will be working directly out of the WebGME project, so we will first clone it:

    git clone https://github.com/webgme/webgme

Then install the project dependencies:

    cd webgme
    npm install

### Install nw
The executor servers (using a graphical interface) use [nw](nwjs.io) so our first step is to install it:

    npm install -g nw

### Setting up the executor
Setting up the executor worker is pretty simple. We will be installing the dependencies, creating the config, then starting it with nw:


    cd <WEBGME-ROOT>/src/server/middleware/worker

    npm install

    cp config_example.json config.json

Now your executor is configured and has the dependencies installed! You can start the executor with:

    nw

The configuration is pretty simple; it just tells the executor what WebGME server is allowed it is using.

### Setting up the WebGME server
<!-- At the time of this writing, the [webgme-setup-tool](https://github.com/webgme/webgme-setup-tool) does not support executors but we will still use it to manage our base install. --> 
First, enable the executor in the WebGME server by opening `config/config.default.js` and setting

    config.executor.enable = true

Next, we will start the WebGME (like any other nodejs project):

    npm start

We will now enable the example plugin which uses an executor, `ExecutorPlugin`. This can be done by opening any project and, while viewing the ROOT node, add "ExecutorPlugin" to `validPlugins` (in the PropertyEditor in the lower right). Finally, open a non-ROOT node and run the plugin by clicking on the "play button" in the top left!

<!-- TODO: Add screenshots -->
