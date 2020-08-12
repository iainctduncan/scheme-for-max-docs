Overview
=========

Scheme For Max (S4M) is an open source project that enables scripting and live coding Max 8
with S7 Scheme. S7 is a minimal, "lisp-y" Scheme implementation, created by Bill 
Schottstaedt at CCRMA, and used in the Common Music algorithmic composition toolkit 
and the Snd audio editor. Scheme for Max is authored by Iain Duncan, and hosted on GitHub.

If you'd like to see a demo of Scheme For Max in action, there is a video on the 
`Computer Music Tools <https://www.youtube.com/channel/UC6ftX7yuEi5uUFkRVJbJyWA>`_ channel on 
YouTube.

These docs are in Scheme-For-Max-Docs repository on GitHub. If you find anything unclear,
incorrect, or just have suggestions, please post a ticket or let me know on the Google Group.

S4M is available as a Max 8 package for both OSX and both 32 and 64 bit Windows, as 
well as source code for Windows and OSX, for Max 7 or 8. 

Features of v0.1 include:

* Hot reloading of scheme code
* Text editor integration (similar to the JS object)
* A built in REPL terminal editor for interactive coding 
* Max messages on inlet 0 execute as scheme code 
* Dynamically registered listener functions for Max messages on inlets 1+
* Sending messages to remote objects by scripting name


Features under development, to be released for v0.2:

* Table access and i/o
* Buffer access and i/o
* Dictionary access and i/o
* Deferred event scheduling
* Integration with Max4Live

S4M provides the following Max patchers:

* s4m - The embedded interpreter
* s4m.repl - A patcher for making a terminal REPL in Max
* s4m.help - An extensive help file demoing all features with sample source code

Development is tracked on GitHub at `github.com/iainctduncan/scheme-for-max <https://github.com/iainctduncan/scheme-for-max>`_
