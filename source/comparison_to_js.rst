Comparison to JS and Node For Max 
---------------------------------
The **js** (JavaScript) object in Max *only* executes in the low priority thread. This is done
to ensure there is no chance of a thread context switch corrupting data and creating unknown
conditions, as detailed above. The fact that the **js** object always defers messages to the low
priority thread has ramifications for what we can reliably do with the **js** object.
Realistically, we can never guarantee that an event handled in JavaScript code 
will be serviced in a timely manner. While there are delay operations in JS,
they only create a *minimum* delay. The event
won't be serviced until the delay is done, and the low priority thread gets to run again.
It's not at all uncommon for the low priority thread to hiccup: visual redrawing
operations or any file system interaction can do it. The upshot of this is that while the 
**js** object has quite a comprehensive set of features, it's not reliable for timing critical operations.
Note that this only matters for events that will be rendered to *audio* - user interface and Jitter operations
execute in the low priority thread anyway, making **js** suitable for UI or Jitter manipulation.

There is often some confusion about how Node For Max fits into all this. Node For Max uses
a completely different model, where the Node script executes in a separate *process* that is spawned 
and supervised by the Max process. Under the hood, Node scripts do not get
executed by Max system code, (the way the **js** and **s4m** objects do), and thus don't have programatic
access to all of Max's internals through the Max C API functions. The Node code is executed by a Node.JS
runtime, outside of Max, with communication to and from Max over a network connection. Messages 
and data are serialized and deserialized automatically so that we don't need to deal with 
any of the networking complexity. The advantage of this is that the Node script
can work with the external file system or communicate with the outside world over networks and
we never have to worry about how long that takes. When Node finishes its work, messages
come back to Max. However, this means that Node is, like *js*, not reliable for timing critical code, as we have
no guarantees of when Node code will complete and send messages back to Max.
This does, however, make Node For Max a good compliment to S4M. By adding a Node object 
to our patchers, we can use it for operations that might take a while and which we don't want 
blocking either a low or high priority **s4m** object. We can have an **s4m** object send messages to 
Node to request a long running operation (such as saving or loading a file), 
or to take advantage of the rich libraries for interacting
with the world outside of Max, and have the results sent back as messages to **s4m** or make 
them accessible in Max dictionaries that **s4m** can read.
 





