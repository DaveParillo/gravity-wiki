
# GravitySetup #

This page details what you need to set up in Linux and Windows so that you can build Gravity for C++ and Java.

## Linux ##

The Linux setup is fairly straightforward.  You just need to install:

* Oracle's Java JDK
* swig
* maven
* bison/flex
* libboost-dev

The last four items above can probably be installed using the package installation system on your flavor of Linux (e.g. sudo apt-get install swig).  If a newer version of swig is needed than is provided by your package management system, you can download the needed version from http://www.swig.org/ and build it.

Installing Java may be a little more involved.  You can skip building Java with the --without-java option to the configure script.  
You can download an installer here: http://www.oracle.com/technetwork/java/javase/downloads/index.html.  The complete installation depends on your system.  Some info on installing on Ubuntu can be found here: http://askubuntu.com/questions/67909/how-do-i-install-oracle-jdk-6 or here: http://www.wikihow.com/Install-Oracle-Java-JDK-on-Ubuntu-Linux.

For Python support (currently only supported on Linux), there are a couple of additional dependencies (note that you can disable Python support with the --without-python option to the configure script):
* python-dev
  * This can be installed with your package management system
* python-protobuf   
  * This can be installed with your package management system or pip (e.g. sudo pip install protobuf)

You must also download and build the following:
* ZeroMQ
    http://download.zeromq.org/zeromq-3.2.3.tar.gz

    Extract in ~/gravity_deps/  Run ./configure, make,and make install

* Google Protocol Buffers
    https://github.com/google/protobuf/releases/download/v2.6.1/protobuf-2.6.1.tar.gz

    Extract in ~/gravity_deps/ Run ./configure, make, and make install
    
    Build the Java runtime library using Maven (see the readme in java\).
    NOTE: Maven does not use the standard proxy environment variables (http_proxy, etc) - you must configure Maven explicitly.  Directions for this can be found here: https://maven.apache.org/guides/mini/guide-proxies.html.
    
    sudo ldconfig

* Copy dependencies into gravity/deps
    * cp [HOME]/gravity_deps/protobuf-2.6.1/src/.libs/libproto* [HOME]/gravity/deps
    * cp [HOME]/gravity_deps/zeromq-3.2.3/src/.libs/libzmq.* [HOME]/gravity/deps
    * cp [HOME]/gravity_deps/zeromq-3.2.3/include/* [HOME]/gravity/deps
    * cp [HOME]/gravity_deps/protobuf-2.6.1/src/protoc [HOME]/gravity/deps
    * cp [HOME]/gravity_deps/protobuf-2.6.1/java/target/protobuf-2.6.1.jar [HOME]/gravity/deps/protobuf-java.jar
    * cp [HOME]/gravity_deps/protobuf-2.6.1/python/google [HOME]/gravity/deps


* Clone from the git repository (or otherwise acquire) the Gravity source tree
    cd to the gravity source directory (e.g "\home\gravity")
    
    ./configure
    --with-protobuf-libdir=[HOME]/gravity/deps
    --with-protobuf-incdir=[HOME]/gravity/deps
    --with-zeromq-libdir=[HOME]/gravity/deps
    --with-zeromq-incdir=[HOME]/gravity/deps
    --with-protoc=[HOME]/gravity/deps/protoc
    JAVAPROTOBUF_DIR=[HOME]/gravity/deps/protobuf-java.jar
    GUAVAJAR_DIR=[HOME]/gravity/src/api/MATLAB/guava-13.0.1.jar

    make

    make test-prep

    make test

    make distributable

* If needed for certain project, create or edit /etc/ld.so.conf.d/gravity.conf and add the following lines
    * /[HOME]/gravity/deps
    * /[HOME]/gravity/lib
    * /[HOME]/gravity/src/api/bin
    * /[HOME]/gravity/src/keyvalue_parser

    sudo ldconfig


## Windows 32/64 bit build ##
The tools below are required to build the 32/64  bit Gravity libraries.  Install them in the order listed.

* .NET Framework 4
    http://www.microsoft.com/en-us/download/details.aspx?id=17851

* Windows 7.1 SDK
    http://www.microsoft.com/en-us/download/details.aspx?id=8279

* Choose a Visual Studio version
    * VS 2010
        * Visual C++ 2010 Express.  Run windows update and install the recommended updates after this installation.
            * http://go.microsoft.com/?linkid=9709949
        * Visual C++ 2010 Express SP1
            * http://www.microsoft.com/en-us/download/details.aspx?id=23691
        * Visual C++ 2010 SP1 Compiler Update For Windows SDK 7.1
            * http://go.microsoft.com/fwlink/?LinkId=212355
    * VS 2012
        * Visual Studio 2012 Express for Windows Desktop.  Run windows update and install the recommended updates after this installation.
            * http://go.microsoft.com/?linkid=9816758

* Google Protocol Buffers
    * Download version 2.6.1 source and extract to local folder (e.g. C:\protobuf-2.6.1)
        * http://code.google.com/p/protobuf/downloads/detail?name=protobuf-2.6.1.tar.bz2&can=2&q=
    * Build libprotobuf libs (''note that switching between VS2010 & VS2012 builds will require repeating this step'')
      * Open project file is vsprojects directory (e.g. C:\protobuf-2.6.1\vsprojects\protobuf.sln) in Visual Studio 2012 Express
      * Click ‘OK’ when prompted to upgrade to VS2012.
      * If building for Visual Studio 2012
        * Right-click on the libprotobuf project in the “Solution Explorer” and select “Properties”.
        * Select Configuration Properties->General and set (if not already set) the platform toolset to “Visual Studio 2012” for both the Debug and Release configurations.
      * If building for Visual Studio 2010
        * Right-click on the libprotobuf project in the “Solution Explorer” and select “Properties”.
        * Select Configuration Properties->General and change the platform toolset to “Windows7.1SDK” for both the Debug and Release configurations.
      * If building 64-bit version
        * Open Configuration Manager
        * Select <New…> from Active Solution platform pull-down selector
        * For Type, select “x64”.
        * Copy Settings from “Win32”.
        * Select “Create new project platform” checkbox.
        * Click OK button
      * Select the desired build platform (Win32 or x64)
      * Build the Release configuration
      * If debug builds of Gravity are required, you also need to build the Debug configuration as follows:
        * Select the Debug configuration
        * Right-click on the libprotobuf project in the “Solution Explorer” and select “Properties”.
        * Select Configuration Properties->General and change “Target Name” from “$(ProjectName)” to “$(ProjectName)_d”
        * Build the Debug configuration
 
* Download protoc.exe (version 2.6.1) and place in protobuf folder (e.g. C:\protobuf-2.6.1)
      http://code.google.com/p/protobuf/downloads/detail?name=protoc-2.6.1-win32.zip&can=2&q=

  Add the protoc.exe location to your path.
  Set the environment variable **PROTOBUF_HOME** to Protobuf root (e.g. C:\protobuf-2.6.1)

* Java Development Kit (JDK) (32-bit or 64-bit depending on build requirements)
    http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html

  Download the most recent Java 7 JDK.

  After you download and install the package, you'll need to add the JDK's bin directory (e.g. C:\Program Files (x86)\Java\jdk1.7.0_xx\bin) to your path.

  Set/Modify the following environment variables:

  **JAVA_HOME** to JDK root (e.g. C:\Program Files (x86)\Java\jdk1.7.0_xx)
  Add %JAVA_HOME%\bin to system path

* swigwin
    http://www.swig.org/download.html

    Add the main swigwin directory (e.g. C:\swigwin-3.0.4) to your path 

* Windows Flex & Bison
    http://sourceforge.net/projects/winflexbison/

    Extract these files and add the installation directory (e.g. c:\win_flex_bison) to your path 

    Set the environment variable **LEX_CMD** to win_flex.exe 
    
    Set the environment variable **YACC_CMD** to win_bison.exe 

* Boost
    http://www.boost.org/

    Download the latest version of the Boost C++ library 

    Set the environment variable **BOOST_HOME** to the Boost root (e.g. C:\boost_1_57_0)

* Maven
    http://maven.apache.org/download.cgi

    Add the Maven bin directory to your path

* Guava
    http://search.maven.org/remotecontent?filepath=com/google/guava/guava/13.0.1/guava-13.0.1.jar

    Set GUAVAJAR_DIR environment variable to full path (including filename) of this file.

* ZeroMQ
  * 32-bit: http://miru.hk/archive/ZeroMQ-3.2.3~miru2.3-x86.exe
  * 64-bit: http://miru.hk/archive/ZeroMQ-3.2.3~miru2.3-x64.exe
  Download and run the installer for the appropriate version of ZeroMQ (Get both if intending both 32-bit & 64-bit Gravity builds)

  Set the environment variable **ZMQ_HOME** to ZeroMQ root (e.g. C:\Program Files (x86)\ZeroMQ 3.2.3). ''Note that you’ll need to change this to toggle between the 32-bit and 64-bit versions''.

* PThreads

  Download (and extract to a local folder) the PThreads libraries from ftp://sourceware.org/pub/pthreads-win32/pthreads-w32-2-9-1-release.zip

  Set environment variable **PTHREADS_HOME** to PThreads pre built root (e.g. C:\pthreads-w32-2-9-1-release\Pre-built.2\)


### Building Gravity ###
* Clone from the git repository (or otherwise acquire) the Gravity source tree
* Set the environment variable **GRAVITY_HOME** to Gravity root directory (e.g. C:\gravity)
* Update environmental variables as required for the desired build configuration
  * Specifically, if changing between 32-bit and 64-bit builds, the following environment variables will need to be set appropriately:
    * ZMQ_HOME
    * JAVA_HOME
* Configure Dependencies
  * Open a command prompt (as Administrator)
  * Navigate to the GRAVITY_HOME directory
  * Run the configDeps batch file
    * This is will prompt you to specify the build configuration (VS2010/VS2012, 32-bit/64-bit)
    * Enter the number for the desired configuration
    * This will create a dependency directory for use in later stages of the build
* Open Gravity solution file in GRAVITY_HOME\build\msvs\Gravity.sln in Visual Studio 2012 Express
* Select the desired build configuration (Release2010, Release2012, Debug2010, Debug2012) and platform  (x86, x64)
* Build the solution

The Gravity distribution will be located in %GRAVITY_HOME%\<Platform>\<Configuration> and will include bin, lib, deps, and include directories.

Add the bin and deps directories to your system path.

## Windows Notes ##
* Visual Studio 2012 is required to open the .sln files to make any changes to the solutions.  This can be installed alongside the 7.1 SDK
* If you also have VS2010 installed and you update to service pack 1 your 64bit compilers will un-install.  
  See this article from Microsoft on this issue: http://support.microsoft.com/?kbid=2519277

## Using Gravity in MATLAB ##

### Linux ###

#### MATLAB's Java version ####

The Gravity distribution is current built with Java 6. Recent versions of MATLAB ship with Java 6 and as such should be compatible. If you build Gravity with a later version of Java (e.g. Java 7) you will need to configure MATLAB to use that same version. This could potentially cause stability issues with MATLAB so proceed with caution. In order to configure MATLAB's java, set a MATLAB_JAVA environment variable to point to your jre directory. For example (on linux):

    export MATLAB_JAVA=/usr/lib/jvm/java-7-oracle/jre

Your path may differ.

#### Configure MATLAB's static java class path to include Gravity ####

Follow the MATLAB instructions for your specific version of MATLAB. A couple examples follow:

##### MATLAB R2010b #####
Edit the classpath.txt file in <MATLAB_INSTALL_DIR>/toolbox/local where <MATLAB_INSTALL_DIR> is the location of the MATLAB installation (e.g. /usr/local/MATLAB/R2010b on linux). Add the following lines to the end of the file:

    <GRAVITY_HOME>/lib/gravity.jar
    <GRAVITY_HOME>/deps/protobuf-java.jar
    <GRAVITY_HOME>/lib/MATLAB/guava-13.0.1.jar
    <GRAVITY_HOME>/lib/MATLAB/MATLABGravitySubscriber.jar

where <GRAVITY_HOME> is whereever you installed your Gravity distribution.

##### MATLAB R2012b #####
For R2012b, you need to add the following lines to <HOME>/.matlab/R2012b/javaclasspath.txt (where <HOME> is your user's home directory):

    <GRAVITY_HOME>/lib/gravity.jar
    <GRAVITY_HOME>/deps/protobuf-java.jar
    <GRAVITY_HOME>/lib/MATLAB/guava-13.0.1.jar
    <GRAVITY_HOME>/lib/MATLAB/MATLABGravitySubscriber.jar

where <GRAVITY_HOME> is whereever you installed your Gravity distribution.

#### Configure MATLAB's library path ####

Follow the MATLAB instructions for your specific version of MATLAB. For example:

Add the following line to the <MATLAB_HOME>/toolbox/local/librarypath.txt (where <MATLAB_HOME> is the MATLAB installation directory):

    <GRAVITY_HOME>/lib
    <GRAVITY_HOME>/deps

where <GRAVITY_HOME> is whereever you installed your Gravity distribution.  Depending on where the protobuf and zmq compiled libraries are installed, you may also need to add /usr/local/lib here.  

This is the mechanism for at least R2010b through R2012b

#### Configure MATLAB to use the system gcc libs ####

MATLAB ships with its own set of runtime GCC libraries. For compatibility with Gravity we'll want MATLAB to use the system libraries consistent with Gravity. In order to do this, we just need to move the MATLAB libs so that MATLAB will use the system libs. For example:


    > cd /usr/local/MATLAB/R2010b/sys/os/glnxa64
    > sudo mkdir old
    > sudo mv libstdc++.* libgcc_s* old

#### Set system library path to include Gravity ####
Create this file (you'll need root privileges): /etc/ld.so.conf.d/gravity.conf
Edit that file to include lines for the path to the Gravity lib and deps directory:

    <GRAVITY_HOME>/lib
    <GRAVITY_HOME>/deps

Then, to refresh the system libs:

    > sudo ldconfig


### Windows ###

#### Configure MATLAB's static java class path to include Gravity ####
Create or edit "javaclasspath.txt" inside the C:\Users\<USER>\AppData\Roaming\MathWorks\MATLAB\<MATLAB_VERSION> directory. Add the following lines: 

    <GRAVITY_HOME>\lib\gravity.jar
    <GRAVITY_HOME>\lib\MATLAB\guava-13.0.1.jar
    <GRAVITY_HOME>\lib\MATLAB\MATLABGravitySubscriber.jar
    <GRAVITY_HOME>\deps\protobuf-java.jar

#### Configure MATLAB's library path ####
Edit the "librarypath.txt" file located in C:\Program Files\MATLAB\<MATLAB_VERSION>\toolbox\local directory. Add the following lines:

    <GRAVITY_HOME>/bin
    <GRAVITY_HOME>/deps

#### Configure system path ####
If you haven't done so already, add the bin and deps directories to your system path.

#### Test Gravity ####
See the MATLAB examples distributed with Gravity for a more complete test, but a simple test to ensure the setup is complete would be to attempt to create a Gravity Node within MATLAB:

    >> path(path, '<GRAVITY_HOME>/include/MATLAB')   % replace GRAVITY_HOME to point to your Gravity installation
    >> gravityNode = GravityNode('TEST');

If this executes successfully you are ready to run the more complete tests.
