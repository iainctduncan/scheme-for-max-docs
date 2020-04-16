Setup and Installation
=======================

Scheme for Max is currently available as a package for Max 8 on OSX, and as source code
that should compile for Max 7, and on Windows. 

The Scheme For Max package can be downloaded from the Releases tab on  `GitHub <https://github.com/iainctduncan/scheme-for-max.com>`_.

To install the package, download the archive, and uncompress it in Max packages directory. On OSX this
is normally in **~/Documents/Max 8/Packages**. Once you have uncompressed here, open
the package manager (File -> Show Package Manager), and select "Installed Packages" in the upper right hand corner.
You should now see an entry for Scheme For Max. Clicking **Launch** ought to open the Help tab.

In addition the Max external, the package contains:

* help/
   * The help file, **s4m.scm.maxhelp**
   * The scheme source files used in the help (**s4m_help_basics.scm** etc)

* extras/
   * The demo patcher(s), **s4m.demo.maxpat**, used in the demo videos

* patchers/
   * The repl patcher, **s4m.repl**
   * The main s4m scheme file, **scm4max.scm**   
   * Optional extra scheme files from s4m or base S7 that you may want 

**Important:** When you install a Max Package, the patchers, extras, and help directories are
automatically added to your Max file search path. If you move the scheme files from the patchers
directory, you will need to add their new location to the search path. In particular, the
**scm4max.scm** file is essential, it bootstraps the scheme side of scheme for max. If you decide
to alter it, you probably want to make a backup (though you can always get it again from GitHub too).

Providing the above are all where they should be, you should be able run the help file
and try out the features of Scheme for Max. 


