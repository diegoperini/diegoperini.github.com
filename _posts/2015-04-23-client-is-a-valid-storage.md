---
layout: post
title: "Client is a valid storage"
description: "A compact introduction to client side data storage"
category: 
tags: ["backend-security"]
---
{% include JB/setup %}
I've been developing backend web software for the last few years and during this small interval, it became a common task to implement database layers, user authentication, client side caching, tracking unauthenticated traffic and user metric collection. Each time a project included such requirement, learning it through hacking was always the best approach for me. My own computer science degree, Stack Overflow answers and blog posts of people from open source communities are my main sources here. In this post, I will try to use simple examples to explain where one can store application data in a client/server infrastructure and what implications may rise from using each storage type. At the end, there is a guideline of mine I obey when I use client side storage (web and mobile) for applications that communicate through web. Reach me if you believe some statements in this post are incorrect or bad advice.
 
####Ultimate Paranoia

I like to think that every client user is an expert hacker who can read and modify your client source code at will. Every configuration file and persistent data is stored in plain text files and none of that data is sand-boxed. In reality, mobile browsers and app storages are more securely handle their data but it is best to assume the worst. 

In other words, there are actually only two locations you deal as storage in terms of security. "Within reach" which means cookies (web), local storage (HTML5), session storage (HTML5), user defaults (iOS), shared preferences (Android), SQLite (iOS, Android) etc. and "beyond reach" which is your server and its databases, local files, SaaS components. 

####Classifying Data

We will semantically classify our data as "first class", "second class" and group them in different ways to represent "states" and "leftovers". I will use this RESTful sign-in response of an example social app to mark them. Eventually, we will distribute this data to our server/client storages without exposing a vulnerability in our app.

{% highlight javascript %}  
// A sign-in response
{
    "id": 69395395,
    "username": "john",
    "age": "19",
    "gender": "male",
    "access_token": "3e416ab380add736c1b_1000322",
    "server_time": "2015-04-23 02:25:00"
}
{% endhighlight %}

When the entity represented by the data is a mirror of your models attributes (a user's profile, a post, a media content), this definition will call it first class data. These entities are exchanged when the clients request latest version of the model or they order changes in them. The values inherited from first class data are significant to the user. In our social app, you retrieve such data to fill a profile screen, allow manipulations to it via predefined inputs and restricted API calls. Let's mark them.

{% highlight javascript %}  
// A sign-in response
{
    "id": 69395395,                                 // first class
    "username": "john",                             // first class
    "age": "19",                                    // first class
    "gender": "male",                               // first class
    "access_token": "3e416ab380add736c1b_1000322",
    "server_time": "2015-04-23 02:25:00"
}
{% endhighlight %}

Second class data exist only to identify the owner of its first class siblings. When alone, they are gibberish to the user. Once attached with additional first class data, they are used to determine targets of updates, source of retrievals or means of their owner entity.

Here they are.

{% highlight javascript %}  
// A sign-in response
{
    "id": 69395395,                                 // first and second class
    "username": "john",                             // first class (also a second class candidate if usernames are unique)
    "age": 19,                                      // first class
    "gender": "male",                               // first class
    "access_token": "3e416ab380add736c1b_1000322",  // second class
    "server_time": "2015-04-23 02:25:00"            // second class
}
{% endhighlight %}

It is possible for a data to be classified by more than one kind. John's id is both first and second class data as without it, John's model would not be complete.

####Gathering Data Together

A state is a collection of at least one second class data with zero or more first class data. We used second class data to keep secure the validity of some state. We can't trigger and get status updates for "john 19 male" without knowing which "john" we are dealing with. John cannot update his profile without proving that he really is "john#69395365". If it is almost new years eve in the country where our server is located (let's assume US), then John now can be 20 if he lives somewhere else (i.e Australia).

There can be other collections that only consist of first class data. We define such collections as leftovers since they lose their tie to the model without a second class sibling. 

Our effort is to store as much data we can on the relevant side while maintaining minimum number of state entries on our server. We will hand-pick some collections to introduce both security and better user experience. 

Easiest ones to pick are collections that cover their model entirely and their model only. 

{% highlight javascript %}  
// John's model (a state)
{
    "id": 69395395,         // first and second class
    "username": "john",     // first class
    "age": 19,              // first class
    "gender": "male"        // first class
}
{% endhighlight %}

This collection is all John is, whoever asks for John will need these values to identify him. Since nobody should wait for John to be online all the time, this collection needs to be stored on the server for real time retrieval.

When John filled input boxes to sign-in, he provided a way for our app to remember his name on the same device in the future. Let's pick a collection to make his next visit an easier one.

{% highlight javascript %}  
// Autocomplete username (a leftover)
{
    "username": "john"      // first class
}
{% endhighlight %}

We can safely store this collection in our client to automatically fill username field in our sign-in box.

John has a personal computer that only he can access, therefore wants to be kept signed-in by our app all the time. He is not comfortable with the fact that his password is remembered by his browser and we want to make sure he feels safe. We need a replacement for his password that will help us identify him on his each request. We call it an access token. 

One creates an access token by digitally signing an authentication request of a user. When John provided his username and password to sign in, he also sends additional info that is special to his device. The more additional data he sends, the harder it becomes for an attacker to mimic his device (even if his access token is exposed). We can use his IP, browser name, OS name, sign-in date, device id (if exists) to generate an access token which can expire after some time, on network change, on different browsers and on a different operating system. Once I finalize this post, I will add my answer on Stack Overflow as an appendix entry for an example token generation algorithm.

We choose John's access token alone as our collection and store it in our client.

{% highlight javascript %}  
// John's token (a state)
{
    "access_token": "3e416ab380add736c1b_1000322"       // second class
}
{% endhighlight %}

John is the only person who can modify his profile. In other words, as long he doesn't modify it, refreshing his profile page or accessing partial info from his profile shouldn't trigger an HTTP request. It will reduce network data consumption of his device and ease some burden on our server. Also his profile will be ready all the time. He may be using multiple clients at the same time so we may want to invalidate our cached profile after some time.

{% highlight javascript %}  
// John's profile (a state)
{

    "username": "john",                                 // first class
    "age": 19,                                          // first class
    "gender": "male",                                   // first class
    "server_time": "2015-04-23 02:25:00"                // second class
}
{% endhighlight %}

Here is a useful cache entry for our client. We stored server time to later decide if the cached data is too old.

####Security Guideline for Storages

We can implement many more features for John. The good thing is, as long as we are persistent with this methodology, it is always apparent where to put our collections. There is a pattern here. We are now ready for the guideline I mentioned before.

***On preparing first class data:***

1. Define all accepted and unaccepted values on server and implement proper input validation both on your server and client. Nobody can be -1 years old after all.

2. Some first class data never require a storage. Try to produce them on server by processing existing data first (i.e counting people in a friend list).

***On preparing second class data:***

1. Only generate them on server. Your generation algorithm must be hidden to your client. You can sign, encrypt and/or obfuscate them if necessary. (see appendix to learn how)

2. Validate them on every retrieval on your server (see appendix to learn how).

***On choosing collections:***

1. Choose the smallest state (a collection with one second class data) that can represent your model alone and store it on your server's most persistent storage (i.e a database).

2. For each feature that serves to your user, store a state (a collection with one second class data) on client.

3. Leftovers (collections without second class data) can be stored on client without hesitation. Never send them back to server though.

4. If your client state is identical to your server state, your client's second class data exposes a vulnerability. Replace your client state's second class data with a new one generated by using the one in the server as input. (see appendix to learn how)

5. A state can only be valid if all its second class data can be validated by your server.

####Appendix

1. [How to generate and validate an authentication token in PHP](https://stackoverflow.com/questions/29387710/ionic-ngcookies-php-authentication-how-secure-can-it-be/29411089#29411089)