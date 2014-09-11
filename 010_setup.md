Setting Up a Bullet Application
===============================

Downloading
-----------

First, you need to download the Bullet Physics library. The latest release at the time of writing is [version 2.82][latest-release]. You can also get the source from the [SVN repository][svn-repo] (Bullet 2.x), or the [GitHub repository][git-repo] (Bullet 3.x in development). Extract the content. The resulted folder does not contain the compiled library, but only the source. In my case, the source folder is *D:\tuto\bullet-2.82-r2704*.

The next step is to compile Bullet from the source in order to use it in our C++ projects. In this documentation, we will only show you how to build Bullet for the MinGW (GCC) compiler (regular Unix Makefiles are similar), but Bullet also supports Visual Studio and Mac. For these platforms, see [the article on the official wiki][setup-wiki].

Preparing Makefiles using CMake
-------------------------------

We will use CMake to prepare the Makefiles in order to compile Bullet into C++ binaries. CMake is a building tool that makes the building process simpler in different operating systems. You can get it [here][cmake-download] on its official website, under the *Binary distributions* section.

Create a folder next to your Bullet source folder. For example, *D:\tuto\bullet-2.82-r2704-build*. This will contain the compiled files.
Open CMake-gui. Next to *Where is the source code:*, browse to the source folder. Next to *Where to build the binaries:*, browse to the new empty folder. This looks like this:

![Source and build directories in CMake][cmake-directories]

After that, click on *Configure*. CMake will ask to choose the *Generator* to use. In our case, we will select *MinGW Makefiles*. Click on *Finish*. CMake will some process some configuration and display the line *Configuring done* at the end of the bottom text box.

If you get the error "Could NOT find GLUT (missing:  GLUT_INCLUDE_DIR)", see the section below.

Once you are ready, click on *Generate*.

Compiling with MinGW
--------------------

But before continuing, make sure you have [MinGW][mingw] installed in your computer (*Download Installer* button at the top right on the home page).

Then open your terminal (On Windows: Windows+R then type *cmd*), cd to your build folder (*cd D:\tuto\bullet-2.82-r2704-build*), and execute the make command (*mingw32-make.exe*). It will compile the Bullet library files for several minutes.

Once it is done, the generated library files can be found at *bullet-2.82-r2704-build\lib*.

### Could NOT find GLUT (missing:  GLUT_INCLUDE_DIR) ###

First, note that Glut is only used in the demo executables of Bullet. The library itself does not use Glut. Thus, if you do not need to compile the demos you can just uncheck the option BUILD_DEMOS and ignore the message.

Otherwise, if you get this error and you want the demos, open CMakeLists.txt in the source folder (*D:\tuto\bullet-2.82-r2704\CMakeLists.txt*) and add at the end:

	#fix for glut not found
	INCLUDE_DIRECTORIES(${BULLET_PHYSICS_SOURCE_DIR}/Glut)
	SET(GLUT_glut_LIBRARY ${BULLET_PHYSICS_SOURCE_DIR}/Glut/glut32.lib)

These lines will force CMake to use the Glut files included in the Bullet source folder. The message in CMake will not disappear, but this should fix the problem at compilation time. 

Creating the project
--------------------

Here are the instructions to create a C++ project with Bullet with the IDEs [Eclipse CDT][eclipse-cdt] and [Code::Blocks][code-blocks]. The steps are similar for other IDEs.

### With Eclipse CDT

* Create a new *C++ Project*.
	* As *Project Type*, under *Executable*, select *Empty Project*.
	* Under *Toolchains*, select *MinGW GCC*

If your toolchain is not listed, uncheck *Show project types and toolchains only if they are supported on the platform*.

The next step is to setup Bullet into the project.

* Right-click the project name in the *Project Explorer* and go to *Properties*, then *C/C++ Build*, *Settings*, and the *Tool Settings* tab.

	* **Include directory:** Under *GCC C++ Compiler*, select *Includes*, then click on the *Add...* icon next to *Include paths (-I)*. Type or browse to the location of your *&lt;bullet source>/src* directory (e.g. *D:\tuto\bullet-2.82-r2704\src*).

	* Under *MinGW C++ Linker*, select *Libraries*.
		* **Libraries directory:** Next to *Library search path (-L)*, add the location of *&lt;bullet build>/lib* (*D:\tuto\bullet-2.82-r2704-build\lib*)
		* **Include directory:** Next to *Libraries (-l)*, **in the exact order** (since one library depends on the next one), add these libraries:
			* BulletDynamics
			* BulletCollision
			* LinearMath

![Include directory settings in Eclipse][eclipse-include]
![Libraries settings in Eclipse][eclipse-libraries]

### With Code::Blocks

* Create a new *Console Application* C++ project.

* Now, we will setup Bullet into the project. Right-click on the project name in *Project Explorer*, then open *Build Options...*  

	* Under the *Search directories* tab:
		* **Include directory:** Under *Compiler*, add the location of your *&lt;bullet source>/src* directory (e.g. *D:\tuto\bullet-2.82-r2704\src*)
		* **Libraries directory:** Under *Linker*, add the location of your *&lt;bullet build>/lib* directory (*D:\tuto\bullet-2.82-r2704-build\lib*)

	* **Libraries:** Under the *Linker settings* tab, next to the *Other linker options*, put in these libraries:

			-lBulletDynamics
			-lBulletCollision
			-lLinearMath

*Note: You can also add the libraries under *Link libraries*. These libraries must be in the exact order since one depends on the next one.*

![Libraries settings in Code::Blocks][code-blocks-libraries]
![...the include directory][code-blocks-include]
![...and the libraries directory][code-blocks-libraries-dir]


Now, you are ready to write your first Bullet application.


[cmake-directories]: img/setup/02_cmake_directories.png
[eclipse-libraries]: img/setup/08_eclipse_libraries_cut.png
[eclipse-include]: img/setup/09_eclipse_include_cut.png
[code-blocks-libraries]: img/setup/10_code_blocks_libraries.png
[code-blocks-include]: img/setup/11_code_blocks_include.png
[code-blocks-libraries-dir]: img/setup/12_code_blocks_libraries_dir.png

[latest-release]: http://code.google.com/p/bullet/downloads/list
[svn-repo]: https://code.google.com/p/bullet/source/checkout
[git-repo]: https://github.com/bulletphysics/bullet3
[cmake-download]: http://www.cmake.org/cmake/resources/software.html
[mingw]: http://www.mingw.org/
[setup-wiki]: http://bulletphysics.org/mediawiki-1.5.8/index.php/Creating_a_project_from_scratch
[eclipse-cdt]: http://www.eclipse.org/cdt/
[code-blocks]: http://www.codeblocks.org/


*Hello World* Application
-------------------------

The official Bullet Physics wiki has a very good [article][hello-wiki] for a *Hello World*.

[hello-wiki]: http://bulletphysics.org/mediawiki-1.5.8/index.php/Hello_World