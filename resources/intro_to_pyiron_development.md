# Introduction

Let’s say you found a bug<sup>1</sup> in Pyiron and want to fix it yourself or understand what is happening in the background when you run a function. First you need to understand the structure of Pyiron

## Different Pyiron modules

Pyiron exists as a bunch of files (aka modules) with code written in them that call/communicate with each other on a remote computer (aka hosting server).

* [pyiron](https://github.com/pyiron/pyiron) - A meta package which seamlessly loads all the pyiron plugins. 
* [pyiron_base](https://github.com/pyiron/pyiron_base) - A package for the core compotents e.g. the job management, data storage and resource management. 
* [pyiron_atomistics](https://github.com/pyiron/pyiron_atomistics) - A interface to atomistic simulation codes including but not limited to GPAW, LAMMPS, S/Phi/nX and VASP. 
* [pyiron_contrib](https://github.com/pyiron/pyiron_contrib) - A package to collect contributions from the pyiron community.
* [pyiron_gpl](https://github.com/pyiron/pyiron_gpl) - A package for all interfaces which require a GPL license in contrast to the BSD license used by all other pyiron packages. 
* [pyiron_continuum](https://github.com/pyiron/pyiron_continuum) - A package to extent pyiron to the contiuum scale. 
* [pyiron_experimental](https://github.com/pyiron/pyiron_experimental) - A package to apply pyiron for the post-processing of experimental measurements. 
* [pyiron_gui](https://github.com/pyiron/pyiron_gui) - A graphical user interface for pyiron based on ipywidgets. 

## Git and GitHub
Git and GitHub are two of the most important tools/concepts that you need to be familiar with.

To allow different developers (aka now you!) to modify, improve, or add to Pyiron, you need a version control system to manage these modifications, keep track of every change, and prevent overlapping of different changes, this is Git. For Git commands, [see](https://gitlab.mpcdf.mpg.de/eisenforschung/cm/documentation/-/blob/master/Git.md)!

You also need a website (a graphical user interface) to host and manage the code (the Git repositories<sup>2</sup>) and the changes, track bugs, and include documentation (what you are reading now) and tests that make sure any changes you introduce do not crash another part of the code, and also allow anyone (outside of MPIE) to use and make changes into the code, this is GitHub. 

1. Bug: Weird error or a result that you did not expect and a little suspicious about.
2. Repository: If you are a Linux (Windows) user then this is a fancy word for project directory (folder) that contains files of your code.

## Cloning 

Now to change and test something in Pyiron, you can't change the code directly. You have to make a copy (clone) of the Pyiron code (or a specific part/module) of it, change it, test it, and then combine it with the current version.  

You have two options on where to do this: 1. locally on your laptop (see section XXX) or 2. on the cluster directly (the rest of this section). The second option is preffered as it is more straightforward and also, if you are playing with a code that depends to one of the closed source/liscenced simulation packages (e.g VASP) you can only this on the cluster.

The first step is clone the latest Pyiron version (Here we are adding an empty file in the pyiron atomistics module)
