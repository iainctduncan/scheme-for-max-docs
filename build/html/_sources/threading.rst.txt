Max Threads, the Scheduler and the S7 Garbage Collector 
=======================================================

One of S4M's distinguishing features is its support for execution in either the Max
high-priority scheduler or low priority UI thread, not possible with either the JS or Node For Max objects. This 
allows S4M to be used for precise timing and scheduling operations for sequencing, live coding,
altering real-time musical input, and for algorithmic composition.  
This page discusses how threading works within Max and S4M and why it matters,
and what you should know about Max Settings to get the best performance.

Max's scheduling model
----------------------
To understand scheduling and threading in Scheme For Max, we need to understand
how Max's scheduler and threading model works. In Max, there are normally between one and three threads running, 
depending on your configuration settings and what you have in your patch. These are the main thread, the scheduler thread,
and the audio thread. (There could also be additional threads if a Max object explictly creates
and manages them, but this is not relevant for our purposes, and can be safely ignored for now.) 

Main thread
^^^^^^^^^^^^^^^^^^^^^^^
The main thread is also called the UI thread, and runs with the lowest priority. When we initiate 
activity by using Max user interface objects (clicking bangs and messages, moving sliders, etc),
we trigger execution in this thread. This thread is also responsible for screen redrawing and
file i/o, and executes at a lower priority so that the screen can refresh without interupting
audio playback or delaying musical events. Generally speaking, operations that involve loading 
files, refreshing UI elements, or creating visuals (with Jitter for example) should be done in this thread.
In Max For Live, the Live API also runs in the low priority thread, similar to control surface
scripts in Live. 

Scheduler thread
^^^^^^^^^^^^^^^^^^^^^^^
The scheduler thread is also called the high-priority thread. This is where timing critical Max
messages should execute, and by default MIDI i/o and **metro** object messages execute in this thread, as well
as any events triggered from the **transport** object. We also
execute in this thread if we trigger Max event messages from audio signal processing, such as with the **~edge** 
object. This thread context only exists as an actual separate thread if you have enabled "Overdrive" in your settings.
Overdrive moves the scheduler thread into an Interupt Service Routine for high timing accuracy and more
frequent scheduling. The only reason not to enable Overdrive is if a patch is entirely aimed at visuals.
We can configure this thread by setting the 'Scheduler Interval' setting in Scheduler
preferences to control how frequently the thread runs (the default is 1ms) and the Poll Throttle setting
to set the maximum number of event messages processed on one scheduler thread pass. (Though 
as is explained shortly, these don't actually matter if you also have selected Audio in Interupt.) 

Audio thread
^^^^^^^^^^^^^^^^^^^^^^^
The audio thread is used to calculate MSP audio rendering. It only runs if audio processing is turned
on, and is the thread that executes all real-time audio processing. On each pass, it calculates the audio samples
necessary to fill up one signal vector of samples. Max is a buffered audio system, meaning that
each audio pass calculates a series of audio samples that are then sent out in a chunk. We set
this number of samples with the Signal Vector Size setting in the Audio Status window. This setting also
affects audio latency as we will always have a latency of at least the size of the Signal Vector multiplied
by the time for one sample at the current sample rate (before adding any sound card latency).
Note that in Max For Live, the signal vector size is locked at 64 samples.

Audio in Interrupt
^^^^^^^^^^^^^^^^^^^^^^^
If, in the Audio Status window, we have selected Audio Interrupt as well as Scheduler Overdrive, or we
are running in Max For Live, we will actually have only two threads running, because the 
audio thread and the scheduler thread are merged into one thread that alternates between
rendering a vector of audio samples and handling scheduler events.
This also means that events are processed once per signal vector size, with event onset times
quantized to this boundary.

This may or may not be better for your use case, depending on how big your audio vector size is
and how much audio CPU intensive processing your patch needs to do. For example, if you are running
heavy audio calculation and thus need a large signal vector size, you might see scheduler thread 
activity get delayed by the time for one signal vector of audio samples.

Delay and Defer operations: 
---------------------------
Max has several objects for delaying Max events and moving them into different threads. 
The **delay** and **pipe** objects delay bangs and messages respectively by some number of milliseconds, 
putting the delayed message on the *scheduler thread* queue to execute in the scheduler thread at a later time. To move
an event from the main (UI) thread into the scheduler thread,  we can send it through a **delay** or 
**pipe** object with the delay time set to zero, which will put the message on to the scheduler 
thread's queue for execution on the *next* pass of the scheduler thread. 

The **defer** and **deferlow** objects do the opposite: any incoming messages will be  
will be put on the queue for the low priority thread to execute on its next pass. 
The **deferlow** object puts the event on the end of the low priority queue while **defer** 
puts on the front.  If we send a message executing in the low priority thread into a **deferlow**, it
will be moved to the back of the queue, while a low priority event into a **defer** simple passes 
the message through without change. 

The S7 Garbage Collector
------------------------
S7 Scheme is a garbage-collected language, meaning that interpreter occasionally does a check
for memory that has been allocated but no longer has any references to it, and can thus be freed.
This saves as us from having to manually manage memory, as one does in languages like C, but has
latency implications because we never know when it will run.  Garbage collectors (GCs) are also used in 
dynamic languages such as JavaScript, Lisp, Ruby, and Python. (S7's GC is a mark-and-sweep 
garbage collector in case your into that sort of thing.) It will run occasionally, and 
typically takes 1-2 ms to run on my machine, depending on how long it has been since it
last ran, how many s7 objects are in memory, and how much memory s7 is using.

We can tell the gc to run on demand by calling **(gc-run)** from Scheme. 
We can enable and disable the garbage collector as well, with **(gc-enable)** and **(gc-disable)**,
and ask to run it only if it's enabled with **(gc-try)**. Most users won't need to bother
with these, but if you are trying to achieve low-latency with a large Scheme program,
manually running the gc at certain intervals or turning it off for the duration of a piece
and then running it between pieces can be helpful.

In practical terms, this means Scheme is not likely to work well for *audio* generation, where
2 ms is a long time, but for manipulation of musical events or musical input, it's not
noticeable. We notice a lag of 5 ms as smearing, and 20 ms as a discrete event, so a 2 ms
lag of the scheduler is not discernable, and if we are running with an audio latency of 
over this, it probably won't have any effect at all. For example, if we have an i/o
vector and signal vector set such that the system has an overall latency of 5-10ms, the
lag from the garbage collector is likely to be lower than the lag from our built in latency
and won't cause any issues.

S4M Thread Selection 
--------------------
The **s4m** object is locked to *either* the high priority or low priority thread with 
the **@thread** attribute. This can be set to 
**h** or **l**, and is **h** by default if you don't set it explicitly. In the s4m object,
any incoming message will be promoted or defered to ensure it's handled in the right thread
before being evaluated by the Scheme intepreter.
This prevents any threading errors in Scheme by ensuring that the 
interpreter won't be interupted in the middle of a critical operation (such as the garbage 
collector running) to start some other conflicting operation in the other thread.
The **@thread** attribute can *only* be set in the object box (not the inspector) as it cannot change
during the life of an **s4m** object. Leaving this empty is the same as **@thread h**. 
Note that this works fine with or without Audio Interrupt set to on; if you do have Audio
Interrupt set, **s4m** will execute all its events once per audio buffer calculation. (Or at least
as many as your scheduler settings allow).

If you want to use Scheme code in both low and high threads, the correct approach is
to have two **s4m** objects, and allow them to communicate with each other either by sending
messages to each other, or by using one of the thread-protected Max data store objects to which
Scheme can write, namely buffers, tables, and dictionaries. This essentially creates an Actor model, 
and if you wanted to, you could even implement a three-priority actor model by having one **s4m**
object only receive messages that had gone through a **deferlow**. This would delay events to the
third (lowest) **s4m** object such that they will only get serviced if Max has time
to get through the whole low priority queue on the main thread pass.

Note that this is also how the Ableton Live API objects work in Max For Live - it is always
in the low thread and any incoming messages are defered. So if you want to interact programmatically
with the Live API, you should do it from a low-thread s4m object. If you want to interact
with the Live API from a seqencer you're running in the high thread, you should have your
high thread s4m object send the low thread object a message telling it to trigger an API
action.

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
**js** object has quite a comprehensive set of features, it's not really reliable for timing critical operations.
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
 








