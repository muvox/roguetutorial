# roguetutorial
Tutorial for setting up CLion, CMake, libtcod and SDL2


I'm writing this tutorial in hopes that it gives a simple(as possible) and a straightforward way to 
setup a CLion project with SDL2 and libtcod libraries.
For myself it took better part of a week to get all this setup, working and running. My Clion version 
at the time of writing was 2019.1.4.

I'm in no way a professional on these matters, it almost feels like I accidentally got this working. 
o to give back to the RogueLikeDev community I documented my steps and made this tutorial.

#Links
  - MinGW-w64 - https://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win32/Personal%20Builds/mingw-builds/installer/mingw-w64-install.exe/download
  - CLion - https://www.jetbrains.com/clion/download/#section=windows
  - lbitcod 1.12.3 x86_64 - mingw - https://github.com/libtcod/libtcod/releases/download/1.12.3/libtcod-1.12.3-x86_64-mingw.zip
  - SDL2 2.0.9 x64 - https://www.libsdl.org/release/SDL2-2.0.9-win32-x64.zip
  - FindSDL2.cmake - https://github.com/tcbrindle/sdl2-cmake-scripts/blob/master/FindSDL2.cmake

#Setup
1. First thing you should do i install mingw-w64. I'm using Windows 10-64 so I installed mingw-w64.
        In the installer I chose:
          - version as 8.1.0
          - Architecture x86_64
          - Threads win32
          - Exception seh
          - Build revision 0

  Sometimes the installer wants you to place it under C:\Program Files\mingw etc. but i placed it in 
  
    C:\MinGW-W64
    
  The folder contains a mingw-w64.bat, a link to mingw website, uninstaller and a folder called mingw64.

2. ***Warning! Messing with env tables might cause some programs to not run, so don't go all Rambo on this part***
   Set mingw to PATH:
   
   Run (on you keyboard Win+R) SystemPropertiesAdvanced. This should open up System Properties window.

   Click Environment Variables. And under System variables look for entry that's Variable is Path. Click it, and press Edit.. 
   and New, then Browse.
   Now you should point to the folder called bin inside mingw64 folder. Mine looks like this: C:\MinGW-W64\mingw64\bin .
   Then slap them ok buttons until the windows have disappeared.
        
   To test if this is set correctly, open command prompt (Win+R, type cmd) and write g++ --version . This should give 
   you version information on g++, the c++ compiler.

3. Next step was downloading and installing CLion. It should be pretty straightforward to download, install and setup. 
   Checking compiler settings, it should recognize mingw and all its required executables.

4. Download libraries and FindSDL2.cmake file.

   Once you've finished downloading the SDL2 and libtcod packages we need to set them up so that they are easily accessible.
   Go ahead and make a folder tree like this C:\Dev\CPP\
   Inside this folder place the extracted SDL2 and libtcod folders. On my system the CPP 
   folder contains libtcod-1.12.3-x86_64-mingw folder and a SDL2-2.0.9 folder.
   Next you should make a CMake folder, and inside that a Modules folder. Example: C:\Dev\CPP\CMake\Modules
   Now download the FindSDL2.cmake file, and place it in this folder. This file is used to make CMake find the sdl2 library.

5. Lets create a project in CLion.
   In the New Project window, choose the C++ Executable option. I left my language standard as C++14.
   Set the location of your project somewhere convenient for you, I used C:/Dev/Projects/nameOfYourProject
   To make sure that the compiler is setup correctly, you should try running the basic hello world 
   program that clion generates for you (Shift+F10).
   This should show you text "Hello, World!" in the clions run window.
   ***This is important so that cmake makes a build directory (in my case it was cmake-build-debug)***

6. Lets add some libraries.
    Inside your project folder you should make lib, include and src folders. 
    Move the main.cpp to the src folder inside CLion. CLion should automatically make this change in the cmake file.
    Next up the libtcod. Inside the extracted libtcod folder 
    (mine was: C:\Dev\CPP\libtcod-1.12.3-x86_64-mingw) there should be a include folder. 
    Select all, and copy them to your projects include folder.
    Then inside libtcod folder you should see a libtcod.a file. Copy this to your projects lib folder. 
    SDL2 doesn't need the same treatment because of the FindSDL2.cmake file.
    Next we need couple of .dll's and a .png file (libtcod uses it) to make the executable work when we finally run it.
    From the libtcod folder copy the libtcod.dll, SDL2.dll and terminal.png files, and place them in the 
    build directory of your project. If you built the basic hello world program, you project folder
    should have a cmake-build-debug folder. Place the three files there.

7. Next we tackle the infamous CMakeLists.txt. Under the "set(CMAKE_CZZ_STANDARD 14)" line, I added the following:

          set(CMAKE_MODULE_PATH "C:/Dev/CPP/CMake/Modules")
    this specifies the folder where cmake should look for the FindSDL2.cmake.
  
          set(SDL2_PATH "C:/Dev/CPP/SDL2-2.0.9/x86_64-w64-mingw32")
    this one tells cmake where the SDL2 itself is
  
          find_package(SDL2 REQUIRED)
    With this we can link the SDL library to our project.
    
          include_directories(${SDL2_INCLUDE_DIR}) 
    I believe SDL2_INCLUDE_DIR is a variable passed to us by the FindSDL2.cmake. With this we tell the cmake where SDL's own include directory is.
    
          include_directories(${PROJECT_SOURCE_DIR}/include)
    This one tells cmake where our own include directory is
    
          link_directories(${PROJECT_SOURCE_DIR}/lib)
    This one tells cmake where to find library files (.lib and .a (maybe just libname without extension on linux))
    
          file(GLOB TCODINC ${PROJECT_SOURCE_DIR}/include/*.h ${PROJECT_SOURCE_DIR}/include/*.hpp)
    Here we tell cmake to find all .h and .hpp files for libtcod (Hence the TCODINC variable name).
    
          set(SOURCE_FILES src/main.cpp ${TCODINC} )
    Here we set a variable for our source files + TCODINC header files.

    Next is the line that was already there, edit it like this:
    
          add_executable(YourProjectName ${SOURCE_FILES} )

    And finally the last line:
    
          target_link_libraries(${CMAKE_PROJECT_NAME} ${SDL2_LIBRARY} tcod)
    Here we tell to link given variables.

    CLion should tell you that you need to reload for changes to apply, press Reload Changes.

8.  Lets make a quick and simple roguelike:
    In main.cpp:
    
          #include "libtcod.h"

          int main() {
            int playerx=40,playery=25;
            TCODConsole::initRoot(80,50,"libtcod C++ tutorial",false);
            while ( !TCODConsole::isWindowClosed() ) {
              TCOD_key_t key;
              TCODSystem::checkForEvent(TCOD_EVENT_KEY_PRESS,&key,NULL);
              switch(key.vk) {
                case TCODK_UP : playery--; break;
                case TCODK_DOWN : playery++; break;
                case TCODK_LEFT : playerx--; break;
                case TCODK_RIGHT : playerx++; break;
                default:break;
                }
              TCODConsole::root->clear();
              TCODConsole::root->putChar(playerx,playery,'@');
              TCODConsole::flush();
            }
              return 0;
          }

  Then slap that sweet Shift+F10 to build and run and you should get a small window with our favorite little character. 
  You might notice from the code above that we can actually move the dude with arrow keys.

# Final Words

  As I said earlier, I'm no professional or master at these things. I made this so that someone who happens 
  to use the same tools as I, may begin his journey to developing roguelikes with C++ a bit faster.

  If you encounter any problems, you can hit me a dm via reddit https://www.reddit.com/user/muvoksi 
  I might be able to help but no promises. I'm still in the beginner phases of c++.

  I might update this same tutorial to work on Linux (i use debian for other programming stuff) but that is still TBA.
