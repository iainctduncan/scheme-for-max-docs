Using Scheme For Max
====================

This page details all functionality offically released in the v0.1.
The **s4m** help file includes example patches for everything here.

s4m
-------
The s4m object is the core patcher object for Scheme For Max. It works
fairly similarly to the JS object, and can have a source file argument to load.
The interpreter is thread safe, so we can have multiple objects in a patch,
with isolated scheme environments.
 
arguments & properties 
--------------------------------------------------------------------------------

**{filename.scm}**
   A source file name as the first argument in the s4m object box
   will be loaded automatically, searching the Max file search path.
   Changing this file in the box will always wipe the scheme environment and reload
   the file, as this recreates the object. Resets will reload this file.

**@ins {num}**
   Sets the number of inlets to num. Changing will reset the interpreter.

**@outs {num)** 
   Sets the number of outlets to num. Changing will reset the interpreter.

Note, there is currently an issue whereby these can *only* be changed in the
object box, and thus must wipe the environment. This may be made dynamic
at a later date. Change in the property inspector or attempt to set
dynamically with messages at this time will not work.

When a source file has been listed in the object box, double clicking the 
object will open the file in the editor. The file will be reloaded on save,
which will *not* reboot the scheme interpreter. We can reboot the interpreter
at any time with the **reset** message.


messages to inlet 0 
--------------------------------------------------------------------------------
Messages to inlet 0 are either handled directly by the s4m object,
or executed as scheme code if no direct handling exists. The interpreter will
evaluate the token list as a scheme s-expression that is being called, with
any symbols evaluating to the their value in the scheme environment. Essentially
they are executed as if wrapped in an outer pair of parentheses. For example,
the message **foobar 1 2 3** will be evaluated in scheme as **(foobar 1 2 3)**.
This means that the message **out a** will only execute properly if the variable **a**
has been defined. To have **a** treated as a max symbol, not a scheme variable,
we single quote it. For example, **out 0 'a** will send the symbo "a" out output
0, having evaluated as **(out 0 'a)** in scheme.   

The messages **reset**, **read**, **scan**, and **eval-string** are handled
by the s4m object, but nontheless the semantics are the same. 
**eval-string "(define a 99)"** will execute as **(eval-string "(define a 99)")**,
which is defined in S7 scheme.

Messages reference for inlet 0:
 
**reset** 
   Resets the scheme intepreter, wiping all active definitions, and reloading any
   sourcefile specified in the s4m object box itself.

**read {filename}** 
   Loads the file {filename} from the Max search path. Executes the Scheme **load** 
   function internally, after finding the full file path. Loading does not erase any
   already active definitions unless they are redefined.

**scan**
   Scans the current patcher and all descendents for objects with scripting names,
   adding them to an internal registry so that they can receive messages with the
   **send** scheme function.

**{bang|int|float|list}**
   Evaluates the function **f-bang**, **f-int**, **f-float**, or **f-list**, with
   the respective max atom(s) as argument(s). I.E. the int **4** sent to inlet 0
   will executes in Scheme as **(f-int 4)**. If no f-type function defined, prints
   an error message to console. Result of evaluation is printed to console if
   not null, or if the **s4m-log-nulls** variable has been set to **#t**. 

**eval-string {string}** 
   Evaluates {string} in scheme. This is used by the REPL object by converting
   the text output to a single symbol with **tosymbol** and the prepending
   **eval-string**. Result of evaluation is printed to console.

**{symbol ...}**
   Evalues the symbol or list of symbols as a scheme s-expression. I.E. If sent
   the list **my-fun 1 2 3**, scheme will evaluate **(my-fun 1 2 3)**. Any symbols will 
   be evaluated as scheme variables unless quoted. For example, on **my-fun 1 a**,
   there will be an error if **a** has not been defined. Result of evaluation
   printed to console. If the first symbol is not callable as a scheme procedure,
   will produce an error.  


messages to inlet 1+
--------------------------------------------------------------------------------
Messages to inlet 1+ are treated as plain Max values or lists of Max atoms; they
are not evaluated as scheme code.
Symbols will become quoted symbols in scheme, with no variable evaluation. As these are 
not evaluated as code, the messages may begin with any type. Keywords are
a good choice as it indicates that the message is not a function call, becauses keywords
can not take the first position in a scheme function call. I.E. **(:foobar 1 2)** is always
an error in scheme, and thus **:foobar 1 2** will always be an invalid message to inlet 0, but
is valid in inlet 1+. It won't *do* anything unless we have registered a listener
function on the inlet receiving the message, with the keyword **:foobar**. 
See Registering Listeners below.

Messages reference for inlet 0:

**{bang|int|float|list}**
   Evaluates scheme function **(s4m-dispatch {inlet} {:bang|:int|:float|:list} {arg})**,
   which will call registered listener functions, with inlet as arg 1, and data as arg 2.
   For example, the message **4** on inlet 1 will call **(s4m-dispatch 1 :int 4)**, which will in turn
   call a listener function if registered.
   If no listener function is registered for the inlet used and the associated keyword
   (:int, :bang, etc), will produce an error. See Registering Listeners.
  
**{symbol ...}** 
   Evaluates scheme function **(s4m-dispatch {inlet} {symbol} arg)**, dispatching
   to listener functions registered with the symbol. Any arguments after the first
   symbol will be bound up in a list passed as **arg**, which may be empty.
   For example, the message **:foobar 1 2 3** on inlet 2 will call 
   **(s4m-dispatch 2 :foobar arg)**, where **arg** is the scheme object **(list 1 2 3)**.
   This will in turn call a listener function if registered.
   If no listener function is registered for the inlet used and the associated symbol
   will produce an error. See Registering Listeners for more details.


Scheme API
-----------
Below are details of the Scheme API. Core S7 functions are not listed here,
only functions added by Scheme-For-Max. Scheme functions are either defined
in **s4m.c** using the S7 Foreign Function Interface, or in Scheme code boostrapped
from the initial load of **scm4max.scm**.

File Loading
^^^^^^^^^^^^

**(load-from-max {filename string})**
   Load a file into Scheme. Acts excactly like Scheme standard **load** but finds
   the full path on the Max file search path first.


Console Output
^^^^^^^^^^^^^^
**(post {args...})**
   Post to the Max console. All arguments will be converted to string representations
   automatically. Post returns the null list.

.. code:: scm

   (define a 99)
   (post "my var a is" a)
   s4m> my var a is 99

Note: Console output by default does not print null responses, so that the console
does not print a message on every function call for side effects, such as **out**.
This can be changed by setting **s4m-log-nulls**.

.. code:: scm

   ;; output value 99 from out 9
   (out 0 99)
   ;; no logging appears on console
   (set! s4m-log-nulls #t)
   (out 0 99)
   s4m> ()

**(s4m-filter-result res)**
   Hook function to allow users to customize which results will be logged to the
   console. Redefine or alter this to return the keyword value **:no-log** to
   indicate result should not be logged.

.. code:: scm

   ;; if we replace what would be returned (res) by :no-log, s4m will not print to console
   (define (s4m-filter-result res)
      (cond 
        ;; turn off logging of the null list if set to do so
        ((and (null? res) (not s4m-log-nulls)) :no-log)
        ;; use the same setting to mute logging lists of nulls: (() () ())
        ((and (list? res) (every? null? res) (not s4m-log-nulls)) :no-log)
        (else res)))  


Outputs
^^^^^^^

**(out {outlet} {value})**
   Output {value} through outlet {outlet}. Value must be a single object.
   Returns the null list, thus by default does not log to console.

.. code:: scm

   ;; output number 99
   (out 0 99)
   ;; XXX: issued on this. output a max list of ints
   (out 0 (list 1 2 3))
   (out 0 '(1 2 3))
   ;; output a bang
   (out 0 'bang)
   ;; output the value of my-var
   (out 0 my-var)
   ;; output the max symbol "set"
   (out 0 'set)
   ;; XXX should output the max message "set 99", not working
   (out 0 (list 'set 99))

**(out-0 {value})**
   Convenience helper for **out X**, as sometimes a single argument function is helpful.
   Defined for outs 0-7, to add more, edit **scm4max.scm**
   Returns the null list, thus by default does not log to console.


Listeners
^^^^^^^^^

Inlet 0
+++++++

For inlet 0 to respond to bang, int, float, or list, we define functions named
accordingly.

.. code:: scm

   ;; respond to bang messages by logging to console and sending bang out
   (define (f-bang)
      (post "f-bang got the bang!")
      (out 0 'bang))
   
   ;; respond to int messages by logging to console and sending int + 1
   (define (f-int num)
      ;; log and output num + 1
      (post "f-int got the int: ", num)
      (out 0 (+ 1 num))) 
   
   ;; respond to lists by sending out in reverse the list elements
   (define (f-list list-arg)
      (map out-0 (reverse list-arg)))

Note that the f-list function will not respond to lists starting with a *symbol*.
Max doesn't consider those to be *list* messages, they are the message of 
the first *symbol*.

Any message starting with a symbol that is not alread reserved by s4m will be called
as a scheme function.

.. code:: scm

   ;; responds to max message "foobar 99" by outputing 99 
   ;; if sent max message "foobar my-var", will output the value of variable my-var
   ;; if sent max message "foobar 1 2 3", will be "too many arguments" error 
   (define (foobar value)
      (post "foobar exectuting, value:" value)
      (out 0 value))
   
   ;; responds to max messages "foobaz ..." with any number of args 
   ;; . args bundles variable list of optional args into a list
   (define (foobaz . args)
      ;; log and output num + 1
      (post "foobaz executing, num args: " (length args))
      ;; output the first arg if there is one, or null list if not
      (cond 
         ((> (length args) 0) (out 0 (args 0)))
         (else (post "no arg"))))

Note that in the above example we need to explictly check length args is > 0,
as in scheme, anything except #false is #true - there is no automatic cast from 0 to #f!


Inlet 1+
++++++++
For inlet 1+, we need to explictly register listener functions.
The listener functions registered with **listen** should always 
take one argument, expecting it to be a list that may be of length zero.
This allows the **s4m-dispatch** to be generic.

**(listen {inlet} {symbol} {function})**
   Register the function to listen on inlet {inlet} for messages starting
   with {symbol}. Listeners are called by s4m's **s4m-dispatch** function, which
   will dispatch calls with the keyword symbols **:bang**, **:int**, **:float**, 
   and **:list** for non symbolic messages. 

.. code:: scm
   
   ;; define a listener for bangs, note that it takes an arg of a list 
   ;; even though this will in practise be empty on bang messages
   (define (my-bang-func args)   
       (post "got the bang!"))
   ;; register it to listen for bangs on inlet 1
   (listen 1 :bang my-bang-func)
   
   ;; define a listener for int messages, using an anonymous function
   (listen 1 :int (lambda (args)
      (out 1 (args 0))))

   ;; the same function can listen for multiple messages
   (define (num-listener args) 
      (out 0 (+ 1 (args 0)))
   (listen 1 :int num-listener)
   (listen 1 :float num-listener)

   ;; a listener using a let to hide the signature weirdness
   (define (my-listener args)
      (let ((num (args 0)))
         (post "num is:" num)
         (out 0 (+ 1 num))))

Listeners are stored internally in the **s4m-listeners** registry,
a nested hashtable of {inlet} {symbol}. To remove a listener, you can
put an empty function in:

.. code:: scm
   
   ;; remove the listener by registering false
   (listen 1 :int #f)

Note that if you redefine a named listener function, it will not
change what happens on the listened to message until you re-register
it, by virtue of how scheme functions work. (We are registering the
actual function, not the symbol of the function.)
 
  
Sending Messages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We can send arbitrary Max messages to other Max objects that have been given
a scripting name. Before doing so, we must send the **scan** message to the
s4m object, which will scan the current patcher and all descendents,
registering scripting names interally in s4m object (in the C code).

**(send {target} {msg} ... {atoms})**
   Send the message {msg}, which may be followed by any number of 
   values to be handled as Max atoms.

.. code:: scm
   
   ;; update the contents of a number box with scripting name num-target
   ;; we quote num-target below as we want the symbol num-target, not the
   ;; value of a variable named num-target. 
   (send 'num-target 99)

   ;; send a message box a message to update to the contens "foobar 1 2 3"
   (send 'msg-target 'set 'foobar 1 2 3)
  
   ;; this means we can send a message as list using scheme's apply function
   (define msg-list (list 'set 1 2 3))
   (apply send (cons 'msg-target msg-list)) 


This is defined for us as send-list.

**(send-list {target} {msg-list})**
   Send the message {msg-list}, a single arg of a list.

.. code:: scm
   
   (define msg-list (list 'set 1 2 3))
   (send-list 'msg-target msg-list)
   

This can be used to integrate with all kinds of max objects, including
updating colls, dicts, tables, etc. We can copy whatever message the object
receives and send them.    


s4m.repl patcher
-----------------
The s4m.repl object is intended to be put in a **bpatcher** and then hooked up.
The left outlet sends the output as a single text symbol. To evaluate
as scheme, we send it to a **prepend eval-string** object and send to inlet
0. This makes it the equivalent of:

.. code:: scm
 
   (eval-string "(define a 99)")

The right outlet sends out a bang on each output to let you know it went.

The s4m.repl patcher wraps the **textedit** box, which has some quirks/bugs.
It wants to send out a bang when one bangs or hits enter in an empty box.
In order to prevent scheme error messages on this instance, **s4m.repl** filters
these out.

If you select **Control Keys** on it, the **s4m.repl** object is listening
to *all* key strokes, no matter where your focus is. So if you use this feature,
be sure to turn it off when you're done. This can be especially confusing if
you have multiple REPLs in different Max windows!

Future plans include making a proper terminal GUI object with history.
