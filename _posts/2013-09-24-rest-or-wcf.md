---
layout: post
title: REST or WCF? Have your say!
tags: rest wcf api
---
When we first started working on BrightstarDB, we had to choose between REST and WCF server technologies. In the end we went with WCF because at the time it was easier to build both IIS-hosted and self-hosted WCF applications (at least that is the reason that I remember!). Since then it has become increasingly clear to me that this was probably the wrong decision because:

   1. WCF makes it harder to write client bindings in different languages. A simple REST interface using HTTP can be accessed by Ruby or Python or anything else.
   2. The Portable Class Library doesn’t ship with WCF client support for all of its different targets, so our Portable Class Library implementation suffers because it can only support an embedded database connection
   3. The particular WCF bindings we ended up using are not supported on Mono either.

(3) is a real deal breaker as I want us to be able to support running a BrightstarDB server under Mono. It would be possible to change to using difference WCF bindings that are supported by Mono, but this would break compatibility with older versions of the server/client protocol and so would be as big a change as simply dropping WCF and changing over to REST.

So, the question for users (or potential users) of BrightstarDB is: Would it be a problem for you if BrightstarDB was changed to use *only* REST as its client/server protocol. In practice this would mean that (a) client connection strings would change in your application and (b) versions of your applications running older versions of BrightstarDB would not be able to connect to a newer BrightstarDB server. I’ve written a bit more about the impact on a new thread in the BrightstarDB Users Google Group, and I would appreciate comments from anyone over on that thread.

If I don’t get a load of negative feedback on this, then I expect that the BrightstarDB 1.5 release will implement the move from WCF to REST.

UPDATE
------

This question is now resolved in favour of using a RESTful API. Interestingly, we received *no* requests for WCF to continue to be supported!