---
layout: post
title: EventFeed Data Model
tags:
---
As we mentioned in a recent post we are going to be using BrightstarDB to create ready-to-go data models that can be used by developers in building their applications. First up is a data model that captures events, subscribers and topics to provide a core data model for any event registration and timeline system. All the code is available for use in the BrightstarDB github repository.

One requirement we often see for intranet based projects is the notion of an event stream that is personalised for each user. The concept is quite simple, subscribers subscribe to topics. Events occur at a given time and these events are classified by one or more topics. Events can come from many systems, (we don’t go into detail about how these events are captured). A subscriber timeline is a list of all events that are classified by the same topic(s) that a subscriber is interested in. More specifically, the subscriber will only see events that occured when they were interested in a given topic. Finally, events have associated data. This data can be different for each event. By using BrightstarDB we can combine a typed model for (Event, Subscriber and Topic) and also use dynamic features to capture the event data.

BrightstarDB was a good choice for this model as it allowed us to quickly iterate and try out new model ideas, and make use of the schema-less features of the underlying triple store.

We have defined a EventFeed service object that provides high level access to the functionality. It has the following operations:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
    public interface IEventFeedService
    {
        void AssertSubscriber(string userName, IEnumerable<Uri> topicsOfInterest);
 
        IEnumerable<IEvent> GetSubscriberTimeline(string userName, DateTime since);
        
        IEnumerable<IEvent> GetTopicTimeline(string topicId, DateTime since);
        
        void AssertTopic(Uri topicId, string label, string description);
        
        dynamic GetEventData(IEvent feedEvent);
 
        void RaiseEvent(string description, DateTime when, IEnumerable<string> topicIds,
                        Dictionary<string, object> eventProperties = null);
 
        void RegisterInterest(string userName, string topicId);
 
        void RemoveInterest(string userName, string topicId);
    }
view rawIEventFeedService.cs hosted with ❤ by GitHub
The data model consists of just three types; IEvent, ITopic and ISubscriber, and is shown in the diagram below.



We hope this model is useful to someone, either as is, or as a starting point for something bigger and better. Also, if you are looking at how dynamic and typed objects can live together then check out the service method for RaiseEvent.

All feedback welcome to contact(at)brightstardb.com, or @brightstardb