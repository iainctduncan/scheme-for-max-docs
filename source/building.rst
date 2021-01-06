Building From Source
========================================
Below are some (in progress) instructions for building from source. Help in improving this area
would be much appreciated, please get in touch if you want to build from source and are not managing.

Short version
--------------
I'm using the Max 8 SDK, though I think the Max 7 should work too. We are using the
"canonical" C API, not the newer C++ API, as it is much better documented at this point.

The repo includes **s7.c**, **s7.h**, and an empty file **mus-config.h**. You 
need **mus-config.h** to build **s7.o**, and then you need to link in **s7.o**. 
There is only one source file for the external, **s4m.c**.
Building it will build the Max external **s4m**. The other objects
are just standard Max patchers.

There is an XCode and a VisualStudio project file in the repo. The VS one works properly,
the XCode one is not set up right as I haven't figured out how to get S7 buidling
properly in XCode, so I'm just being stupid and compiling S7 to object code with GCC
and linking it in. (Pull requests or patches to fix the XCode file would be lovely!)

My linker flags: 

* -framework MaxAudioAPI
* -framework JitterAPI
* ${C74_SYM_LINKER_FLAGS}
* s7.o
* -L.
* -ldl
* -lm


Long version
-------------
To build from your source, you'll need to have installed the `Max 8 SDK <https://cycling74.com/downloads/sdk>`_.
I believe S4M should compile fine with the Max 7 SDK as well, but this has not been verified. If you have 
not previously compiled a Max external with the SDK, I recommend you start by compiling some of the samples that
come with the SDK, as the XCode and VisualStudio setup is finicky. It should theoretically 
be possible to build without the full XCode IDE, but I haven't managed to make this
work. If you know how, please let me know.

The SDK comes with sample projects with project files for XCode and VisualStudio. I'll be honest, I know just
enough about XCode to be dangerous, so the instructions below are not "the right way".  I 
altered a sample XCode project file from the samples in the SDK, and my alterations are detailed here.

Note than on XCode at least, these XCode project files from C74 assume that the SDK files are in a specific place
relative to your code. So you should clone the repository such that you have the following path layout, 
where the SDK path matches whatever version you download, and **Scheme4max** is the top level of the Git repository.

**~/Documents/Max 8/Packages/max-sdk-8.0.3/source/Scheme4max**

So for me, the following works:

**$ cd ~/Documents/Max\ 8/Packages/max-sdk-8.0.3/source**

**$ git clone https://github.com/iainctduncan/Scheme-for-max.git**


In the cloned repository, you should see a source directory, **s4m**, with the following:

* **s7.c** - the main S7 C file (from S7)
* **s7.h** - the S7 header file (from S7)
* **mus-config.h** - an empty file that is necessary to compile S7
* **scm/** - a directory with various Scheme files.  
* **build_s7.sh** - a bash helper for building S7
* **s4m.xcodeproj** - the XCode project file, tweaked from a C74 example

Someone who knows XCode could tell me how to set it up to build S7 properly as a dependency, 
but as I don't, I'm just building the S7 object file and then linking it. build_s7.sh does this too.

**$ gcc -c s7.c**

If it worked, you should now have **s7.o**. 
Now we can build from XCode. Let's open the xcode project file and verify the set up.

We need to edit the linker flags. Select **s4m** in the left hand toolbar, and look for the 
**Linking** section in the main window, with an entry called **Other Linker Flags**. 
Double click this to add Linker Flags. My linker flags are the following:

* -framework MaxAudioAPI
* -framework JitterAPI
* ${C74_SYM_LINKER_FLAGS}
* s7.o
* -L.
* -ldl
* -lm

The first three are for the C74 SDK, and the rest are linking in S7.
With these in place, we should be able to build. Building will generate *a lot* of warnings
about S7 pointers casts. I need to either figure out how to tell XCode it's ok or fix these, as I'm following examples
from the S7 FFI, but they are harmless right now.

A successful build should build a new version of the external, by default in **Packages/max-sdk-8.x.x/externals**.
If you want to change where this goes you can do so, it's somewhere in the XCode config labyrinth. 
Note that on Windows, you need to build the 32 and 64 bit versions separately by compiling with
different targets.

If you plan on hacking on Scheme for Max, you'll want to get Max launching automatically on compile. 
To make the development workflow better, under **Product -> Scheme** you should see **max-external**. 
Choose **Edit Scheme** and the **Run** submenu.
Select your Max.app as the executable to run. This will relaunch Max whenever you compile,
so you don't have to close and open Max to load your new external whenever you make changes.

If you have previously installed the binary package, you may need to move it out of your Max
package path to make sure you get the right external loading. The Python script **make-release.py**
automates bundling up a build, putting the tarball in the right place, and moving my SDK package
out of view so that I can test a new build. **uninstall.py** just puts things back. You can 
alter these for your system if they seem useful, but be warned, it makes file system changes!
This is *not* necessary for building, it's just a convenience tool to make testing releases 
more practical.

If you want to experiment further with S7, you can download the S7 source directly from 
CCRMA. It comes with a lot of examples of using S7 with C. If you want to add features
to Scheme For Max, you'll probably want to do this. But all we need for simple compilation
is **s7.c**, **s7.h**, and an empty **mus-config.h**.

I will happily improve these instructions with feedback, and add a section on VisualStudio if someone
can help there. If you get a build going off the instructions, please let me know on the Google
Group so I know someone has managed. If you need help, post there too.

There is a regression test suite in the patchers folder. Some of the stuff tested requires
delays to avoid interacting tests. You might not get the same results as I do. If they
ever pass, we are good. If they sometimes fail (or fail when you try to run them all) there
is a good chance it's just execution time differences on your system - you can try
running the subsuites separately, or even openin those and running the tests. I do make sure they
all pass on both OSX and Windows prior to releases. Hopefully they can eventually
be made robust enough to work everywhere. They allow complete regression testing from the click of one
button though, so that's something!





