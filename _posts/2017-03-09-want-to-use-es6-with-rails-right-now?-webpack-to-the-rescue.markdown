---
layout: post
title:  "Want to use ES6 with Rails right now? Webpack to the rescue"
date:   2017-03-09
categories: rails4 es6
---
Today I’m going to talk about setting up a Rails application so you can write complex JavaScript features using ECMAScript 6, also known as ECMAScript 2015, and its marvellous new features.

Nowadays we can do everything with JavaScript from the backend to the frontend, but for most of the challenges that I face daily as a Software Developer, I use Ruby on Rails since it’s a better fit for my needs, and also because I really enjoy building web applications using it. But when it comes to building more complex frontend features, JavaScript is the way to go, and that’s ok. JavaScript is a great programming language and over the past few years has improved immensely, particularly with the new ES6, and also with the arrival of new packaging and compilation tools, and also new frameworks and libraries.

Recently Rails 5.1.0.beta1 was released and I’m really happy with the new developments. Rails is embracing npm and Webpack, and as such we will be able to manage JavaScript dependencies with npm via Yarn and be able to compile JavaScript with Webpack with the new Webpacker gem.

Those new developments are still in beta and today most of us are still working in production applications that are still in Rails 4 (or less) and the upgrade might not be an option for now.

So in this article I will explore one way (of many) to configure a Rails application so we can write ES6 code, and include it in our applications without much effort.

## Problem

While developing a Rails 4 application at some point I needed to incorporate some features that required a lot of JavaScript code. When I started the development of those features I soon felt the need to have classes and modules to better organize my code, so ES6 immediately came to my mind. Aside from those needs, writing code in ES6 felt more future proof since sooner or later Rails would embrace it (it should happen in 5.1.0).

Summing up my problem, I wanted to write ES6 code in a Rails 4 application, and maintain the normal behaviour of my asset pipeline without messing up with sprockets.

## Solution

The solution I ended up with consisted in creating a separate folder called app-js in the rails project root directory that contains all the ES6 code and compiling it to browser compliant ES5 code that then would be place in the asset pipeline.

To get the ES6 code compiled and placed in the asset pipeline I used Webpack with Babel and I used npm to manage JS packages. In the end I got two configuration files package.json and webpack.config.js.

There are many more workarounds to get ES6 in Rails environment. Here I’ll explore one alternative that satisfied my needs.

The project directory ended up looking like this:

![project directory]({{ site.baseurl }}/media/rails4-es6/1-final-directory.png)

In the following sections I will explain this setup in more detail, while I walk you through the needed steps.

## Set up NodeJS and npm

To get started we have to install a few tools. We need to install NodeJS and npm as we will use those to install and manage the packages we need for our app. Assuming you’re using brew, to install them on a mac you can do it with:

```bash
brew install node
```

That’s it! With Node and npm installed head over to the root directory where your Rails project lives and run the following command:

```bash
npm init
```

Go through all the steps by filling out the information you think is appropriate and in the end a package.jsonfile will be generated. Don’t worry too much, you can always change it later. This file will allow you to manage the node modules your JS project depends on.

Now we need a compiler for our ES6 code. This is where Babel comes in. Babel is a JavaScript compiler that helps you write the latest version of JavaScript by compiling it to ES5 code, that today’s browsers can understand. We are going to install the babel-loader and babel-core packages that will be used to work with Webpack, as well as the ES2015 presets to load the code we will write.

```bash
npm install --save-dev babel-loader
npm install --save-dev babel-core
npm install --save-dev babel-preset-es2015
npm install --save-dev babel-polyfill
```

The sample app that I’m going to build to demonstrate this setup uses also [konva.js](https://konvajs.github.io/), a two dimensional canvas library. To be able to import it and use it in our code we have to install it:

```bash
npm install konva --save
```

From this point on you can install as many packages as you need. In this sample app we will only need konva. In the end of the process you should have a package.json file that looks something like this:

{% gist 4decc78c970cc82b62ddfe4d8785c795 %}

Since our first configuration file, package.json is finished, we will now create and configure the second configuration file we need, webpack.config.js.

## Set up Webpack

Webpack is a module bundler that takes assets like JavaScript files with many dependencies and turns them into something that you can provide to a client web page. To do that, it uses loaders that are defined in a configuration file, to know how to compile the assets. In our case, we want to compile ES6 code to ES5 code compatible with the current browsers. We will do this by providing a JavaScript file as an entry point and then Webpack will analyse it and get all the dependencies used in our code, to generate a compiled file that we can include in the asset pipeline of the Rails project.

We have already installed all the packages we need in the previous step. The remaining task we have to perform is to create the Webpack configuration file itself. Go to the project root directory and create a new file named wepack.config.js defined as follows:

{% gist ca0d510a65b28214256ce28d9aa378d1 %}

We can identify three main areas in the configuration file, the entry point, output file and the module loaders.

We start by defining the path for the entry point. This is the main file in our JS application where all the subsequent dependencies will be loaded as they appear in the code. The output defines the name and path of the compiled file. After the compilation we will have only one file that can be added to the Rails asset pipeline and be treated like a normal JavaScript file. Finally we have the module loaders where we specify the JS project directory the babel-loader and the presets. In our case we have ES2015 as the only preset.

With Webpack configured you are ready to go. Now all you have to do to get your code compiled and in the assets pipeline is to run:

```bash
webpack
```

The output of that command should be something like this:

```bash
Hash: a591cde9072bb3bd1422
Version: webpack 1.14.0
Time: 2721ms
    Asset    Size  Chunks             Chunk Names
app-js.js  877 kB       0  [emitted]  main
   [0] multi main 40 bytes {0} [built]
    + 305 hidden modules
```

A file with all your JS code (in the app-js folder) will be generated and placed in the asset pipeline. One advantage of compiling your JS I found useful is that it automatically detects syntax errors, so you don’t have to load the JS and check the errors in the browser web page.

While developing your JS features, to avoid running Webpack every time you change something you can use the following command that will compile the code every time it detects any change:

```bash
webpack --watch
```

To get a better feeling on this, let’s take a look at a sample application where you can see this setup.

## Sample Application

In the root app directory we will see four new things. We have two new folders, the app-js directory where we will put our JS code and the node-modules directory where the packages managed by npm will be placed. We have also the two configuration files, package.json and webpack.config.js.

![webpack config]({{ site.baseurl }}/media/rails4-es6/2-package-webpack.png)

Inside the app-js directory we can write an entire JS project that uses all the new ES6 features. It will be compiled into one file and placed in the asset pipeline. The code I have here is just one small example. I took inspiration in the [konva.js](https://konvajs.github.io/) demo and created a few JavaScript classes.

![javascript classes]({{ site.baseurl }}/media/rails4-es6/3-app-js.png)

The entry end point is the main.js file. This is the main file of our application where the code will start to execute. As you can see in this file we are using modules by importing classes that we depend on.

{% gist 15f7dcdcea47a880b9af9f677e0a139b %}

Next we can see an example of two classes of this app where I use some ES6 features like classes and inheritance, modules, const and let.

{% gist 297642ebadfdd23b63523e2194c92825 %}

If you need it, you can also create a folder name like ‘server-actions’ where you can define Ajax requests for controller actions. You can later bind these actions to some button in the interface so you can for example persist the data in the database.

This code is an example of what you can do with this configuration. I my case I wanted to use ES6 with classes and modules so I could build and grow my application in more structured way, since I was building a quite complex JS feature.

I also recommend that you add to gitignore the node-modules directory and the compiled file. Finally to get the file compiled in Heroku when you deploy it you need a post build instruction in the package.json configuration file to compile you JS assets and to do that your Heroku application setup should include the [NodeJS builpack](https://elements.heroku.com/buildpacks/heroku/heroku-buildpack-nodejs).

## Conclusion

Webpack, npm and ES6 are here to stay, and so is Rails. The JavaScript ecosystem always brings us something new, which is great, it means the community is working really hard to develop great tools that help us build better web applications. But sometimes I feel a bit lost on what tools I should use, since we have so many, and new ones are arriving all the time. On the other hand, Ruby on Rails is already 11 years old and is now a mature framework and when things settle down a bit in the JS world they eventually come to the Rails framework. Rails 5.1.0 is the proof of that, since it will include Yarn and Webpack.

In the meanwhile even though Rails 5.1.0 is in beta you can still write ES6 and include it in your application. All you need is two small configuration files like the ones I mentioned. Later when Rails 5.1.0 is released, and your app is ready for the upgrade, you can reuse all the code you wrote. I’m really happy with the new developments in Rails and looking forward to see what the developers and the community will do with them.

The code for the sample app can be found here: [https://github.com/mariorcardoso/rails_webpack_npm_es6](https://github.com/mariorcardoso/rails_webpack_npm_es6)
You can see the app live here: [https://rails-webpack-npm-es6.herokuapp.com/](https://rails-webpack-npm-es6.herokuapp.com/)
