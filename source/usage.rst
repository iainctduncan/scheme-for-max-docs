Using Scheme For Max
====================

This page details all functionality offically released in the v0.2.
The **s4m** help file includes example patches for most of what is here.

The s4m object
---------------
The **s4m** object is the core patcher object for Scheme For Max. It works
fairly similarly to the **js** object, and can have a source file argument to load.
The interpreter is thread safe, so we can have multiple objects in a patch,
with each running its own isolated Scheme environments.
 
On creation of an **s4m** object, the interpreter first loads **s4m.scm**, which
boostraps the S4M environment with any housekeeping that has been implemented in Scheme.
This includes loading several other files, all of which must be present in the Max
file path. If you have installed S4M from a package, this should all "just work". 
If you're getting errors about missing .scm files, you have likely installed the package
incorrectly, have moved somethings, or have an error with your Max filepaths settings.
Max *must* be able to find both the compiled s4m external and the scm files for bootstrapping.

One of the files loaded by **s4m.scm** is **s74.scm**. This holds pure Scheme
functions that have nothing to do with Max, but augment S7 with convenience functions
you may want to use. If you want to run pure S7, you can comment out this inclusion.
It's mostly functions that are common in more batteries-included Schemes, such 
as Racket or Clojure. 
 
You can change the **s4m.scm** file if you want to change bootstrap behaviour across all
your **s4m** objects as well, such as to always load a personal library of Scheme 
definitions.

Arguments & Attributes 
--------------------------------------------------------------------------------

**{filename.scm}**
   An (optional) source filename as the first argument in the s4m object box
   will be loaded automatically (after the bootstrap files) searching the Max file search path.
   Changing this file in the box will always wipe the Scheme environment and reloads
   the file, as this recreates the **s4m** object. Reset messages will also reload this file.

**@ins {num}**
   Sets the number of inlets to num. This can only be set in the object box. 
   Changing will reset the interpreter as it recreates the object.

**@outs {num)** 
   Sets the number of outlets to num. This can only be set in the object box. 
   Changing will reset the interpreter as it recreates the object.

**@thread { h | l | a }**
   Sets the thread in which Scheme code is executed. Setting to **h** will 
   ensure all messages are promoted to the high-priority scheduler thread, while
   **l** defers all inputs to the low priority main thread. Setting to **a** (any)
   allows execution in whichever thread the incoming message is in, with no 
   promotion or deferal. (**a** is meant for testing/debugging and is unstable.)
   This attribute can only be set in the object box. 
   Changing will reset the interpreter as it recreates the object.

**@log-repl {1 | 0}**
   Sets whether the interpreter prints the output from the REPL to the Max console.
   When log-repl is enabled, any Scheme expression evaluated will have its return value printed. 
   This can be changed anytime in both the inspector and by sending a **log-repl 1** message.

**@log-null {1 | 0}**
   Sets whether the interpreter prints the output from the REPL to the Max console
   when the output is the null list in Scheme, returned by many Scheme functions 
   that are called for their side effect. It is often useful to turn this off
   when running scheduled Scheme functions on a timer, for example.
   This can be changed anytime in both the inspector and by sending a **log-null 1** message.

When a source file has been listed in the object box, double clicking the 
object will open the file in the editor. The file will be reloaded on save,
which will *not* reboot the Scheme interpreter. We can reboot the interpreter
at any time with the **reset** message.

Reserved Words and Naming Conventions
-------------------------------------
In Max, messages to an object that consist of the name of an attribute and a value are
(typically) executed automatically to set that attribute, if the object permits it. This means
that we can't have Scheme functions with names that collide with attributes of **s4m**
if we want these functions to be callable as automatically evaluated s-expressions on 
inlet 0 - the message will never get to the Scheme interpreter. A further complication is
that in Max, the messages **int**, **float**, **symbol**, and **list** are implicit - 
if you send an object the message **int 4** or **list a b c**, they are treated by the 
receiving object as being the exact same as the message **4** or **a b c**. 

The easiest solution is to avoid using the following as names in Scheme unless you know
exactly why you are doing it:

Max reserved words: **int**, **float**, **symbol**, **get**, **set**

S4M attributes: **thread**, **ins**, **outs**, **log-repl**, **log-null**

S4m commands: **reset**, **scan**, **read**, **eval-string**

The astute reader will have noticed that **list** is missing above. This is because
**list** is already a Scheme function. This is also why our handlers for bang, int, float, and
list are called **f-bang**, **f-int**, **f-float**, and **f-list** - to avoid a weird
inconsistency as we can't call the Scheme list hander **list**.

S4M does not yet use the **get** and **set** message for anything special, but it's 
common in Max to use these to set state of an object, so consider them reserved
for future use. Fortunately for us, in Scheme the set command is **set!**, so we
can happily have **set foo bar** messages implemented in the object (but not in Scheme)
in future without collision.

With regard to naming conventions, Scheme variables and functions can accept special characters
and normally separate words with hyphens. Reader functions are usually **(foobar-ref {key})** and
setter functions are usually **(foobar-set! {key} {value})**. 
We are following Scheme and Max naming conventions
where possible, with Scheme as the priority when in doubt. Predicates end in ? and
functions with side effects end in !, so we have for example: **dict-ref**, **dict-set!**, 
and **dict?**. 

S4M also includes short form aliases for most function names to simplify live coding
and building s-expressions in Max messages where space may be at a premium. These
are 4 to 6 character abbreviations and do not follow Scheme conventions (e.g. **tabr**, **tabs**, etc.)
Aliases do not create an extra stack frame; they are registered at the C level
so there is no speed penatly in using them.

There are various functions defined in the bootstrap files. If you want to know
whether your function name is going to shadow anything already defined, just evaluate
it in the REPL and see if you get anything. You should get an 'unbound variable' error
in the console if there is no naming collision.




Messages to inlet 0 
--------------------------------------------------------------------------------
Messages to inlet 0 are either handled directly by the s4m object as a Max message,
or evaluated as Scheme code if no direct handling exists. If the message is not
handled by the **s4m** object as a Max message, the interpreter will
evaluate the message as a Scheme s-expression that is being called, with
any symbols evaluating to their value in the Scheme environment. 
This means messages are essentially executed as if wrapped in an outer pair of parentheses. For example,
the message **foobar 1 2 3** will be evaluated in Scheme as **(foobar 1 2 3)**.
This also means that the message **out a** will only execute properly if the variable **a**
has been defined. To have **a** treated as a Max symbol, not a Scheme variable,
we need to quote it in the standard Lisp fashion. For example, **out 0 'a** will send 
the symbol "a" out output 0, having been evaluated as **(out 0 'a)** in Scheme.   

The messages **reset**, **read**, **scan**, and **eval-string** are handled
directly by the **s4m** object, without passing to the interpreter (see Message reference below).

To evaluate a block of code which includes punctuation that is problematic in regular
Max message boxes, the code should be converted to a string 
symbol with the **tosymbol** object, and then sent to Scheme by using a 
**prepend eval-string** object. This will result in **s4m** receiving something like
**eval-string "(define a 99)"**, which will execute as **(eval-string "(define a 99)")** 
in the Scheme interpreter.

Practically speaking, there is no difference in execution between sending inlet
0 of an **s4m** object either **define a 99** or **eval-string "(define a 99)"**,
but the second will allow nested s-expressions, as in 
**eval-string "(define my-list (list 1 2 3))"**. 


Messages reference for inlet 0:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
 
**reset** 
   Resets the Scheme intepreter, wiping all active definitions, and reloading any
   sourcefile specified in the s4m object box itself.

**read {filename}** 
   Loads the file {filename} from the Max search path. Executes the Scheme **load** 
   function internally, after finding the full filepath by searching the Max filepaths. 
   Loading does not erase any already active definitions unless they are redefined.
   Rereading a file will redefined any definitions.

**scan**
   Scans the current patcher and all descendents for objects with scripting names,
   adding them to an internal registry so that they can receive messages with the
   **send** scheme function.

**{bang | int | float | list}**
   Evaluates the function **f-bang**, **f-int**, **f-float**, or **f-list**, with
   the respective max atom(s) as argument(s). I.E. the int **4** sent to inlet 0
   will executes in Scheme as **(f-int 4)**. If no f-type function defined, prints
   an error message to console. Return value of evaluation is printed to the Max console.

**eval-string {string}** 
   Evaluates {string} in scheme. This is used by the REPL object by converting
   the text output to a single symbol with **tosymbol** and the prepending
   **eval-string**. Return value of evaluation is printed to the Max console.

**{symbol ...}**
   Evalues the symbol or list of symbols as a Scheme s-expression. If sent
   the list **my-fun 1 2 3**, scheme will evaluate **(my-fun 1 2 3)**. Any symbols will 
   be evaluated as Scheme variables unless quoted. For example, on **my-fun 1 a**,
   there will be an error if **a** has not been defined. Return value of evaluation is
   printed to the Max console. If the first symbol is not callable as a Scheme procedure,
   will produce an error (as if a symbol is evaluated surrounded with parentheses).

Messages to inlet 1+
--------------------------------------------------------------------------------
Messages to inlet 1+ are treated as plain Max values or lists of Max atoms; they
are not evaluated as Scheme code.
Symbols will become quoted symbols in scheme, with no variable evaluation. As these are 
not evaluated as code, the messages may begin with any type. Keywords are useful here
as they indicate visually that the message is not a function call, becauses keywords
can not be used as function names in Scheme. Thus **(:foobar 1 2)** is always
an error in Scheme. This means **:foobar 1 2** will always be an invalid message to inlet 0, but
may be valid in inlet 1+ depending on what we have defined. It won't *do* anything unless we 
have registered a listener function on the inlet receiving the message, with the keyword **:foobar**. 
See Registering Listeners below. (If all this is confusing, skip it for now - you could use
s4m productively without ever using inlets 1+)

Messages reference for inlet 0:

**{bang | int | float | list}**
   Evaluates Scheme function **(s4m-dispatch {inlet} {:bang | :int | :float | :list} {arg})**,
   which will call registered listener functions, with inlet as arg 1, and data as arg 2.
   For example, the message **4** on inlet 1 will call **(s4m-dispatch 1 :int 4)**, which will in turn
   call a listener function if a matching listener is registered.
   If no listener function is registered for the inlet used and the associated keyword
   (:int, :bang, etc), this will produce an error. See Registering Listeners.
  
**{symbol ...}** 
   Evaluates scheme function **(s4m-dispatch {inlet} {symbol} arg)**, dispatching
   to listener functions registered with the symbol. Any arguments after the first
   symbol will be bound up in a list passed as **arg**, which may be empty.
   For example, the message **:foobar 1 2 3** on inlet 2 will call 
   **(s4m-dispatch 2 :foobar arg)**, where **arg** is the Scheme value **(list 1 2 3)**.
   This will in turn call a listener function if registered.
   If no listener function is registered for the inlet used and the associated symbol, this
   will produce an error. See Registering Listeners for more details.


Registering Listeners for Max messages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Inlet 0
+++++++

For inlet 0 to respond to bang, int, float, or list, we define functions named
as below:

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

  ;; respond to float messages by sending out arg * 0.5 
   (define (f-float num)
      (post "f-float got the float: ", num)
      (out 0 (* 0.5 num))) 
 
   ;; respond to lists by sending out in reverse the list elements as sequential messages
   (define (f-list list-arg)
      (map out-0 (reverse list-arg)))

Note that the f-list function will not respond to lists starting with a *symbol*.
Max doesn't consider those to be *list* messages, they are the message of 
the first *symbol*.

Any message starting with a symbol that is not alread reserved by S4M will be called
as a Scheme function.

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
because in Scheme anything except **#false** is **#true** - there is no automatic cast from **0** to **#f**.


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
change what happens on the listened-to message until you re-register
it, by virtue of how Scheme functions work. (We are registering the
actual function, not the symbol of the function!)
 


s4m.repl patcher
-----------------
The s4m.repl object is intended to be put in a **bpatcher** and then hooked up.
The left outlet sends the output as a single text symbol. To evaluate
as Scheme, we send it to a **prepend eval-string** object and send to inlet
0. This makes it the equivalent of:

.. code:: scm
 
   (eval-string "(define a 99)")

The right outlet sends out a bang on each output to let you know it went out.

The s4m.repl patcher wraps the **textedit** box, which has some quirks/bugs.
It wants to send out a bang when one bangs or hits enter in an empty box.
In order to prevent Scheme error messages on this instance, **s4m.repl** filters
these out.

If you select **Control Keys** on it, the **s4m.repl** object is listening
to *all* key strokes, no matter where your focus is. So if you use this feature,
be sure to turn it off when you're done. This can be especially confusing if
you have multiple REPLs in different Max windows!

Future plans include making a proper terminal GUI object with history.
