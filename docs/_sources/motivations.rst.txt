Motivation - Why Lisp in Max?
==============================
The purpose of s4m is to bring some of the power, convenience, and flexibility of
Lisp to the Max platform. This grew from several motivations and disatisfactions
with the existing JavaScript options (the js object, and the newer Node-for-Max object).

**Project Goals:**

* Hot-reloading of code, so that one can use the Lisp workflow of incrementally
  changing code while the program runs, preserving state where desired
* Ability to send code to a running max patch from a REPL in a terminal, to allow
  live coding directly in the Max environment or from a connected network REPL (ie Emacs)
* Ability to create domain specific languages (DSL's) appropriate for algorithmic composition,
  ideally building on the work that has been done in this area with projects like Common Music
* Deep integration with Max, through the C SDK, with fine grained control of execution
  timing and threading
* Ability to move between a lisp context and the Max context more easily than is possible
  with the JS object


Scheme Lisp provides all the above very well. We can load source files into
the interpreter without needing to wipe out the running session. This allows us
to keep state variables in one file and code under development in another, reloading
function definitions as we go, while the patch is running.  

We can live-code from a REPL editor by sending blocks of code to the s4m.scm object
as strings, prepended with the **eval-string** symbol. **s4m.repl** is a simple multi-line terminal
editor object that we can use to interact with the interpreter in real time in Max. 
Future plans include supporting network REPLs through Emacs and the like.

Scheme also allows us to create simple DSL's so that we can send blocks of code
to the interpreter that we dynamically generate with Max patcher objects. Scheme's minimal syntax
and use of whitespace as token separators make this easy to implement. The s4m.scm
object takes any list of max symbols sent to inlet 0, and treats them as a list of tokens
in a Lisp s-expression, evaluating them as if they were enclosed in an outer set of
parantheses. I.E. The max message **define foobar 1** becomes **(define foobar 1)**.
When combined with max message building objects like **sprintf**, **pack**, and so on,
and the **$** interpolation features of the message box, this makes it easy to assemble
valid scheme expressions from standard Max objects and symbols. Future plans
include porting the threading macros from Clojure to enable compound statements
in max expressions, such as **+ 1 my-var -> out 0** to represent **(out 0 (+ 1 my-var))**

The **s4m.scm** external is open-sourced, and written using the C SDK and the S7 Scheme
Foreign Function Interface.  S7's FFI enables defining scheme functions from C, and 
assembling Scheme objects in C from Max atoms. The user-developer can thus add
any functionality in the Max SDK to the scheme interpreter, and can also call scheme
functions from C code that in response to messages to the s4m.scm object.  
We can also move execution from Scheme to C by defining a wrapper, giving us 
the opportunity to optimize execution timing and performance as needed. 

Further details on this topic will be forthcoming, including why S7 was chosen and how it compares
to other Lisps.

