Lisp and Max in Detail
======================

Dynamic Evaluation
^^^^^^^^^^^^^^^^^^
While it is possible to send code to JS to have it dynamically evaluated, the js object
doesn't support this specifically, and it is thus very cumbersome. Generation
of runnable JS code from outside of the object is very difficult, given the syntax
heavy nature of JS, the frequent use of strings in JS code, and the fact that other
Max objects don't work very easily with strings. In contrast, building valid lisp
s-expressions in max messages is simple.  

* In Lisp, tokens are separated by spaces, as are tokens in Max messages
* Max messages don't like strings: quotation marks need to be escaped, and
  frequently disappear when messages are sent to each if care is not taken to treat
  the whole message as one max symbol
* The single-quote convention for quoting a symbol in lisp works in Max
  messages: Max accepts 'foobar as a symbol and will pass it on to other objects intact
* In most places, Max will accept a leading colon for a symbol, allowing use of
  Lisp keywords, that always evaluate to themselves.
* While parentheses are used to enclose s-expressions in lisp, lisp macros can
  be used to 

Syntax
^^^^^^
Lisp syntanx

The main supported option in Max is the JS object, and the newer Node-for-Max object.

* JavaScript syntax does not map well to Max syntaxt, meaning that the 
* The JS object uses older JavaScript
* The Node-for-Max object runs in a separate process with very limited direct Max API access  
