Playing for Data
===============================================================================

This source code release accompanies the paper  

**Playing for Data: Ground Truth from Computer Games**  
Stephan Richter, Vibhav Vineet, Stefan Roth and Vladlen Koltun. In ECCV 2016.  

Clone via `git clone --recursive https://bitbucket.org/visinf/projects-2016-playing-for-data ./playing-for-data`.


Website
-------------------------------------------------------------------------------
https://download.visinf.tu-darmstadt.de/data/from_games/index.html


License
-------------------------------------------------------------------------------
This code is released under the MIT license, see LICENSE.md for full text as well as 3rd party library acknowledgements.

Requirements:
-------------------------------------------------------------------------------
* CMake (for making OpenEXR and zlib)
* Visual Studio (for compiling renderdoc) - tested on VS 2013
* Python - tested with Anaconda
* MATLAB (for annotating frames) - tested on ML 2013a


Let <PFD_DIR> be the Playing for Data directory with the following structure:

* renderdoc  | Graphic Debugger - dump frames to disk
* scripts    | Python scripts for extracting data from games using renderdoc
* README.md  | this file.


Preparation (OpenEXR):
-------------------------------------------------------------------------------
1. Use CMake to build zlib *<PFD_DIR>/renderdoc/renderdoc/3rdparty/zlib*
2. Use CMake to build IlmBase (located at *<PFD_DIR>/renderdoc/renderdoc/3rdparty/openexr/IlmBase*).  
   Disable the `NAMESPACE_VERSIONING` option.  
   Set `CMAKE_INSTALL_PREFIX` to your preferred directory.   
3. Compile IlmBase by building the `INSTALL` target.
4. Use CMake to build OpenEXR (located at *<PFD_DIR>/renderdoc/renderdoc/3rdparty/openexr/OpenEXR*).  
   Make sure that `ILMBASE_PACKAGE_PREFIX` in CMake is set to the install directory of IlmBase
   (the one you specified as `CMAKE_INSTALL_PREFIX` in the 2nd step).  
   Disable the `NAMESPACE_VERSIONING` option.  
   If you built zlib in a separate build directory, copy the *zconf.h* from that directory to the `ZLIB_INCLUDE_DIR`
5. Compile OpenEXR. Make sure that the ilmbase libraries (Half, IlmThread) are on your library search path 
   (for setting the path under windows see Preparation 2. below), 
   otherwise compiling the IlmImf project may fail with some cryptic error (Visual Studio returns `"cmd.exe" exited with code -1073741515`).
   

Preparation (renderdoc):
-------------------------------------------------------------------------------
1. Compile the complete *renderdoc* solution.
2. Make sure the OpenEXR dlls are in a directory pointed to by your path variable.
   (Setting the path under windows is done at
    *Control Panel/System and Security/System/Advanced System Settings/Environment Variables*)



Setup for capturing:
-------------------------------------------------------------------------------
1. Run *renderdocui.exe* with admin privileges.
2. Specify the path where dumped frames are stored: *Tools/Options/Directory for temporary capture files*
3. Enable global process hooking in *Tools/Options/Allow global process hooking* (May not be needed by all games)
4. Open the *Capture Executable* Tab.
5. Point the *Executable Path* to your game
6. If you are using the global hook, the other settings don't matter as they are set in a different process.  
   If you need to modify them, check *<PFD_DIR>/renderdoc/renderdoccmd/renderdoccmd_win32.cpp*, starting at line 610.
7. *Enable Global Hook*
8. Start the game
9. If you see an overlay on the top left of the screen, it works. Otherwise check if your path points to the OpenEXR libraries.
   F12 will enable the capture mode and dump a frame every 40 frames. Stop capturing by pressing F12 again.


Setup for processing frames (single frame):
-------------------------------------------------------------------------------
1. Start *renderdocui.exe*
2. Load a capture file via *Open Log*
3. Open the python shell
4. Run the script we provided via *Run scripts* or use the interactive shell.  
   You may need to configure the python script we provided to your environment.


Setup for processing frames (automated):
-------------------------------------------------------------------------------
*renderdoc* saves UI settings in `%APPDATA%/renderdoc/UI.config`. 
If it does not exist, run *renderdoc* once and close it again.  
Open the file and edit the following values:

* Point `LoadScriptFile` to the script you want to run for each capture file.
* Set `ExecuteScriptOnLoad` to true. This will load the script you specified above
  and execute it right after a capture file has been loaded.
* You can start *renderdoc* from the command line: `renderdocui.exe <path to capture file>`
* To close *renderdoc* after processing is done, add `renderdoc.AppWindow.Close()` to your script


Annotating the data
-------------------------------------------------------------------------------

* Running `initLabels.m` will create a *label.mat* containing
  the class names, ids, and colors for annotation. 

* For annotating of a single frame, we need the following artifacts to be
  extracted using *renderdoc*:
   * <frame>__final.png    |  rendered image, rgb 3 channels, 8-bit each
   * <frame>__id.mat 	     |  *meshID*, *texID*, *shaderID* images created from the ID buffers, single channel double for each image.
   * <frame>__tex.txt      |  resource ID to hash list
   * <frame>__mesh.txt     |  resource ID to hash list
   * <frame>__shader.txt   |  resource ID to hash list

* The resource ID to hash lists have the format `%d,%08x.%08x\n`
  in each line, where the first argument is a resource ID and the last two
  arguments are 2 64-bit integers forming the 128-bit hash for this resource.

* After extracting artifacts for each frame, label it by running
  `labelMTS(dir, frame)`, where *dir* is the directory containing all extracted resources and *frame*
  is the prexix identifying resources of a specific frame.
  Make sure to press the *STOP* button in the top left corner after you
  are done labeling each frame. It saves your annotations.

* `autolabelStencil.m` is an example of how to label MTS automatically
  using a class mask derived from e.g. a stencil buffer.

* After labeling a certain amount of mesh, shader, texture tuples (MTS),
  you can start creating some labeling rules, which will further speed up labeling.
  If e.g. a certain shader is only used for cars, we can assign 
  the *car* label to all MTS tuples that contain this shader.
  Create these kind of rules by running `createShortcurs` with 
  minimum occurence thresholds of your choice.

* Use `labelMTSSingleFrame(dir, frame)` if you want to make annotations
  for only this frame. Using `labelMTS` will propagate annotations to all frames.

* Finally, segment images by calling `segmentImages(dir, frame)`.

Some technical background on the annotation process
-------------------------------------------------------------------------------
During extraction we save ID buffers using resource IDs, 
which will differ between game sessions. We convert them
to persistent hashes using the lists mentioned above.
These associations will be cached in 
<frame>__res2hash.mat

During annotation (labelMTS.m) we create the following files:
* <frame>__res2hash.mat    | caches resource IDs to hashes for a frame
* hash2cid.mat             | global MTS to class annotations
* shaderHash2class.mat     | shortcuts from association rule mining
* meshHash2class.mat       | shortcuts from association rule mining
* texHash2class.mat        | shortcuts from association rule mining


