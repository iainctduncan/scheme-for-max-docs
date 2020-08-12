Automated Tests
----------------

Regression tests for Scheme For Max are in the **tests** directory of the package.
You can look at these to see how various elements should work.
You can also run all the tests at once with the Ruby test runner.

Running the tests
-----------------
You will need to install the **max-tests** package, which can be downloaded as 
a package or cloned from source here. `https://github.com/Cycling74/max-test<https://github.com/Cycling74/max-test>`_

To use the automated test runner, you will also need Ruby installed, and the
ruby sqlite3 and rosc gems. Note that the test runner requires you to 
pass the location of your Max application. It will then run *every* test
it finds on your Max search path.


.. code:: bash
    $ gem install sqlite3 rosc 
    $ cd ~/Documents/Max\ 8/Packages/Scheme-For-Max/tests
    $ ruby test.rb "/Applications/"    

Making New Tests

- save as foobar.test.maxpat on the 
