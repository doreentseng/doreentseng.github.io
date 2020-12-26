---
layout: post
title:  "Error when Creating A New App with create-react-native-app"
date:   2017-09-07
excerpt: "How to fix the error."
category: create-react-native-app
tag:
- create-react-native-app
- ReactNative
- npm
- yarn
comments: true
---

## Description

It drove me crazy cause I spent the whole day to "start" this app using `create-react-native-app`.  
At last I found the solution. If you are in a familiar situation like me, have a try and good luck!

### How I installed it
I did as this page <https://facebook.github.io/react-native/docs/getting-started.html>

~~~ shell_session
npm install -g create-react-native-app

create-react-native-app DoreensProject

cd DoreensProject
npm start
~~~

### Result I got

~~~ shell_session
> DoreensProject@0.1.0 start /Users/apple/GithubTree/DoreensProject
> react-native-scripts start

11:32:38: Unable to start server
See https://git.io/v5vcn for more information, either install watchman or run the following snippet:
  sudo sysctl -w kern.maxfiles=5242880
  sudo sysctl -w kern.maxfilesperproc=524288
        

npm ERR! Darwin 14.5.0
npm ERR! argv "/usr/local/bin/node" "/usr/local/bin/npm" "start"
npm ERR! node v6.11.2
npm ERR! npm  v3.10.10
npm ERR! code ELIFECYCLE
npm ERR! DoreensProject@0.1.0 start: `react-native-scripts start`
npm ERR! Exit status 1
npm ERR! 
npm ERR! Failed at the DoreensProject@0.1.0 start script 'react-native-scripts start'.
npm ERR! Make sure you have the latest version of node.js and npm installed.
npm ERR! If you do, this is most likely a problem with the DoreensProject package,
npm ERR! not with npm itself.
npm ERR! Tell the author that this fails on your system:
npm ERR!     react-native-scripts start
npm ERR! You can get information on how to open an issue for this project with:
npm ERR!     npm bugs DoreensProject
npm ERR! Or if that isn't available, you can get their info via:
npm ERR!     npm owner ls DoreensProject
npm ERR! There is likely additional logging output above.

npm ERR! Please include the following file with any support request:
npm ERR!     /Users/apple/GithubTree/DoreensProject/npm-debug.log
~~~

## Environment

* node&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6.11.2
* npm&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.10.10
* react-native&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.0.1
* react-native-cli&nbsp;&nbsp;&nbsp;&nbsp;0.47.2

## Solution
It seems that there's something wrong with this npm's version.  
So I use `yarn` instead of `npm` then... everything goes fine!
~~~ shell_session
brew install yarn
npm uninstall -g create-react-native-app
yarn global add create-react-native-app

create-react-native-app DoreensProject

cd DoreensProject
yarn start
~~~
The version of `yarn`
* yarn&nbsp;&nbsp;&nbsp;&nbsp;0.27.5

## Reference
- <https://github.com/react-community/create-react-native-app/issues/236>

