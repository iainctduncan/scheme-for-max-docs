Overview
=========

Scheme For Max (S4M) is an open source project that enables scripting and live coding Max 8
and Max For Live with S74 Scheme. S74 Scheme is a thin layer over s7 Scheme, which is a minimal, 
Common-Lisp inspired Scheme implementation, created by Bill 
Schottstaedt at CCRMA, and used in the Common Music algorithmic composition toolkit 
and the Snd audio editor. S74 adds some beginner friendly and higher level Scheme language features
to core S7, drawing from sources such as Racket, Clojure, and Chicken Scheme. 
Scheme for Max is authored by Iain Duncan, and hosted on GitHub.

If you'd like to see a demo of Scheme For Max in action, there are videos on the 
`Music With Lisp <https://www.youtube.com/channel/UC6ftX7yuEi5uUFkRVJbJyWA>`_ channel on 
YouTube.

If you're new to Scheme, there is a tutorial that assumes no Lisp experience here:
https://iainctduncan.github.io/learn-scheme-for-max/introduction.html

And a more advanced tutorial on building sequencers here:
https://iainctduncan.github.io/s4m-stk/

These docs are in Scheme-For-Max-Docs repository on GitHub. If you find anything unclear,
incorrect, or just have suggestions, please post a ticket or let me know on the Google Group.

S4M is available as a Max 8 package for OSX and both 32 and 64 bit Windows, as 
well as source code for Windows and OSX, for Max 7 or 8. It also is tested using
Ableton Live 10 or 11 and Max For Live.

Features of v0.3 include:

* Hot reloading of scheme code
* A built in REPL terminal editor for interactive coding 
* Max messages on inlet 0 automatically execute as Scheme code 
* Dynamically registered listener functions for Max messages on inlets 1+
* Sending messages to remote objects by scripting name
* Ability to run in either the high or low priority thread
* Table access and i/o
* Buffer access and i/o
* Dictionary access and i/o, including nested lookup
* Integration with the Max transport controls 
* High-accuracy event and function scheduling
* Support for Max time notation for scheduling
* Quantization with master transport settings
* Support for Max4Live and the Live API
* Garbage collector interface functions for high performance

The scheduling and thread support make Scheme For Max particularly useful for
timing critical tasks, where the JavaScript object can not be reliably
used as it runs only in the low-priority thread.

S4M provides the following Max patchers:

* s4m - The embedded interpreter
* s4m.repl - A patcher for making a terminal REPL in Max
* s4m.help - An extensive help file demoing all features with sample source code

Development is tracked on GitHub at `github.com/iainctduncan/scheme-for-max <https://github.com/iainctduncan/scheme-for-max>`_
