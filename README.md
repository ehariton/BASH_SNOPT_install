# BASH_SNOPT_install
  How to install Windows Subsystem for Linux (BASH) and SNOPT optimizer on windows PC
  Bash is basically an Ubuntu operating sytsem inside your computer. 
  You can install SNOPT If you have windows 10+. 
  Verified to work with Windows 10 build 19042

# BASH
To run SNOPT you will need the Bash for Windows linux subsystem. This subsystem runs a linux computer on your machine that can access all the same files as windows.
1) Get admin priviledges for your computer. This typically involves a request to your system admin.
2) Open a windows command prompt in admin mode 
3) Force WSL v1.0 install by typing `wsl --set-default-version 1`. There is a problem with WSL 2.0 and Cisco AnyConnect VPN that prevents internet access.
4) Install Ubuntu by using the command in the windows command prompt: `wsl --install -d Ubuntu`
If that fails, try [following this guide](https://docs.microsoft.com/en-us/windows/wsl/install), or doing a manual install using [this guid](https://docs.microsoft.com/en-us/windows/wsl/install-manual) or [this guide](https://www.windowscentral.com/how-install-bash-shell-command-line-windows-10)
5) Once the WSL install is successful, the WSL window will open and ask for a Unix username and password. Alternatively, the computer may ask you to restart it. On restart, the WSL window will open after a few minutes and ask for a unix unsername and password.
6) Verify that you have WSL 1 installed by opening the windows compand promt and typing `wsl -l -v`
8) From here on out, anytime you see this:

  `this is a command in bash/WSL` 
  
  ```
  this is a series 
  of commands to be in bash/WSL
  
  ```
  
  `>>> this is a command in python`
 

# UPDATES
  Now we need to update your WSL system files to the latest versions

  ```
  sudo apt-get update
  sudo apt-get upgrade -y
  
  ```
  
  You can run this every few months to pull the latest security updates.
  
# FOLDERS
Create a directory on your C: drive where we will install everything else and we will refer to this as the _OMDAO_ folder. If you want to name this something else you'll have to change all the rest of the commands in the walkthrough to reflect your new folder name.

```
cd /mnt/c
mkdir OMDAO
cd OMDAO

```

you can make this folder named whatever you want, just remember to change it in all the other commands below!!!!!
Both windows and linux will have full access to this folder so any changes you make on one will be refecleted on the other!
Do NOT put this folder in /myDocuments/. If you do, you will experiences file premission problems like [this](https://askubuntu.com/questions/1049895/permission-error-copying-files-into-ubuntu-on-windows-with-windows-copy-paste).

# MiniConda
Download Miniconda for Linux: note that anaconda itself is no longer free for commercial use :(. You must download the Miniconda file below into your OMDAO folder. If you try to download it in `/home/` it will say _Warning: Permission Denied_. 

```
cd /mnt/c/OMDAO
curl -O https://repo.anaconda.com/miniconda/Miniconda3-4.7.12.1-Linux-x86_64.sh
bash Miniconda3-4.7.12.1-Linux-x86_64.sh

```

Some of the newer miniconda versions don't work well with WSL but if you want to try them out you can [download them here](https://docs.conda.io/en/latest/miniconda.html).
There may be some challenges adding miniconda to your path if you don't install using the `curl -O` method described above.

Press enter and then navigate to the bottom of the output and type 'yes' to install.

When prompted _Do you wish the installer to initialize Miniconda3 by running conda init?_ type `yes`

Pin WSL to your taskbar by Right clicking it in the windows task bar and selecting _Pin to Task Bar_

Then close WSL and reopen it.

# Environment
  We're going to use conda environments to allow us to have multiple python environments simultaneously. That way if your are working on multiple projects, and each project has a different python version it's working with, you can keep them separate.
  
  ```
  conda create -n mdaowork python=3.8 -y
  conda activate mdaowork
  
  ```
  
If you get the error: _No packages found in current linux-64 channels matching: python 3.8*_. 
The resoultion should be to run `conda install anaconda-clean` and then re-trying the original command.
  
  Verify that python 3.8 is installed in this environment

  `python --version`
  
  example output: _'Python 3.8.12'_

# bashRC
To activate the environment and open the OMDAO folder every time you open bash, we need to edit bashRC

  `sudo nano ~/.bashrc`

  Scroll down to the bottom then add these four lines:

  ```
  ## Always activate mdaowrok environment on startup ##
  conda activate mdaowork
  ## Always cd into OMDAO folder when bash starts ##
  cd /mnt/c/OMDAO
  ```
  
  Then exit and save your changes using the commands found at the bottom of the screen (`ctrl+x`, `y`, `enter`). 
  
  Verify this has worked by restarting WSL. It should drop you in your environment, indicated by the _(mdaowork)_.
    

# Packages 
  We're going to install a bunch of smaller support programs here.  Install matplotlib and plotly for plotting. Note: At least on OS X, matplotlib requires using conda to install it. Then we install plotly which is a useful plotting library. Testflo is used to test components in the OpenMDAO installation:
  
  ```
  conda install matplotlib -y
  pip install plotly testflo
  
  ```

Miniconda may generate an error when trying to conda install matplotlib. If that is the case, try:

  `pip install matplotlib`
  
# OpenMDAO Install
Use pip to install OpenMDAO from its github repo the following sets of commands will create a folder inside OMDAO folder where OpenMDAO2.0 will be installed. 

For OpenMDAO users, we encourage you to pip install Openmdao

  `pip install openmdao`

For Developers, we encourage you to clone the OpenMDAO repo and install from there.

  ```
  cd /mnt/c/OMDAO
  git clone https://github.com/OpenMDAO/OpenMDAO.git
  cd OpenMDAO
  pip install -e .
  
  ```
  
  When we use the command _pip install -e ._, this means every time we update a file in this folder, in will be re-installed. This makes pulling updated version of OpenMDAO as we can just use `git pull` get grab those version. Then this pagage and their libraries are auto-updated so you can reference them in your simulations. If you don't use `-e .` then you'll have to manually re-run `pip install` each time you update the OpenMDAO library. 
  
  ## Operation Not Permitted Error
  If you get an _Operation Not Permitted_ [error issue](https://askubuntu.com/questions/1115564/wsl-ubuntu-distro-how-to-solve-operation-not-permitted-on-cloning-repository) when using `git clone` that's because the Windows file system isn't happy with the linux permissions. This typically happens on computers where you're not admin. We're going to have to do a quick workaround which is to "automatically mount your Windows drives under WSL with the metadata option. 
  
  `sudo nano /etc/wsl.conf`
  
  Then add these lines into that file:
  
  ```
  [automount]
  options = "metadata"
  ```
  
  Now we need to exit our WSL instance, and shut down all it's background processes as well. You can do this by either Rebooting the whole computer, or using the `wsl --shutdown` command in the regular windows command promt. After this re-open WSL and git cloning should now work
  
  ## Test the OpenMDAO Installation
  Now lets test the openMDAO installation by running testflo in the folder where we installed OpenMDAO e.g. _/mnt/c/OMDAO/OpenMDAO_ folder

  `testflo .`

  Example output:
  
  ```
  (mdaowork) earetski@GRLWL2021061196:/mnt/c/OMDAO/OpenMDAO$ testflo .
  SSS...........................................................................  etc
  Passed:  1065
  Failed:  0
  Skipped: 153
  ```
  
  Skipped tests happen due to a lack of MPI or other third party software.
  
# Remove Old PyOptSparse Install (if required)
  If you have a previous version of PyOptSparce/SNOPT you will need to remove them before continuing. Do this by opening a python prompt:

  ```
  python
  >>> import pyoptsparse; pyoptsparse.__file__
  >>> exit()
  ```
  example output: 
  
  ```
  (mdaowork) earetski@GRLWL2021061196:/home$ python
  Python 3.8.10 (default, Jun  4 2021, 15:09:15)
  [GCC 7.5.0] :: Anaconda, Inc. on linux
  Type "help", "copyright", "credits" or "license" for more information.
  >>> import pyoptsparse; pyoptsparse.__file__
  '/home/earetski/miniconda3/lib/python3.8/site-packages/pyoptsparse/__init__.py'
  >>> exit()
  ```
  
  where 
  `'/home/earetski/miniconda3/lib/python3.8/site-packages/pyoptsparse/__init__.py'` is your _<OLD_INSTALL_LOCATION>_
  
  the output from this will list the location where PyOptSparce is installed and we'll need to remove all of these locations. Now we're going to remove that file location manually. 
  
  Remove the file using: 
  `rm -r <OLD INSTALL LOCATION>`
  
  keep trying to locate pyoptsparse, removing it's file location until you get the error:

  *>>> import pyoptsparse; pyoptsparse.__file__
  Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  ModuleNotFoundError: No module named 'pyoptsparse'*
  
  now you're ready to continue to a fresh install
  
  
# PyOptSparse Install
1) We install a bunch of dependencies / compiler for pyoptsparse first

  ```
  sudo apt-get install mpi g++ mpich swig -y
  pip install sqlitedict mpi4py mdolab-baseclasses
  ```
For some reason bdolab-baseclasses was failing to import automatically so we'll install it directly

2) Get pyoptsparse (a 3rd party optimizer that works well with OpenMDAO)
  
  ```
  cd /mnt/c/OMDAO
  git clone https://github.com/mdolab/pyoptsparse
  cd pyoptsparse
  ```

  If you want to install SNOPT, go to the next section. 
  
  If you DO NOT want to instal SNOPT run this:

  ```
  cd /mnt/c/OMDAO/pyoptsparse
  pip install -e .
  
  ```

# SNOPT
  Get SNOPT source files. These are available for free if you work at NASA.

  Brows to the location where you downloaded the snopt source files

  copy the individual SNOPT src 7.7.1 files (not the folder but the individual files) 
  
  (do not include snopth.f if present, this file should have already been removed)
  
  paste those files into your pysnopt source directory (shown below)
  
  _c:/OMDAO/pyoptsparse/pyoptsparse/pySNOPT/source_

  now we go back to the bash command line to install pyoptspare which will run SNOPT
  
  ```
  cd /mnt/c/OMDAO/pyoptspare
  pip install -e .
  
  ```

  A huge amount of text will scroll by your screen.
  
  To check to make sure you have a compatible pyoptsparse and SNOPT version run test flow in the pyoptsparse folder.
  
  `testflo .`
  

# Dymos
Install DYMOS which will allow the user to solve time-based ODEs

For users we suggest using 
`pip install dymos`

For developers we recommend building from source:
  ```
  cd /mnt/c/OMDAO
  git clone https://github.com/OpenMDAO/dymos.git
  cd dymos
  pip install -e .
  
  ```


# pyXDSM
py XDSM allows for the creation of XDSM diagrams in python using LaTeX libraries. pyXDSM install instructions for ubuntu can be found [here](http://mdolab.engin.umich.edu/content/xdsm-overview) or [here](https://github.com/mdolab/pyXDSM). We want to clone pyXDSM as well as grab the required latex packages

  ```
  pip install pyXDSM
  sudo apt-get install texlive-latex-base texlive-latex-extra texlive-latex-recommended texlive-pictures -y
  
  ```
  
now we need verify we have pyXDSM and the latex packages install run
  
  `pdflatex`
  
example output:

  ```
  This is pdfTeX, Version 3.14159265-2.6-1.40.20 (TeX Live 2019/Debian) 
  (preloaded format=pdflatex) restricted \write18 enabled.
  **
  ```


Hit `ctrl+c` to exit from the window.

If _pdflatex_ returns an error we need to install the full LaTeX packages

  `sudo apt-get install texlive-full`

# You are Done! All the remaining packages are optional

# Juila Install
$ git clone git@github.com:dingraha/CCBlade.jl

# start julia
$ julia
julia> ]
pkg> status
# this lists all the packages installed in julia
# update paths to CCBlade and OpenMDAO
# you can type help to get help
# navigate to <OpenMDAO> folder above where CCBLade is cloned
pkg> ;
# or
julai> ;
shell> cd /mnt/c/<OpenMDAO>
# add dependant packages
pkg> add FLOWMath
# install packages that were just cloned
pkg> develop CCBlade.jl/
# develop is like pip install -e . 


pkg> status
# (v1.2) pkg> status
#    Status `~/.julia/environments/v1.2/Project.toml`
#  [e1828068] CCBlade v0.1.0 [`../../../../../mnt/c/OMDAO/CCBlade.jl`]
#  [6cb5d3fb] FLOWMath v0.2.1
#  [2d3f9b48] OpenMDAO v0.1.0 [`../../../../../mnt/c/OMDAO/OpenMDAO.jl`]
#  [438e738f] PyCall v1.91.2
#  [d330b81b] PyPlot v2.8.2

# navigate to aviary_quadrotor/aviary_quadrotor/subsystems/rotor  using the shell command
# then install the CCBladeAviaryQuadrotor.jl project/project
pkg> develop CCBladeAviaryQuadrotor.jl/
pkg> status
# [e1828068] CCBlade v0.1.0 [`../../../../../mnt/c/OMDAO/CCBlade.jl`]
#   [6b0330fc] CCBladeAviaryQuadrotor v0.1.0 [`../../../../../mnt/c/OMDAO/aviary_quadrotor/aviary_quadrotor/subsystems/rotor/CCBladeAviaryQuadrotor.jl`]
#   [6cb5d3fb] FLOWMath v0.2.1
#   [2d3f9b48] OpenMDAO v0.1.0 [`../../../../../mnt/c/OMDAO/OpenMDAO.jl`]
#   [438e738f] PyCall v1.91.2
#   [d330b81b] PyPlot v2.8.2
  
# AGG
  With the new WSL installs, Matplotlib is playing much more nicely with graphing, so we probably don't need to do anything with AGG anymore. I suggest you skip this section unless you are having AGG errors.

  We need to set some defaults for Matplotlib so that it doesn't try to open graphs
  Identify the location of matplotlibrc, the file that holds default settings for matplotlib

  ```python -c "import matplotlib; print(matplotlib.matplotlib_fname())"```

  example output: 
  
  _/home/earetski/anaconda3/envs/mdaowork/lib/python3.6/site-packages/matplotlib/mpl-data/matplotlibrc_ this is your _<matplotlibrc file location>_

  now we neeed to edit this file

  `sudo nano <matplotlibrc file location>`

  Enter your admin password. Then scroll down till you find the line where it says: 
  
  `# backend      : Qt5agg`
  or this
  `# backend      : agg`

  change this line to the following by removing the Qt5 and the # tag
  
  `backend      : agg`
  
  then save the file by hitting `ctrl+X`, `y`, `enter`


  
# XMING
  Things have gotten better with the more recent versions of WSL. We are advising people not to worry about the below instructions unless you are having difficulty running `plt.show()` when running python / matplotlib in bash.
  
  Ubuntu on Windows does not allow pop-up windows by default. This makes it hard when using plt.show() in matplotlib. Hours could be spent saving and opening files instead of a simple pop-up window. These steps will correct this:
[Download Xming](https://sourceforge.net/projects/xming/). 
Note that when you install this, your computer will not allow you so save the file in the default location that Xming wants to place itself. Instead, find another location on your local machine. 'Documents' has worked for me. You may need to 'ignore' a few downloads. It will still work without those files. Be sure to run the XLaunch.exe file. Xming needs to be running in order for plots to display. Think about making it so your computer runs this program during startup. See directions at the bottom of this tutorial.
Install the tk package, note: remove the 3 if using Python 2:

  `sudo apt-get install python3-tk`

  `sudo nano ~/.bashrc`
  
  Add add the following to the bottom of bashRC:
  
  ```
  ## Make it so Ubuntu will open plots in matplotlib
  export DISPLAY=localhost:0.0
  ## Tell MatPlotLib to use Tk Package
  export MPLBACKEND="TkAgg"
  ``
  
  Exit and save the file (`ctrol+X`,`y`,`enter). You will need to close and reopen your Ubuntu window for this to take effect. You should now be able to use matplotlib's plt.show() function!

Setup XMING to run at computer startup. Go to where you saved Xming on your computer (if you forgot, you can always just search for it) and open the file location and run XLaunch.exe. You will have a window pop up for you to select some options. Select 'Multiple Windows' and make sure the Display number is 0. Click Next then select 'Start no client', then next. Select 'Clipboard' then next. Last, click 'Save configuration' and save it as 'config.xlaunch' (no quotes). You now saved the conifuration Xming will run every time you open it. Now you're going to want to create a shortcut for XLaunch.exe file to your startup program list. To do this, do to where your startup programs are located on your machine. That should be in the path below:

  _C:\Users\<YOUR_USER_NAME>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup_
  
(note: you'll have a different username than me, replace <YOUR_USER_NAME> with yours)
In this location, place the Xlaunch.exe shortcut. Then right click on it and select 'Properties'. Modify the Target line under the 'Shortcut' tab. Change it to point where you have the Xming location saved along with -run "config.xlaunch". For me, it reads as:
C:\Users\<YOUR_USER_NAME>\Documents\Xming\XLaunch.exe -run "config.xlaunch"
Now click 'OK' and everything is set to go! 

# HELP

## MPI Install problems
# Sometimes there are problems with mpi
# check your Ubuntu installation version 
$ lsb_release -a
# In versions before 16.04, you may have to set the following at the command line:
$ export KMP_AFFINITY=disabled
# https://github.com/Microsoft/WSL/issues/785
# you won't get a return prompt if it does work
# if that doesn't work try
# $ KMP_AFFINITY=disabled

# Make sure that your computer name is listed as one of your hosts 
# incase you get wierd MPI errors like:
# sudo:unable to resolve host
# Fatal error in PMPI_Init_thread: Other MPI error, error stack:
# MPIR_Init_thread(467)..............: 
# MPID_Init(177).....................: channel initialization failed
# MPIDI_CH3_Init(70).................: 
# MPID_nem_init(319).................: 
# MPID_nem_tcp_init(171).............: 
# MPID_nem_tcp_get_business_card(418): 
# MPID_nem_tcp_init(377).............: gethostbyname failed,
# https://stackoverflow.com/questions/23112515/mpich2-gethostbyname-failed
# 
# Resolution:
$ more /etc/hostname
# make sure your computer name listed in *hostname* 
# EXAMPLE:
$ (mdaowork) earetski@GRSLW17090053:/mnt/c/OMDAO$ more /etc/hostname
$ GRSLW17090053

# If not added your hosts via:
$ sudo nano /etc/hosts
# add the line:
$ 127.0.1.1 <hostname>

