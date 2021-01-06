S4M Scheme API
---------------
Below are details of the Scheme API. Core S7 functions are not listed here,
only functions added by S4M. Scheme functions are either defined
in **s4m.c** using the S7 Foreign Function Interface, or in Scheme code boostrapped
from the initial load of **s4m.scm**, including the **s74.scm** file.


File Loading
^^^^^^^^^^^^

**(load-from-max {filename string})**
   Load a file into Scheme. Acts excactly like Scheme standard **load** but finds
   the full path on the Max file search path first.

.. TODO: stuff about threading and file loading??
 

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
This can be changed by setting **log-null** to 1, with either a message, the attribute,
or the inspector.


**(s4m-filter-result res)**
   Hook function to allow users to customize which results will be logged to the
   console. Redefine or alter this to return the keyword value **:no-log** to
   indicate result should not be logged.

.. code:: scm

   ;; if we replace what would be returned (res) by :no-log, s4m will not print to console
   (define (s4m-filter-result res)
      (cond 
        ;; turn off logging of the returned integers
        ((int? res) :no-log)
        ;; for everything else, do the normal thing 
        (else res)))  


Outputs
^^^^^^^

**(out {outlet} {value})**
   Output {value} through outlet {outlet}. Value must be a single object.
   Returns the null list, thus by default does not log to console.

.. code:: scm

   ;; output number 99
   (out 0 99)
   ;; output a max list of ints
   (out 0 (list 1 2 3))
   (out 0 '(1 2 3))
   ;; output a bang
   (out 0 'bang)
   ;; output the value of my-var
   (out 0 my-var)
   ;; output the max symbol "set"
   (out 0 'set)
   ;; output the max message "set 99"
   (out 0 (list 'set 99))

**(out-0 {value})**
   Convenience helper for **out X**, as sometimes a single argument function is helpful.
   Defined for outs 0-7, to add more, edit **s4.scm**
   Returns the null list, thus by default does not log to console.

  
Sending Messages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We can send arbitrary Max messages to other Max objects that have been given
a scripting name. Before doing so, we must send the **scan** message to the
**s4m** object, which will scan the current patcher and all descendents,
registering scripting names interally in the **s4m** object (in the C code).

**(send {target} {msg} ... {atoms})**
   Send the message {msg}, which may be followed by any number of 
   values to be handled as Max atoms.

.. code:: scm
   
   ;; update the contents of a number box that has scripting name num-target
   ;; we quote num-target below as we want the symbol num-target, not the
   ;; value of a variable named num-target. 
   (send 'num-target 99)

   ;; send a message box a message to update to the contents "foobar 1 2 3"
   (send 'msg-target 'set 'foobar 1 2 3)
  
   ;; this means we can send a message as list using scheme's apply function
   (define msg-list (list 'set 1 2 3))
   (apply send (cons 'msg-target msg-list)) 


There is also a convenience helper, **send-list**.

**(send-list {target} {msg-list})**
   Send the message {msg-list}, a single arg of a list.

.. code:: scm
   
   (define msg-list (list 'set 1 2 3))
   (send-list 'msg-target msg-list)
   

This can be used to integrate with all kinds of Max objects, including
updating colls, dicts, tables, etc. We can copy whatever message the object
receives and send them.    


Table I/O 
^^^^^^^^^
We can write and read integer data directly to and from named Max tables, as well as query
for the existence and size of Max tables. Max tables are useful
for storing integer data that we want to share between an **s4m** object
and the rest of Max (or another **s4m** object. Tables are queried in each function, so if you want
to read or write a lot of data, the vector conversions functions will be faster than a loop
of single point table operations.

Short form aliases exist for all functions and are identical except for the function name.


**(table? {table-name})**
    Returns #true if the symbol {table-name} corresponds to a named Max table.

**(table-length {table-name})**
    Returns integer length of table {table-name}. Error if not a valid table.
    Alias: **tabl**

**(table-ref {table-name} {index})**
    Returns value at {index} in table {table-name}. Alias: **tabr**

**(table-set! {table-name} {index} {value})**
    Sets value at {index} in table {table-name} to {value}. Alias: **tabs**

**(table->vector {table-name} {opt. index} {opt. count})**
    Returns a new Scheme vector from contents of a table, starting
    at index 0 or {index} in the table, copying entire table or {count} points
    if optional {count} given. Alias **t->v**

**(table-set-from-vector! {table-name} {table-index} {vector} {opt. vector-index} {opt. count})**
    Sets contents in a table from a Scheme vector, starting at {table-index}. Copies the entire
    vector, or from {vector-index} for a total of {count} points if given. If table is not large
    enough, copies whatever fits. Returns a new vector of the contents copied.
    Alias: **tabsfv**

**(vector-set-from-table! {vector-name} {vector-index} {table-name} {opt. table-index} {opt. count})**
    Sets content in an existing vector from a Max table, starting at {vector-index}. Copies
    the entire table, or from {table-index} for a total of {count} points if given. If vector
    is not large enough, copies whatever fits. Returns a new vector of the contents copied.
    Alias: **vecsft**


Buffer I/O 
^^^^^^^^^^
We can write and read float data directly to and from named Max buffers, as well as query
for the existence and size of Max buffers. Max buffers are useful
for storing float data that we want to share between an **s4m** object
and the rest of Max (or another **s4m** object, they do not need to be used for audio samples
but can hold any collection of floats. Buffers are queried and locked in each function, so if you want
to read or write a lot of data, the vector conversions functions will be faster than a loop
of single point buffer operations.

Short form aliases exist for all functions and are identical except for the function name.


**(buffer? {buffer-name})**
    Returns #true if the symbol {buffer-name} corresponds to a named Max buffer.

**(buffer-size {buffer-name})**
    Returns integer length of buffer {buffer-name}. Error if not a valid buffer.
    Note that this is called buffer-size and not buffer-length to match Max naming, where
    length gives the length in seconds and size in samples.
    Alias: **bufsz**

**(buffer-ref {buffer-name} {opt. channel} {index})**
    Returns value at {index} in buffer {buffer-name}. If called with two arguments,
    channel defaults to zero. Alias: **bufr**

**(buffer-set! {buffer-name} {opt. channel} {index} {value})**
    Sets value at {index} in buffer {buffer-name} to {value}. If called with two 
    arguments, channel defaults to zero. Alias: **bufs**

**(buffer->vector {buffer-name} {opt. channel} {opt. index} {opt. count})**
    Returns a new Scheme vector from contents of a buffer, reading from {channel} or channel zero,
    starting at index 0 or {index} in the buffer, copying entire buffer or {count} points
    if optional {count} given. Alias **b->v**

**(buffer-set-from-vector! {buffer-name} {opt. buffer-channel} {opt. buffer-index} {vector} {opt. vector-index} {opt. count})**
    Sets contents in a buffer from a Scheme vector, using {buffer-channel} or channel zero and
    starting at {buffer-index} or index zero. Copies the entire vector, or from {vector-index} for 
    a total of {count} points if given. If buffer is not large enough, copies whatever fits. 
    Returns a new vector of the contents copied.
    Alias: **bufsfv**

Note: there is not yet a buffer version of table-set-from-vector!  

..
**(vector-set-from-buffer! {vector-name} {vector-index} {buffer-name} {opt. buffer-index} {opt. count})**
    Sets content in an existing vector from a Max buffer, starting at {vector-index}. Copies
    the entire buffer, or from {buffer-index} for a total of {count} points if given. If vector
    is not large enough, copies whatever fits. Returns a new vector of the contents copied.
    Alias: **bufsft**


Dictionary I/O & hash-tables
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
We can read and write key-value stores in Max dictionaries and S7 Scheme hash-tables,
which are unordered key-value hashmaps. When converting nested values in Max dictionaries to Scheme, 
arrays become Scheme vectors, and nested dictionaries become Scheme hash-tables. Only symbol, keyword, or
string keys are supported by the conversion functions.

In Lisp dialects that support keywords, the idiomatic practice is to generally use keywords for
hash-table keys. A keyword is a symbol that starts with a colon, and always evaluates to itself,
which means that they do not need to be quoted. 
This works well in Max too, Max will simply treat them as string keys that happen to start with a colon. 
This ensures you never get mixed up about whether a symbol in 
a Max message is supposed to be evaluated or not - if it's a keyword it won't matter if this 
gets eval'd by the interpreter. Keywords are recommended as dict and hash-table keys wherever possible.

Note that in S7, querying a hash-table for a non-existent key returns **#false**. In Max, there
is no boolean false value, and booleans are usually expressed as **0**. This means **0** is often 
a valid value to store, and thus getting **#false** back can be misleading. Instead of returning
a potentially valid value on a key error, the Max dict functions raise Scheme errors, which can
be caught to return a default value of your choosing. 


S7 hash-table and keyword examples
++++++++++++++++++++++++++++++++++++++++++++

.. code:: scm

    ;; a keyword always evaluates to itself
    (eval :foobar)
    s4m> :foobar
    
    (define key :foobar)
    (eval key)
    s4m> :foobar
   
    ;; which means quoting is unnecessary and does nothing
    (eval ':foobar)
    s4m> foobar
 
    ;; make a hashtable with two keyword slots
    (define my-hash (hash-table :a 1 :b 2))
    s4m> (hash-table :a 1 :b 2)
    
    ;; get value at :a
    (hash-table-ref my-hash :a)
    s4m> 1
    
    ;; applicative syntax of the same
    (my-hash :a)
    s4m> 1
    
    ;; a non-existing key from an S7 hash-table returns false (not an error!)
    (my-hash :z)
    s4m> #f

    ;; set a value in a hash-table, sets value and returns value
    (hash-table-set! my-hash :foobar 99)
    s4m> 99

    ;; applicative syntax of the same
    (set! (my-hash :foobar) 99)
    s4m> 99

    ;; make a nested hash-table
    (set! (my-hash :pets) (hash-table))
    s4m> (hash-table)
     
    ;; now we can set and get recursively using applicative syntax
    (set! (my-hash :pets :dog) 'Poppy-Poodle)
    s4m> Poppy-Poodle

    (eval my-hash)
    s4m> (hash-table :a 1 :b 2 :foobar 99 :pets (hash-table :dog Poppy-Poodle)) 
    
    (hash-table-ref my-hash :pets :dog)
    s4m> Poppy-Poodle
    (my-hash :pets :dog) 
    s4m> Poppy-poodle


     

Max Dictionary API
++++++++++++++++++++++++++++++
Examples below can be used with the tests-dict.maxpat test patcher in the patchers directory.

**(dict-ref {dict-name} {symbol|list key} )**
    Return value in dict {dict-name} at {key}, where dict-name is a symbol, and key can be either
    a list or symbol. If key is a list, recurses through the list. Raises an **'key-error'** error 
    if the key is invalid. Alias **dictr**

.. code:: scm

    ;; get a value from max dict named "test-dict", at key "a"
    (dict-ref 'test-dict 'a)
    s4m> 1

    ;; get a value that is a nested dict, becomes a hash-table
    (dict-ref 'test-dict 'b)
    s4m> (hash-table 'ba 2 'bb 3)
   
    ;; get value at key "ba" in nested dict at key "b" 
    (dict-ref 'test-dict (list 'b 'ba) )
    ;; same thing with alternative quoting syntax
    (dict-ref 'test-dict '(b ba))     
    s4m> 2

    ;; get the value at index 2 in the nested vector at key "c"
    (dict-ref 'test-dict '(c 2) )
    s4m> 33

    ;; out of range index 3 raises error
    (dict-ref 'test-dict '(c 3) )
    s4m> ERROR


**(dict-set! {dict-name} {symbol|list key} {value} )**
    Sets value in dict {dict-name} at {key}, where dict-name is a symbol, and key can be either
    a list or symbol. If key is a list, recurses through the list. Returns value set. 
    Raises error if key invalid. 
    Alias **dicts**


.. code:: scm

    ;; set a value in max dict named "test-dict", at key "z"
    (dict-set! 'test-dict 'z 44)
    s4m> 44

    ;; set a value that is a hash-table, becomes a nested dict
    (dict-set! 'test-dict 'y (hash-table :a 1 :b 2))
    s4m> (hash-table :a 1 :b 2)
   
    ;; set value at key "bc" in nested dict at key "b" 
    (dict-set! 'test-dict (list 'b 'bc) 111)
    s4m> 111

    ;; attempt set at invalid key list (there is no 'foo entry to recurse through)
    (dict-set! 'test-dict (list 'foo 'b) 99)
    s4m> ERROR


**(dict-replace! {dict-name} {symbol|list key} {value} )**
    Sets value in dict {dict-name} at {key}, where dict-name is a symbol, and key is a list
    of symbols or integers. If the list recursion hits an unused key, creates a nested dict
    for it, similar to the Max "replace" message to dicts. (Nested arrays do *not* get automatically
    created, also like the Max replace message). 
    Returns value set.  Raises error message if key or dict invalid. 
    Alias **dicts***


.. code:: scm

    ;; set a value that is a hash-table, creating an intermediate hash-table automatically
    (dict-replace! 'test-dict (list 'foo 'bar) 99)
    s4m> 99
   
