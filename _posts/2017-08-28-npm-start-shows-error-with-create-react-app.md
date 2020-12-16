---
layout: post
title:  "npm start Shows Error with create-react-app"
date:   2017-08-28
excerpt: "react-scripts start but command not found"
category: React
tag:
- create-react-app
- react
comments: true
---

## Description

I pull a project down which I haven't tested for two months. I remove the node_modules folder and try 
`npm install`.
But it shows me the error.

Error:
~~~ shell_session
> asus-chat@0.1.0 start /Users/apple/SourceTree/asus_chat
> react-scripts start

sh: react-scripts: command not found

npm ERR! Darwin 14.5.0
npm ERR! argv "/usr/local/bin/node" "/usr/local/bin/npm" "start"
npm ERR! node v6.11.2
npm ERR! npm  v3.10.10
npm ERR! file sh
npm ERR! code ELIFECYCLE
npm ERR! errno ENOENT
npm ERR! syscall spawn
npm ERR! asus-chat@0.1.0 start: `react-scripts start`
npm ERR! spawn ENOENT
npm ERR! 
npm ERR! Failed at the asus-chat@0.1.0 start script 'react-scripts start'.
npm ERR! Make sure you have the latest version of node.js and npm installed.
npm ERR! If you do, this is most likely a problem with the asus-chat package,
npm ERR! not with npm itself.
npm ERR! Tell the author that this fails on your system:
npm ERR!     react-scripts start
npm ERR! You can get information on how to open an issue for this project with:
npm ERR!     npm bugs asus-chat
npm ERR! Or if that isn't available, you can get their info via:
npm ERR!     npm owner ls asus-chat
npm ERR! There is likely additional logging output above.

npm ERR! Please include the following file with any support request:
npm ERR!     /Users/apple/SourceTree/asus_chat/npm-debug.log
~~~

## Environment

* node&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6.11.2
* npm&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.10.10
* react&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;15.5.4
* react-dom&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;15.5.4
* react-scripts&nbsp;&nbsp;&nbsp;&nbsp;1.0.3 

## Solution

According to the author of Create React App:  

1. We absolutely should not be installing react-scripts globally.<br />
2. And we also don't need ./node_modules/react-scripts/bin/ in package.json as this answer implies.<br />
Something went wrong when dependencies were installed the first time.<br />

The author suggest doing these three steps:

1. `npm install -g npm@latest` to update npm because it is sometimes buggy.<br />
2. `rm -rf node_modules` to remove the existing modules.<br />
3. `npm install` to re-install the project dependencies.<br />

## References

* [npm start error with create-react-app](<https://stackoverflow.com/questions/39959900/npm-start-error-with-create-react-app> "Go")