Overview
=========

Scheme For Max (s4m) is an open source project that enables scripting and live coding Max 8
with S7 Scheme. S7 is a minimal, "lisp-y" Scheme implementation, created by Bill 
Schottstaedt at CCRMA, and used in the Common Music algorithmic composition toolkit 
and the Snd audio editor.

S4M is available as a Max 8 package, as well as source code that should build
on both Windows and OSX, for Max 7 or 8. The Max package is available for 64-bit OSX
only at this time, but help building on Max 7 and Windows would be most welcome!
Releases are available on the GitHub site.

Features of v0.1 include:

* Hot reloading of scheme code
* Text editor integration (similar to the JS object)
* A built in REPL terminal editor for interactive coding 
* Max messages on inlet 0 execute as Scheme code 
* Dynamically registered listener functions for Max messages on inlets 1+
* Sending messages to remote objects by scripting name


Features under development, to be released for v0.2 include:

* Dictionary access and i/o
* Table access and i/o
* Buffer access and i/o
* Integration with Max4Live

S4M provides the following Max patchers

* s4m.scm - The embedded interpreter
* s4m.repl - A patcher for making a terminal REPL in Max
* s4m.scm.help - An extensive help file demoing all features with sample source code

