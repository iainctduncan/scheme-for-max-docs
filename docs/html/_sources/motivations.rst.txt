Motivation - Why Lisp in Max?
==============================
The purpose of Scheme For Max is to bring some of the power, convenience, and flexibility of
Scheme and lisp to the Max platform. 

**Project Goals:**

* Hot-reloading of code, so that one can use the lisp workflow of incrementally
  changing code while the program runs, preserving state where desired
* Ability to send code to a running max patch from a REPL in a terminal, to allow
  live coding directly in the Max environment or from a connected network REPL (ie Emacs)
* Ability to create domain specific languages (DSL's) appropriate for algorithmic composition,
  ideally building on the work that has been done in this area with projects like Common Music
* Deep integration with Max, through the C SDK, with fine grained control of execution
  timing and threading
* Ability to move between a lisp context and the Max context more easily than is possible
  with the JS object

Embedding the S7 Scheme interpreter in a Max external has worked out well for achieving these.

We can load source files into the interpreter without needing to wipe out the running session. This allows us
to keep state variables in one file and code under development in another, reloading
function definitions as we go, while the patch is running.  

We can live-code from a REPL editor by sending blocks of code to the s4m object.
**s4m.repl** is a simple multi-line terminal
editor object that we can use to interact with the interpreter in real time in Max. 
Future plans include supporting network REPLs through Emacs and the like.

Scheme also allows us to create simple DSL's so that we can send blocks of code
to the interpreter that we dynamically generate with Max patcher objects. Scheme's minimal syntax
and use of whitespace as token separators make this easy to implement. The s4m
object takes any list of max symbols sent to inlet 0, and treats them as a list of tokens
in a Lisp s-expression, evaluating them as if they were enclosed in an outer set of
parantheses.  We can thus assemble valid Scheme expressions from standard Max objects and symbols.
With S7's support for Common Lisp style macros, we can extend this into a domain
specific language as powerful as we want. 

The **s4m** external is open-sourced, and written using the C SDK and the S7 Scheme
Foreign Function Interface.  S7's FFI enables defining Scheme functions from C, and 
assembling Scheme objects in C from Max atoms. The user-developer can thus add
any functionality in the Max SDK to the Scheme interpreter, and can also call Scheme
functions from C code in response to messages to the **s4m** object.  
We can thus move operations from Scheme to C and vice versa, allowing us to opimize
for performance or flexibility wherever we'd like.

Longer term plans for this documentation include further sections introducing
S7 Scheme, documenting the basics of the language, comparing it to other lisps,
and explaining why it was chosen.

Of course, as Scheme For Max is open-source, you're welcome to port it to 
another lisp implementation too! 

