Building From Source
--------------------

To build from your source, you will need to have installed the `Max 8 SDK <https://cycling74.com/downloads/sdk>`_.
I believe s4m should compile fine with the Max 7 SDK as well, but this has not been verified. If you have 
not previously compiled a Max external with the SDK, I recommend you start by compiling some of the samples that
come with the SDK, at least until you get XCode building a dummy external. It should be theoretically 
be possible to build without the full XCode IDE, but I haven't found good enough instructions to make this
work, so I'm just using XCode as a (giant) compiler. If you know how, please let me know!

The sample projects come with project files for XCode and VisualStudio. I'll be honest, I know just
enough about XCode to be dangerous, so the instructions below are likely not optimal. I have
altered a sample XCode project file from the samples in the SDK, with my alterations detailed heere.

Note than on XCode at least,
these project files assume that the SDK files are in a specific place relative to your code. So you should 
clone the repository such that you have the following path layout, where the SDK path matches whatever version you
download, and **scheme4max** is the top level of the Git repository.

**~/Documents/Max 8/Packages/max-sdk-8.0.3/source/scheme4max**

So for me, the following works:

**$ cd ~/Documents/Max\ 8/Packages/max-sdk-8.0.3/source**

**$ git clone https://github.com/iainctduncan/scheme-for-max.git**


In the cloned repository, you should see a source directory, **s4m.scm**, with the following:

* **s7.c** - the main S7 C file (from S7)
* **s7.h** - the S7 header file (from S7)
* **mus-config.h** - an empty file required to compile S7
* **scm/** - directory with various scheme files.  
* **build_s7.sh** - a bash helper for building S7
* **s4m.scm.xcodeproj** - the XCode project file

Someone who knows XCode could tell me how to set it up to build S7 properly as a dependency, 
but as I don't, I'm just building the S7 object file and then linking it. build_s7.sh does this too.

**$ gcc -c s7.c**

If it worked, you should now have **s7.o**. 
Now we can build from XCode. Open the xcode project file; and below are the XCode settings I 
use.

We need to edit the linker flags. Select **s4m.scm** in the left hand toolbar, and look for the 
**Linking** section in the main window, with an entry called **Other Linker Flags**. 
Double click this to add Linker Flags. My linker flags are the following:

* -framework MaxAudioAPI
* -framework JitterAPI
* ${C74_SYM_LINKER_FLAGS}
* s7.o
* -L.
* -ldl
* -lm

With these in place, we should be able to build. Building will generate lot of warnings
about S7 pointers. I need to figure out how to turn those off, but they are harmless.
A successful build should build a new version of **s4m.scm.mxo**, by default in **Packages/max-sdk-8.x.x/externals**.
If you want to change where this goes you can do so, it's somewhere in the XCode config labrynth. 

To make the development workflow better, under **Product -> Scheme** you should see **max-external**. 
Choose **Edit Scheme** and the **Run** submenu to setup up whether Max launches automatically 
on compile. Select your Max.app as the executable to run. This will relaunch Max whenever you compile,
so you don't have to close and open Max to load your new external whenever you make changes.

If you have previously installed the binary package, you may need to move it out of your Max
package path to make sure you get the right external loading. The Python script **make-release.py**
automates bundling up a build, putting the tarball in the right place, and moving my SDK package
out of view so that I can test a new build. **uninstall.py** just puts things back. You can 
alter these for your system if they seem useful.

If you want to experiment further with S7, you can download the S7 source directly from 
CCRMA. It comes with a lot of examples of using S7 with C. If you want to add features
to Scheme-For-Max, you'll probably want to do this. But all we need for simple compilation
is **s7.c**, **s7.h**, and an empty **mus-config.h**.

The C Code is ugly right now, I know. We'll be refatoring and cleaning it up after I get
the Max automated test package working nicely, promise!

I will happily improve these instructions with feedback, and add a section on VisualStudio if someone
can help there. If you get a build going off the instructions, please let me know on the Google
Group so I know someone has managed. 




