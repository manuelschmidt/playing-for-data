Playing for Data
===============================================================================

Requirements:
- CMake (for making OpenEXR and zlib)
- Visual Studio (for compiling renderdoc) - tested on VS 2013
- Python - tested with Anaconda
- MATLAB (for annotating frames) - tested on ML 2013a


Let <PFD_DIR> be the Playing for Data directory with the following structure:
	- renderdoc  | Graphic Debugger - dump frames to disk
	- scripts    | Python scripts for extracting data from games using renderdoc
	- README.md  | this file.


Preparation (OpenEXR):
-------------------------------------------------------------------------------
1. Use CMake to build zlib <PFD_DIR>/renderdoc/renderdoc/3rdparty/zlib
2. Use CMake to build IlmBase (located at <PFD_DIR>/renderdoc/renderdoc/3rdparty/openexr/IlmBase)
   Disable the NAMESPACE_VERSIONING option.
   Set CMAKE_INSTALL_PREFIX to your preferred directory.   
3. Compile IlmBase by building the INSTALL target.
4. Use CMake to build OpenEXR (located at <PFD_DIR>/renderdoc/renderdoc/3rdparty/openexr/OpenEXR)
   Make sure that ILMBASE_PACKAGE_PREFIX in CMake is set to the install directory of IlmBase 
   (the one you specified as CMAKE_INSTALL_PREFIX in the 2nd step). 
   Disable the NAMESPACE_VERSIONING option.
   If you built zlib in a separate build directory, copy the zconf.h from that directory to the ZLIB_INCLUDE_DIR
5. Compile OpenEXR. Make sure that the ilmbase libraries (Half, IlmThread) are on your library search path 
   (for setting the path under windows see Preparation 2. below), 
   otherwise compiling the IlmImf project may fail with some cryptic error (Visual Studio returns '"cmd.exe" exited with code -1073741515').
   

Preparation (renderdoc):
-------------------------------------------------------------------------------
1. Compile the complete renderdoc solution.
2. Make sure the OpenEXR dlls are in a directory pointed to by your path variable.
   (Setting the path under windows is done at
    Control Panel/System and Security/System/Advanced System Settings/Environment Variables)



Setup for capturing:
-------------------------------------------------------------------------------
1. Run renderdocui.exe with admin privileges.
2. Specify the path where dumped frames are stored: 'Tools/Options/Directory for temporary capture files'
3. Enable global process hooking in 'Tools/Options/Allow global process hooking' (May not be needed by all games)
4. Open the Capture Executable Tab.
5. Point the 'Executable Path' to your game
6. If you are using the global hook, the other settings don't matter as they are set in a different process.
   If you need to modify them, check <PFD_DIR>/renderdoc/renderdoccmd/renderdoccmd_win32.cpp, starting at line 610.
7. Enable Global Hook
8. Start the game
9. If you see an overlay on the top left of the screen, it works. Otherwise check if your path points to the OpenEXR libraries.
   F12 will enable the capture mode and dump a frame every 40 frames. Stop capturing by pressing F12 again.


Setup for processing frames (single frame):
-------------------------------------------------------------------------------
1. Start renderdocui.exe
2. Load a capture file via Open Log
3. Open the python shell
4. Run the script we provided via Run scripts or use the interactive shell.
   You may need to configure the python script we provided to your environment.


Setup for processing frames (automated):
-------------------------------------------------------------------------------
renderdoc saves UI settings in %APPDATA%/renderdoc/UI.config. 
If it does not exist, run renderdoc once and close it again.
Open the file and eit the following values:
- Point LoadScriptFile to the script you want to run for each capture file.
- Set ExecuteScriptOnLoad to true. This will load the script you specified above
  and execute it right after a capture file has been loaded.
- You can start renderdoc from the comand line: renderdocui.exe <path to capture file>
- To close renderdoc after processing is done, add 'renderdoc.AppWindow.Close()' to your script


Annotating the data
-------------------------------------------------------------------------------
- Coming soon.
