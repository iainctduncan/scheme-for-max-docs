Algorithmic Processes and Live-Coding
----------------------------------------------------------
We can use the **delay** and **listen** functions to create
sequencers and repeating algorithmic processes, and interact
with these in real-time by changing variables or redefining functions.
This section provides some helpful examples and tips. This document
assumes familiarity with lambda functions and let blocks, though
one could certainly use the examples here to learn them.

This kind of algorithmic composition is explored in detail in the (highly 
recommended!) book **Notes from the Metalevel**, by Dr Heinrich Taube. 
The book uses S7 Scheme with the Common Music runtime, but examples are easily adapted to 
Scheme For Max.


Dynamically Redefining Scheduled Functions 
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
We can use the interpreter to redefine scheduled functions on the fly, but this
requires some care to get the behaviour you are expecting. When we register
a scheduled function or a listener function, we are registering the *function*,
not the *symbol for the function*. This means that if a function is registered in the
scheduler (whether as a listener or using delay), redefining the function 
attached to the symbol for the function's *name* has no effect on the function called 
when the timer callback fires.  
If we want to ensure that redefining the value of a given symbol changes what an 
already registered listener does, we need to make a wrapper function that evaluates 
the global symbol on each pass. This is best illustrated with examples:


Example of function registration that is unaffected by changes to the
symbol used to define the function:

.. code:: scm

    ;; define a function that posts :foobar to the console
    (define (my-listener) (post :foobar))
    
    ;; register it to get evaluated every second
    (listen-ms 1000 my-listener)

    ;; now we see :foobar posted every second 

    ;; redefine my-listener
    (define (my-listener) (post :foobaz) )

    ;; no change to what is getting posted, because the old 
    ;; my-listener function is still registered!

    ;; register the new function, replacing the function registered for listen-ms
    (listen-ms 1000 my-listener)

    ;; now we get :foobaz posted 


Example of function registration that ensures changing the listener
will have an immediate effect:

.. code:: scm

    ;; define a function that posts :foobar to the console
    (define (my-listener) (post :foobar))
    
    ;; register a lambda function that evaluates whatever is currently
    ;; attached to the symbol 'my-listener 
    (listen-ms 1000 (lambda () (eval (list 'my-listener))))
    ;; same, different syntax
    (listen-ms 1000 (lambda () (eval '(my-listener))))

    ;; every second we are getting :foobar posted now

    ;; redefine my-listener
    (define (my-listener) (post :foobaz) )

    ;; now foobaz is getting posted


The same scoping rules apply for variables that your function uses. If we want changes to 
a top level variable to immediate be reflected in our registered functions,
we have those functions use values looked up from global symbols:

.. code:: scm

    ;; define and register function that outputs the note-number currently at 'note-num
    ;; this will print an error message if note-num does not exist
    (define note-num 60)
    (define (my-listener) (out 0 note-num))

    (listen-ms 1000 my-listener)
    ;; function is outputing 60 

    (set! note-num 72)
    ;; function is now outputing 72 


Capturing variables with lexical closures 
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
To capture the value of a global (or outer) variable, but keep the listener function using
it regardless of any subsequent changes to the global variable, we need to use a lexical
closure. There are several ways of doing this, one is to use a **let** block.  

.. code:: scm

    ;; assume we have a global var, note-num, that we manipulate somehow
    (define note-num 60)
    
    (define my-listener
      (let ((captured-note-num note-num))
        (lambda () (out 0 captured-note-num))))
    
    (listen-ms 1000 my-listener)

    ;; 60 now coming out every second

    (set! note-num 72)

    ;; no change, captured-note-num is not subject to changes to symbol note-num


This is a very common pattern in Lisp languages, often refered to as "let over lambda".
A variable is declared in the let definitions and its value is read from the global
when the let block runs. The let block then returns a function that uses the symbol
local to the let block, thus the function returned takes the let environment with it.

Another way of doing this is to use function arguments, and capture
those with wrapper lambdas. In this case, we would define a function we want to have
run at the scheduled time, with some local arguments, and register a wrapper function
that takes no arguments as the actualy delayed callback:


.. code:: scm

    ;; assume we have a global var, note-num, that we manipulate somehow
    (define note-num 60)
    
    (define (my-delay-fun note-num-param)
      (out 0 note-num-param))

    (define my-delay-fun-callback
      (let ((captured-note-num note-num))
        (lambda ()
          (my-delay-fun captured-note-num))))
    
    (listen-ms 1000 my-delay-fun-callback)

    ;; 60 now coming out every second
    (set! note-num 72)

    ;; no change, the local note-num-param captured the value of 
    ;; note-num at the time of the lambda



Below is an example of reading from both mutable top level variables 
and captured variables: 

    
.. code:: scm

    ;; top level vars for note-num and velocity
    (define note-num 60)
    (define velocity 90)
    
    (define my-listener
      (let ((captured-note-num note-num))
        ;; captured-note-num is in the let, but velocity is global
        (lambda () (out 0 (list captured-note-num velocity)))))
    
    (listen-ms 1000 my-listener)

    ;; 60 90 now coming out every second

    (set! note-num 72)
    (set! velocity 120)

    ;; 60 120 coming out - velocity changes, but note-num does not


Temporal Recursion / Recursive Scheduling 
++++++++++++++++++++++++++++++++++++++++++++

It is safe to have a scheduled function re-schedule another function, sometimes called
"temporal recursion" in the algorithmic composition literature. The function
definition uses standard Lisp recursion:

.. code:: scm

    ;; my self-scheduled callback
    (define (my-callback) 
      (post "executing!") 
      (delay-t 480 my-callback))
    
    ;; start it up immediately, it will then repeat every 480 ticks
    (delay 0 my-callback)

These require some management if you want to be able to cancel their execution (without just
resetting the intrepreter!) One method is to write to a global (or other scope) variable that
 stores the last registered callback handler and can thus be used to stop playback.
 
.. code:: scm

    ;; a variable to hold the handle, outside of the scope of the callback
    (define cb-handle #f)

    (define (my-callback) 
      (post "executing!") 
      (set! cb-handle (delay-t 480 my-callback)))

    ;; start it up, and capture the callback    
    (set! cb-handle (delay 0 my-callback))

    ;; at some point, we can stop playback by cancelling the callback
    (cancel-delay cb-handle)


If we use the transport aware delay functions with quantize (**delay-tq**) this
can be used to spawn repeating processes synchronized with musical time and
kept in sync with the transport tempo. You may want to make your own version
of transport control functions that manage callback cancellation and then
stop the transport using the aforementioned transport functions.


Recurive Scheduling and Lexical Closures 
+++++++++++++++++++++++++++++++++++++++++

By combining lexical clojures with self-scheduling delay calls, we can
make functions that change the arguments for their next iteration.  
Below is an example of a function that counts down 4 repetitions.

    
.. code:: scm

    (define global-num-reps 3)

    (define (my-fun rep-num)
      (post "rep:" rep-num)
      ;; do stuff for this iteration

      ;; make arguments for next iteration
      (let ((next-rep-num (- rep-num 1)))
        
        ;; in let body we schedule next iteration with the captured arg
        (delay 1000 (lambda ()
          (my-fun next-rep-num)))))
      
    ;; now we need to kick it off with a lambda capturing the first arg
    (delay 1000 (lambda ()(my-fun global-num-reps)))
          
    
The above will just count forever. Below is the same improved so that
the process stops after the count reaches the last rep:

.. code:: scm

    (define global-num-reps 3)
    
    (define (my-fun rep-num)
      (post "rep:" rep-num)
      ;; do stuff for this iteration
    
      ;; make arguments for next iteration
      (let ((next-rep-num (- rep-num 1)))
    
        ;; schedule next iteration only if next-rep-num is greater than zero
        ;; otherwise, post :done
        (if (and playing (> next-rep-num 0))
          (delay 1000 (lambda () (my-fun next-rep-num)))
          (post :DONE))))

    ;; start it up
    (delay 1000 (lambda ()(my-fun global-num-reps))) 
    
         

Controlling Recursive Processes
++++++++++++++++++++++++++++++++

We might need to be able to stop a process early, so we can also check
a global to see if we should still be playing before scheduling the next
iteration:

.. code:: scm

    ;; toggle for stopping playback
    (define playing #t)
    (define global-num-reps 100)
    
    (define (my-fun rep-num)
      (post "rep:" rep-num)
      ;; do stuff for this iteration
    
      ;; make arguments for next iteration
      (let ((next-rep-num (- rep-num 1)))
    
        ;; schedule next iteration only if next-rep-num is greater than zero
        ;; otherwise, post :done
        (if (and playing (> next-rep-num 0))
          (delay 1000 (lambda () (my-fun next-rep-num)))
          (post :DONE))))

    ;; start it up
    (delay 1000 (lambda ()(my-fun global-num-reps))) 
    ;; stop playback early
    (set! playing #f)



Finally, what if we want to be able to stop only certain processes? If
we capture the callback handle for each delay call in a top level
variable, we can cancel recursion for a process at anytime by cancelling that handle:

.. code:: scm

    ;; toggle for stopping playback
    (define playing #t)
    (define global-num-reps 100)
    (define process-handle #f)
    
    (define (my-fun rep-num)
      (post "rep:" rep-num)
      ;; do stuff for this iteration
    
      ;; make arguments for next iteration
      (let ((next-rep-num (- rep-num 1)))
    
        ;; schedule next iteration only if next-rep-num is greater than zero
        ;; otherwise, post :done
        (if (and playing (> next-rep-num 0))
          ;; schedule and save the handle to the global process-handle var 
          (set! process-handle (delay 1000 (lambda () (my-fun next-rep-num))))
          (post :DONE))))

    ;; start the process, saving the callback handle
    (set! process-handle (delay 1000 (lambda ()(my-fun global-num-reps))))

    ;; stop only this process 
    (cancel-delay process-handle)



