---
layout: post
title:  "Websockets And Action Cable In Rails 5"
date:   2016-03-24
categories: web sockets rails
---
As you may know, Rails 5 is here, and has an exciting sidekick! Ladies and gentlemen, let's put our hands together and welcome **Action Cable**, the novelty framework that integrates **WebSocket** communication with Rails.

Today I want to share some thoughts about this framework, kicking off with WebSockets, moving on to Action Cable, and then wrap my head around the problems that WebSockets are the answer to and the array of solutions one may put in place.

The grand finale will be on the structure and dynamics of our hero of the day - Action Cable.

## WebSockets

**What are WebSockets?** If you try to find that answer on the internet, you may get statements like:
\
\
_"WebSockets are those cool things the Node people get to use"_
\
\
or
\
\
_"I heard that WebSockets are the future"_
\
\
![Shia Labeouf SNL]({{ site.baseurl }}/images/websockets-rails-5/gifmachine-2.gif)

**WebSockets are basically another layer of communication between client and server but, unlike HTTP requests, their connections are _stateful_.** This means that the link between client and server remains constant and connected.

**WebSocket connections enable simultaneous and bidirectional communication**, allowing both client and server to send messages at any time through the channel.

Another advantage of using WebSockets is **once you establish the connection you don’t need to exchange much metadata** (http headers). Since there is one WebSocket per client **you don’t need to identify yourself every time you send data to the server**, because the server already knows who you are as it knows who opened the channel.

The final (and obvious) advantage is that **you can have truly real-time features**, like chat rooms or notifications, **without the need of, for example, using polling, that adds loads to the servers.**

On the other hand, the big advantage of using HTTP instead of WebSockets is that there is a lot of stuff already implemented for http, such as caching, routing, and multiplexing, that still isn't _WebSockets ready_.

## Action Cable

**So what is Action Cable?** In its own Github page:

> _Action Cable seamlessly integrates WebSockets with the rest of your Rails application. It allows for real-time features to be written in Ruby in the same style and form as the rest of your Rails application, while still being scalable and maintaining performance. It's a full-stack offering that provides both a client-side JavaScript framework and a server-side Ruby framework. You have access to your full domain model written with Active Record or your ORM of choice._

It sounds awesome. **Action Cable is a powerful addition to Rails, providing a clean interface for both client-side Javascript code as well as server-side Rails code.** One thing that I found particularly interesting is Rails 5 comes with Redis by default because Action Cable uses it as a pub-sub service for the channel's subscribers.

## Why we need WebSockets

This is all very interesting but we, as a company, use technology to solve our clients’ problems. **So why do we need WebSockets and Action Cable?** I found three main reasons.

The first one is when one needs to **send and receive data rapidly with the server**, for example, in online browser-based games that need to exchange several messages per second with the server, or information about the stock market, that is constantly changing.

Another is **streaming**. WebSockets are a good option, although I think Rails is not commonly used for streaming applications. This may change in the near future with the help of WebSockets and Action Cable.

And last, but not least, **live elements**, such as **comment sections** that are automatically updated when new content is added without a page refresh and, of course, chat rooms. Here we want the page to update when the data changes on the server without the user's intervention, making the most out of Action Cable's capabilities.

### But there are alternatives to WebSockets.

**Polling** is a popular solution!

**Polling makes the client ask the server periodically if there is any new data.** The advantage? It's rock solid and very simple to set up. The main disadvantage is adding load to the servers. HTTP caching is very good to alleviate that load.

**Polling is ok for things like comments sections, but not for rapid communication needs.** One example is Basecamp chat app, that used 3-second polling for 10 years, but with Basecamp 3 moved on to Action Cable.

**Other "not-so-popular" options are Long-Polling and Server-Sent Events.**

In **Long Polling** the client sends a request to the server for new data and, if the server has new data, it sends a response back like a normal HTTP request; if not, it holds the request and completes the response when new data appears.

**This quickly falls apart if data changes often.** Also, long-polling techniques are way more complicated to implement than polling.

**Server-sent events** are one-way connections from server to client. These, not only lack proper support, but also do not allow the client to send data back to the server.

## How Action Cable works

Just to get the feeling of Action Cable in action I built a simple chat app. It's login based, has one room to exchange messages, and you can see the users that are online. I'm now going to talk about the main parts related to Action Cable.

### Server side structure:

When you create your Rails 5 application you have now one more folder inside the app directory, the **channels** folder.

![Channels Folder]({{ site.baseurl }}/images/websockets-rails-5/channels-folder.png)

Under the folder channels you have the **application_cable** folder that contains the **channel.rb** file and the **connection.rb** file.

The **connection.rb** file inherits form **ActionCable::Connection::Base** and it's used for general authentication. We can use this module to query the database for a specific user that is making the connection and ensure that the user is allowed to listen to the channel.

The **channel.rb** inherits from **ActionCable::Channel::Base** and it's similar to the ApplicationController in our normal Rails Application.

![appearance_channel.rb and room_channel.rb]({{ site.baseurl }}/images/websockets-rails-5/appearance-and-room-channel.png)

Then, we have two channels under the folder, the **appearance_channel.rb** and the **room_channel.rb**. Those were the channels that I created. The channels have the same subscribed and unsubscribed methods:

* The **subscribed** callback is invoked when a client-side subscription is initiated;
* The **unsubscribed** callback is invoked when a client-side subscription is terminated.

**AppearanceChannel** has also two more methods that can be invoked by the client. **RoomChannel** has one more method, the **speak** method, that is also used by the client to send messages to the channel.

![AppearanceChannel]({{ site.baseurl }}/images/websockets-rails-5/appearance-channel.png)

![Client Side Structure]({{ site.baseurl }}/images/websockets-rails-5/client-side-structure.png)

From the client-side we have the **cable.coffee** file that is responsible to create the connection between client and server.

![Channel's Subscribers]({{ site.baseurl }}/images/websockets-rails-5/channels-subscribers.png)

Then we have the channel's subscribers. In this case we have the appearance subscriber that joined the appearance channel, and we have the room channel subscriber that joined the room channel.

![Channel Subscription]({{ site.baseurl }}/images/websockets-rails-5/channel-subscription.png)

We can see in the image above how the channel subscription is made.  
The subscribers have some methods in common:

* The **connected** method is called when the subscription is ready for use on the server;
* The **disconnected** method is called when the subscription has been terminated by the server;
* And finally the **received** method that is called when the server sends data to the client.

Then each channel can implement different methods that interact with the channel they subscribe on the server-side. That is the case with the **speak**, **appear** and **away** methods that can send data and call the corresponding methods on the server-side.

## In a nutshell

**Rails is alive and is bringing us more (and better) frameworks to help create products more efficiently.**

Action Cable is one of those frameworks and I truly believe it can be very useful to build live elements that are so popular in nowadays web applications.

I'm also curious about the gems that can build on top of Action Cable, and looking forward to how the community will embrace this new tool.
