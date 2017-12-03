---
title: "Installing node-red on Bash on Ubuntu on Windows"
excerpt_separator: "<!--more-->"
categories:
  - "Bash on Ubuntu on Windows"
tags:
  - node-red bash windows
---
A simple how-to showing the process of installing node-red on Bash for Ubuntu on Windows

<!--more-->

First you need node-js and npm

```
$ sudo apt-get install nodejs
$ sudo apt-get install npm
```

Wait for the entire Internet to download. Then install node-red as per their instructions

```
$ sudo npm install -g --unsafe-perm node-red
```

Try to run it and you will be greeted with an error

```
$ node-red
/usr/bin/env: ‘node’: No such file or directory
```

So lets fix it. First lets locate nodejs

```
$ whereis nodejs
nodejs: /usr/bin/nodejs /usr/lib/nodejs /usr/include/nodejs /usr/share/nodejs /usr/share/man/man1/nodejs.1.gz

```

Next, lets add a symbolic link from node to nodejs:

```
$ sudo ln -s /usr/bin/nodejs /usr/bin/node
```

Now let us retry node-red

```
$ node-red
21 Sep 06:25:00 - [info]

Welcome to Node-RED
===================

21 Sep 09:25:00 - [info] Node-RED version: v0.17.5
21 Sep 09:25:00 - [info] Node.js  version: v4.2.6
21 Sep 09:25:00 - [info] Linux 4.4.0-43-Microsoft x64 LE
21 Sep 09:25:00 - [info] Loading palette nodes
21 Sep 09:25:01 - [warn] ------------------------------------------------------
21 Sep 09:25:01 - [warn] [rpi-gpio] Info : Ignoring Raspberry Pi specific node
21 Sep 09:25:01 - [warn] ------------------------------------------------------
21 Sep 09:25:01 - [info] Settings file  : /home/jaspers/.node-red/settings.js
21 Sep 09:25:01 - [info] User directory : /home/jaspers/.node-red
21 Sep 09:25:01 - [info] Flows file     : /home/jaspers/.node-red/flows_jaspers-5510.json
21 Sep 09:25:01 - [info] Creating new flow file
21 Sep 09:25:01 - [info] Starting flows
21 Sep 09:25:01 - [info] Started flows
21 Sep 09:25:01 - [info] Server now running at http://127.0.0.1:1880/
```

Voila. Hope this was helpful
