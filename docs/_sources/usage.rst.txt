Using Scheme For Max
====================

s4m.scm Object Reference 
------------------------

Properties of s4m.scm
^^^^^^^^^^^^^^^^^^^^^^
A source file name as the first argument in the s4m.scm object box
will be loaded automatically, searching the Max file search path.
Changing this file in the box will always wipe the scheme environment and reload
the file, as this recreates the object.

**@ins {num}**
   Sets the number of inlets to num

**@outs {num)** 
   Sets the number of outlets to num

Note, there is currently an issue whereby these can *only* be changed in the
object box, and thus must wipe the environment. This may be made dynamica
at a later date. Do not change in the property inspector or attempt to set
dynamically with messages at this time.

When a source file has been listed in the object box, double clicking the 
object will open the file in the editor. The file will be reloaded on save,
which will *not* reboot the scheme interpreter. We can reboot the interpreter
at any time with the **reset** message.


Messages to inlet 0 
^^^^^^^^^^^^^^^^^^^
Messages to inlet 0 are either handled directly by the s4m.scm object,
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
by the external directly, but nontheless the semantics are the same. 
**eval-string "(define a 99)"** will execute as **(eval-string "(define a 99)")**,
which is defined in S7 scheme.

Messages reference for inlet 0:
 
**reset** 
   Resets the scheme intepreter, wiping all active definitions, and reloading any
   sourcefile specified in the s4m.scm object box itself.

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


Messages to inlet 1+
^^^^^^^^^^^^^^^^^^^^
Messages to inlet 1+ are treated as simple Max values or lists of Max atoms; they
are not evaluated as scheme code.
Symbols will become quoted symbols in scheme, with no variable evaluation. As these are 
not evaluated as code, the messages may begin with any type. Keywords are
a good choice as it's clear at a glance that the is not a function call, becauses keywords
can not take the first position in a function call. I.E. **(:foobar 1 2)** is always
an error in scheme, and thus **:foobar 1 2** will always be an invalid message to inlet 0, but
is valid in inlet 1+. It won't *do* anything unless we have registered a listener
function on the inlet receiving the message, with the keyword **:foobar**. 
See Registering Listeners below.

Messages reference for inlet 0:

**{bang|int|float|list}**
   Evaluates scheme function **(s4m-dispatch {inlet} {:bang|:int|:float|:list} {arg})**,
   which will call registered listener functions, with inlet as arg 1, and data as arg2.
   For example, the message **4** on inlet 1 will call **(s4m-dispatch 1 :int 4)**, which will in turn
   call a registered listener function called my-listener as **(listener 4)**.
   If no listener function is registered for the inlet used and the associated keyword
   (:int, :bang, etc), will produce an error. See Registering Listeners.
  
**{symbol ...}** 
   Evaluates scheme function **(s4m-dispatch {inlet} {symbol} arg)**, dispatching
   to listener functions registered with the symbol. Any arguments after the first
   symbol will be bound up in a list passed as **arg**, which may be empty.
   For example, the message **:foobar 1 2 3** on inlet 2 will call 
   **(s4m-dispatch 2 :foobar arg)**, where **arg** is the scheme object **(list 1 2 3)**.
   This will in turn call a listener that has been registered with 
   **(listen 2 :foobar {function})***. 
   If no listener function is registered for the inlet used and the associated symbol
   will produce an error. See Registering Listeners for more details.


Scheme API
-----------
Below are details of the Scheme API. Core S7 functions are not listed here,
only functions added by Scheme-For-Max.

Console Output
^^^^^^^^^^^^^^

**post {args...}**
   Post to the Max console. All arguments will be converted to string representations
   automatically. Post returns the null list.

.. code:: scm

   (define a 99)
   (post "my var a is" a)
   s4m> my var a is 99


Registering Listeners
^^^^^^^^^^^^^^^^^^^^^
left off here...
 
