# BASH_SNOPT_install
  How to install Windows Subsystem for Linux (BASH) and SNOPT optimizer on windows PC
  Bash is basically an Ubuntu operating sytsem inside your computer. 

# BASH
To run SNOPT you will need the Bash for Windows linux subsystem. This subsystem runs a linux computer on your machine that can access all the same files as windows.
1) Get admin priviledges for your computer. This typically involves a request to your system admin.
2) Install WSL [following this guide](https://docs.microsoft.com/en-us/windows/wsl/install)
3) If that fails, try doing a manual install using [this guid](https://docs.microsoft.com/en-us/windows/wsl/install-manual) or [this guide](https://www.windowscentral.com/how-install-bash-shell-command-line-windows-10)

  Start bash by opening it up on the windows menue, 
  from here on out, anytime you see this:

  `this is a command` - that means type a command into the bash promt

  _<A_NAME>_ - replace this with a name of your choice (no spaces please)

# FOLDERS
Create a directory on your C: drive where we will install everything else and we will refer to this as _<OMDAO_FOLDER>_

`cd /mnt/c`

`mkdir <OMDAO_FOLDER>`

you can make this folder named whatever you want, just remember to change it in all the other commands below!!!!!
Both windows and linux will have full access to this folder so any changes you make on one will be refecleted on the other!
Do NOT put this folder in /myDocuments/. If you do, you will experiences file premission problems like [this](https://askubuntu.com/questions/1049895/permission-error-copying-files-into-ubuntu-on-windows-with-windows-copy-paste).

# MiniConda
Download Miniconda for Linux: note that anaconda itself is no longer free for commercial use :(.

`curl -O https://repo.anaconda.com/miniconda/Miniconda3-4.7.12.1-Linux-x86_64.sh`

`bash Miniconda3-4.7.12.1-Linux-x86_64.sh`

Some of the newer miniconda versions don't work well with WSL but if you want to try them out you can [download them here](https://docs.conda.io/en/latest/miniconda.html).
There may be some challenges adding miniconda to your path if you don't install using the `curl -O` method described above.

Press enter and then navigate to the bottom of the output. Careful, if you navigate too far you will pass where you need to agree to terms, and it will abort the install.
When prompted whether or not you want to prepend to the bashrc file, type `yes`. Alternatively, you can follow the directions below to modify the bashrc file yourself.

# bashRC
bashRC is a file that specifies what happens when you start up bash. We are going to setup miniconda on your path (if you selected not to do this automatically in the previous section) by editing bashRC

1) find out what your home directory folder is named:

  `cd ~/..`
  
  `ls`
  
  example output:

  ```
  (base) earetski@GRLWL2021061196:/mnt/c/OMDAO$ cd ~/..
  (base) earetski@GRLWL2021061196:/home$ ls
  earetski
  ```
  
  we can see from this that _earetski_ is my home directory name
  you should copy this name down.

2) replace _<YOUR NAME>_ in the following commands with the correct folder name of your home directory
now we edit bashrc:
  
  `sudo nano ~/.bashrc`

  scroll to the bottom of the page then add the following two lines of text:

  ```
  ## always add miniconda to path at start of session ##
  export PATH="/home/<YOUR_NAME>/miniconda/bin:$PATH"
  ```

  now exit bashrc by typing `ctrl+X`
  
  type `y` to save the edits to the file and then hit `enter`
  
3) Verify the install by exiting the bash window (close it). Then restart it. when you type `conda` your system should report that it can find the command and will display something lik this:
  
  ```
  usage: conda [-h] [-V] command ...

  conda is a tool for managing and deploying applications, environments and packages.

  Options:

  positional arguments:
    command
      clean        Remove unused packages and caches.
      compare      Compare packages between conda environments.
      config       Modify configuration values in .condarc. This is modeled after the git config command. Writes to the user .condarc file (/home/earetski/.condarc) by default.
      etc
  ```

# Environment
  We're going to use conda environments to allow us to have multiple python environments simultaneously. That way if your are working on multiple projects, and each project has a different python version it's working with, you can keep them separate.
  
  `conda create -n mdaowork python=3.8` 

  Activate the environment 

  `source activate mdaowork`
  
  Verify that python 3.8 is installed in this environment

  `python --version`
  
  example output: `'Python 3.8.10 :: Continuum Analytics, Inc.'`
  See the help if you can't get this to work

# UPDATES
  Now we need to update your WSL system files to the latest versions

  `sudo apt-get update`
  
  `sudo apt-get upgrade`
  
  You can run this every few months to pull the latest security updates.

# Packages 
  We're going to install a bunch of smaller support programs here. If pip is not already installed, we need to install it, it's a useful program for installing other programs

  `conda install pip`

  it will either say that all the requested packages were already installed, or you should see a lot of output go by.
  
  next we want to make sure that the pip version is the most up to date. Type:

  `pip install --upgrade pip`

  We will also neet git to install all OpenMDAO libraries

  `sudo apt-get install git`

  I think pyDOE2 is still a requirement as well.
  
  `pip install pyDOE2`

  Next Install matplotlib and plotly for plotting. Note: At least on OS X, matplotlib requires using conda to install it.
  `conda install matplotlib`

  Miniconda may generate an error when trying to conda install matplotlib. If that is the case, try:

  `pip install matplotlib`

  then install plotly using:

  `pip install plotly`

  these are also useful programs
  
  `pip install PyQt5`


# bashRC Take II
To activate the environment and open the <OMDAO_FOLDER> every time you open bash, we need to edit bashRC again

  `sudo nano ~/.bashrc`

  Scroll down to the bottom then add these four lines:

  ```
  ## Always activate mdaowrok environment on startup ##
  source activate mdaowork
  ## Always cd into <OMDAO_FOLDER> when bash starts ##
  cd /mnt/c/<OMDAO_FOLDER>
  ```
  
  Make sure to replace _<OMDAO_FOLDER>_ with the real folder name you selected at the start of this install!! Then exit and save your changes using the commands found at the bottom of the screen (`ctrl+x`, `y`, `enter`). 


# OpenMDAO Install
Use pip to install OpenMDAO from its github repo the following sets of commands will create a folder inside _<OMDAO_FOLDER>_ where OpenMDAO2.0 will be installed

  ```
  cd /mnt/c/<OMDAO_FOLDER>`
  git clone https://github.com/OpenMDAO/OpenMDAO.git
  cd openmdao
  pip install -e .
  ```
  
  when we install things with `pip install -e .`, every time we update a file in this folder, in will be re-installed. This makes pulling updated version of OpenMDAO as we can just use `git pull` get grab those version. Then this pagage and their libraries are auto-updated so you can reference them in your simulations. If you don't use `-e .` then you'll have to manually re-run `pip install` each time you update the OpenMDAO library. 


# PyOptSparse Install
# now we install a bunch of dependencies for pyoptsparse first

$ pip install sqlitedict
$ sudo apt-get install mpi
$ sudo apt-get install g++
$ sudo apt-get install mpich
$ pip install mpi4py
$ sudo apt-get install swig
$ pip install mdolab-baseclasses
# for some reason bdolab-baseclasses was failing to import automatically so we'll install it directly

# If you have a previous version of PyOptSparce/SNOPT you will need to remove them before continuing
# do this by opening a python prompt
$ python
>>> import pyoptsparse; pyoptsparse.__file__
'/home/earetski/anaconda3/envs/mdaowork/lib/python3.6/site-packages/pyoptsparse-2.1.5-py3.6-linux-x86_64.egg/pyoptsparse/__init__.py'
# the output from this will list the location where PyOptSparce is installed and we'll need to remove all of these locations
# now we're going to remove that file location manually
>>> exit()
$ rm -r /home/earetski/anaconda3/envs/mdaowork/lib/python3.6/site-packages/pyoptsparse-2.1.5-py3.6-linux-x86_64.egg/pyoptsparse
# keep trying to locate pyoptsparse, removing it's file location until you get the error:
>>> import pyoptsparse; pyoptsparse.__file__
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    ModuleNotFoundError: No module named 'pyoptsparse'
# now you're ready to continue to a fresh install

# Get pyoptsparse (a 3rd party optimizer that works well with OpenMDAO)
$ cd /mnt/c/<OMDAO_FOLDER>
$ git clone https://github.com/mdolab/pyoptsparse
$ cd pyoptspare
$ python setup.py build_ext --inplace

# If you want to install SNOPT than skip to the next section!
# Otherwise, to skip installing SNOPT run this:
$ cd /mnt/c/<OMDAO_FOLDER>/pyoptspare
$ python setup.py install


   _____   _   _    ____    _____    _______ 
  / ____| | \ | |  / __ \  |  __ \  |__   __|
 | (___   |  \| | | |  | | | |__) |    | |   
  \___ \  | . ` | | |  | | |  ___/     | |   
  ____) | | |\  | | |__| | | |         | |   
 |_____/  |_| \_|  \____/  |_|         |_| 
 ===============================================
# If you have windows 10+, then you can install SNOPT
# Get SNOPT source files from Eliot and Justin (Free if you work at NASA)

# Brows to the location where you downloaded the snopt source files
# copy the individual SNOPT src 7.7.1 files (not the folder but the individual files) 
# paste those files into your pysnopt source directory (shown below)
# c:/<OMDAO_FOLDER>/pyoptsparse/pyoptsparse/pySNOPT/source
# (do not include snopth.f if present, this file should have already been removed)

# now we go back to the bash command line to install pyoptspare which will run SNOPT
$ cd /mnt/c/<OMDAO_FOLDER>/pyoptspare
$ python setup.py install

# To check to make sure you have a compatible pyoptsparse and SNOPT version run test flow
$ cd /mnt/c/<OMDAO_FOLDER>/pyoptspare
$ testflo .
# 


     /\                    
    /  \      __ _    __ _ 
   / /\ \    / _` |  / _` |
  / ____ \  | (_| | | (_| |
 /_/    \_\  \__, |  \__, |
============  __/ | = __/ | ======
             |___/   |___/

# We need to set some defaults for Matplotlib so that it doesn't try to open graphs
# Identify the location of matplotlibrc, the file that holds default settings for matplotlib
$ python -c "import matplotlib; print(matplotlib.matplotlib_fname())"

# example output: /home/earetski/anaconda3/envs/mdaowork/lib/python3.6/site-packages/matplotlib/mpl-data/matplotlibrc

# now we neeed to edit this file
$ sudo nano <matplotlibrc file location>

# enter your admin password
# scroll down till you find the line where it says: 
# backend      : Qt5agg
# or this
# backend      : agg
# change this line to the following by removing the Qt5 and the # tag

backend      : agg

# then save the file by hitting ctrl+X, ENTER


  _______                _     ______   _         
 |__   __|              | |   |  ____| | |        
    | |      ___   ___  | |_  | |__    | |   ___  
    | |     / _ \ / __| | __| |  __|   | |  / _ \ 
    | |    |  __/ \__ \ | |_  | |      | | | (_) |
    |_|     \___| |___/  \__| |_|      |_|  \___/
==================================================
# Test the openmdao installation.
$ pip install testflo
$ cd /mnt/c/<OMDAO_FOLDER>/openmdao
$ testflo .

# Hopefully toward the bottom of the output you see something like:
# EXAMPLE:
# Passed:  1065
# Failed:  0
# Skipped: 153

# Skipped tests happen due to a lack of MPI or other third party software

  _____   __     __  __  __    ____     _____ 
 |  __ \  \ \   / / |  \/  |  / __ \   / ____|
 | |  | |  \ \_/ /  | \  / | | |  | | | (___  
 | |  | |   \   /   | |\/| | | |  | |  \___ \ 
 | |__| |    | |    | |  | | | |__| |  ____) |
 |_____/     |_|    |_|  |_|  \____/  |_____/
===============================================
# Install DYMOS which will allow the user to solve time-based ODEs
$ cd /mnt/c/<OMDAO_FOLDER>
$ git clone https://github.com/OpenMDAO/dymos.git
$ cd dymos
$ pip install -e .

                 __   __  _____     _____   __  __ 
                 \ \ / / |  __ \   / ____| |  \/  |
  _ __    _   _   \ V /  | |  | | | (___   | \  / |
 | '_ \  | | | |   > <   | |  | |  \___ \  | |\/| |
 | |_) | | |_| |  / . \  | |__| |  ____) | | |  | |
 | .__/   \__, | /_/ \_\ |_____/  |_____/  |_|  |_|
 | | ====  __/ | ==================================                                   
 |_|      |___/

# py XDSM allows for the creation of XDSM diagrams in python using LaTeX libraries
# pyXDSM install instructions for ubuntu:
# http://mdolab.engin.umich.edu/content/xdsm-overview
# https://github.com/mdolab/pyXDSM

$ cd /mnt/c/<OMDAO_FOLDER>
$ git clone https://github.com/mdolab/pyXDSM.git
$ cd pyXDSM
$ pip install -e .

# now we need pdflatex to work
$ pdflatex
# if this returns an error we need to install LaTeX packages
$ sudo apt-get install texlive-full

# only use the links below if you don't have enough storage space for the full install
$ sudo apt-get install texlive-latex-base
$ sudo apt-get install texlive-latex-extra
$ sudo apt-get install texlive-latex-recommended
$ sudo apt-get install texlive-pictures

  _____                                          
 |_   _|                                         
   | |    _ __ ___     __ _    __ _    ___   ___ 
   | |   | '_ ` _ \   / _` |  / _` |  / _ \ / __|
  _| |_  | | | | | | | (_| | | (_| | |  __/ \__ \
 |_____| |_| |_| |_|  \__,_|  \__, |  \___| |___/
============================== __/ |  ============           
                              |___/  
Ubuntu on Windows does not allow pop-up windows by default. This makes it hard when using plt.show() in matplotlib. Hours could be spent saving and opening files instead of a simple pop-up window.
These steps will correct this:
Download Xming from https://sourceforge.net/projects/xming/
Note: You will need elevated priveldges to install this. Request this through IdMax (EUSO Elevated Privileges). Be sure to take the training in SATERN for you to be able to have these preveldges. The course is ITS-002-09 (in SATERN).
Note that when you install this, your computer will not allow you so save the file in the default location that Xming wants to place itself. Instead, find another location on your local machine. 'Documents' has worked for me. You may need to 'ignore' a few downloads. It will still work without those files. Be sure to run the XLaunch.exe file. Xming needs to be running in order for plots to display. Think about making it so your computer runs this program during startup. See directions at the bottom of this tutorial.
Install the tk package:
sudo apt-get install python3-tk
(remove the 3 if using Python 2)
Next, type in the Ubuntu window:
cd ~
nano .bashrc
Note: you may need to install nano for the above line to work. You can do this by typing:
sudo apt install nano
This will open up .bashrc for you to edit. At the very bottom of the file, add the following:
## Make it so Ubuntu will open plots in matplotlib
export DISPLAY=localhost:0.0
## Tell MatPlotLib to use Tk Package
export MPLBACKEND="TkAgg"
Then hit ctrl+x on your keyboard to save and close the file. You will need to close and reopen your Ubuntu window for this to take effect. You should now be able to use matplotlib's plt.show() function!
SETUP FOR RUNNING Xming AT COMPUTER STARTUP:
Go to where you saved Xming on your computer (if you forgot, you can always just search for it) and open the file location and run XLaunch.exe. You will have a window pop up for you to select some options. Select 'Multiple Windows' and make sure the Display number is 0
Click Next then select 'Start no client', then next. Select 'Clipboard' then next. Last, click 'Save configuration' and save it as 'config.xlaunch' (no quotes). You now saved the conifuration Xming will run every time you open it. Now you're going to want to create a shortcut for XLaunch.exe file to your startup program list. To do this, do to where your startup programs are located on your machine. That should be in the path below:
C:\Users\rpthacke\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
(note: you'll have a different username than me, replace rpthacke with yours)
In this location, place the Xlaunch.exe shortcut. Then right click on it and select 'Properties'. Modify the Target line under the 'Shortcut' tab. Change it to point where you have the Xming location saved along with -run "config.xlaunch". For me, it reads as:
C:\Users\rpthacke\Documents\Xming\XLaunch.exe -run "config.xlaunch"
Now click 'OK' and everything is set to go! 

  _    _          _         
 | |  | |        | |        
 | |__| |   ___  | |  _ __  
 |  __  |  / _ \ | | | '_ \ 
 | |  | | |  __/ | | | |_) |
 |_|  |_|  \___| |_| | .__/ 
==================== | | ===    
                     |_|

### Help 1 ###
# Sometimes there are problems with mpi
# check your Ubuntu installation version 
$ lsb_release -a
# In versions before 16.04, you may have to set the following at the command line:
$ export KMP_AFFINITY=disabled
# https://github.com/Microsoft/WSL/issues/785
# you won't get a return prompt if it does work
# if that doesn't work try
# $ KMP_AFFINITY=disabled


### Help 2 ###
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


### Help 3 ###
# If you have trouble and are stuck we suggest the following resources:
www.google.com


### Help 4 ###
# If you want this doc to be updated, please send us an email
eliot.d.aretskin-hariton@nasa.gov



### Help 5 ###
Juila Install
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


### HELP 6 ###
When using miniconda, I wasn't able to install python 3.8 I got the error:
# earetski@GRLWL2021061196:/usr/bin$ conda create -n mdaowork python=3.8
Fetching package metadata: ....
Error: No packages found in current linux-64 channels matching: python 3.8*

Resolution: 
# conda install anaconda-clean

after installing the above I was able to find the correct python version
