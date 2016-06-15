---
layout: post
title:  "Transaction brainstorming"
categories: notes
---

Transaction is caused by some user interaction with the page.
Lets categorize transactions into two groups(possibly a retarded idea):

1. Cause hard/soft[[1](#fn1)] navigation 
2. Don't cause navigation change        

## When to start transaction

### 1. Navigation

#### Hard navigation

As the whole page is reconstructed our analytics script is also reinitiated
and we can start the transaction on our script load.

#### Soft navigation

So we want to start the transaction when navigation starts but we don't know when it does.
 
Options/Ideas (K - know how to do, U - unknown, need to research ):

* K-BAD: For every major framework write something that listens to framework broadcast events and starts/ends transaction via our API. 
* U-DECENT: Instrument some History interface methods and listen to hashChange[[2]] event. 
    * In Chrome can at least instrument pushState
    * Have others done something similar? Why not?
    * Security issues, browser support, reliability etc.
* U-IFY: Most of the time if not always some DOM changes need to happen when navigation takes place, otherwise it seems pretty
     pointless to even do the change. Sometimes the new DOM is asked from the server but it can also be cached and the framework
     doesn't even do the request but takes it straight out of some internal cache eg. angular can use $templateCache[[4]] service.
     So we most likely cannot use XHRs for predicting the start and end. 
     It might be possible to listen to DOM changes via MutationObserver[[5]] and try to predict the start and end.
     * MO browser support is pretty shit are there alternatives?
     * Performance overhead?
        
### 2. No Navigation
        
Any user interaction that doesn't cause a navigation change.
Click, select, scroll, form submit etc. 
In a fairy tail solution the transaction starts when user does something that might be slow or fail and
it ends when user perceives that his/her action is completed.

In reality we have no way of knowing what exactly user wants. Button might be clicked
and 3 requests done, but user might only be interested in the results of the first request.
  
What we hope to do is to match the transaction start with some event start time and try to
understand what work actually happens due to that event. And if all/most of the work is done
the transaction ends.

What events are we interested in? In some cases button click is important 
and in some other case it might not be. I am guessing that for the moment we only care about 
events that cause a server request.     
   
Options/Ideas

* use our API to define in application code where transaction starts
    * This can't probably be the only way as no one will care enough to manage it.
    * It is probably still useful to some people so we should have it.                    
        
## When to end transaction

Ending transaction means that user interaction has ended. He/She has gotten the response from
the application or it might have also failed.

Is there a need to differentiate between single page application and mostly server rendered app?
I don't think there are many pages where nothing is loaded asynchronously on client side anymore.

The difference between nav and no nav and hard and soft is not so clear here. 
But it seems like it makes things a bit easier. 
 
### 1. Navigation

#### Hard Navigation

* K-VBAD DOM completed
    * Only accurate on mostly server rendered pages
* K-BAD All XHRs completed
    * HTTP polling, eg. our welcome page, can possibly detect this and work around
    * Some totally unimportant XHR might slow the whole transaction. How often is this the case?                           
* K-IFY                  
        
### 2. No Navigation


        
## Notes/explanations

### <a name="fn1"></a> Hard and Soft navigations

Hard navigation - normal navigation, whole page is reloaded. F5, link is clicked etc.

Soft navigation - URL changes but only part of the page is reloaded. Url is changed via History interface API[[3]] 
or # is used to keep track of changed url. In angular it is turned on via $locationProvider.html5Mode(true). 
Most frameworks have some kind of client side router plugin that handles the router changes.

{% highlight javascript %}
console.log('dank memes');
{% endhighlight %}


[2]: https://developer.mozilla.org/en/docs/Web/API/WindowEventHandlers/onhashchange
[3]: https://developer.mozilla.org/en-US/docs/Web/API/Window/history
[4]: https://docs.angularjs.org/api/ng/service/$templateCache
[5]: https://developer.mozilla.org/en/docs/Web/API/MutationObserver