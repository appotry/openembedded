Getting started
Contents [hide](hide.md)
1 Setting up the toolchain and doing a build
1.1 Directory Structure
1.2 Obtaining BitBake
1.2.1 Getting a working bitbake
1.2.2 Using releases
1.3 Obtaining OpenEmbedded using Git
1.3.1 Updating OpenEmbedded
1.4 Create local configuration
1.5 Setup the environment
1.6 Start building
1.7 Adding Packages
2 Problems
3 Portability issues
4 Productivity notes
Setting up the toolchain and doing a build

Before starting to configure your OE installation by following the instructions on this page make sure you have read how OE fits in with your host distribution and the required software for compilation.
Directory Structure

The base directory of your Openembedded environment (/stuff/) is the location where sources will be checked out (or unpacked).
You must choose a location with no symlinks above it
If you work in a chrooted environment and have ccache installed it is highly recommended to 'su - 

&lt;username&gt;

' after you have chrooted. Compilation may fail because ccache needs a valid $HOME, which is usually set when using a user account. It is recommended that ccache is not installed on systems used to build OpenEmbedded as it has been known to introduce other subtle build failures.
To create the directory structure:
$ mkdir -p /stuff/build/conf
$ cd /stuff/
Obtaining BitBake

To start using OE, you must first obtain the build tool it needs: bitbake
It is recommended to run bitbake without installing it, as a sibling directory of openembedded/ and build/ directories. Indeed, as bitbake is written in python it does not need to be compiled. You'll just have to set the PATH variable so that the BitBake tools are accessible (see Setup the environment section).
Getting a working bitbake
Bitbake switched from a svn repository to a git one, and the former is stuck at version 1.8.13, so when you try to build you may face an error: "Bitbake version 1.10.2 is required and version 1.8.13 was found". In that case please fetch released version or use git repository.
Which version is safe to use? Last release one is always working. When OE changes require newer version of BitBake metadata is changed and you will get message like above.
One note for those who want to play with development versions of BitBake - Python 2.6 may be required by newer versions. This can be a problem for some Linux distributions.
Basically the easier and faster solution (at the moment I'm writing) is to get release one.
> wget http://download.berlios.de/bitbake/bitbake-1.10.2.tar.gz
Using releases
Visit BitBake homepage and download tarball with latest release. For normal usage we suggest using 1.8.x (stable branch) versions. Unpack it to /stuff/bitbake/.
Obtaining OpenEmbedded using Git

Note: Once upon a time OpenEmbedded was using Monotone for version control. If you have an OE Monotone repository on your computer, you should replace it with the Git repository.
Note: These are only brief instructions. For a longer description about using Git with OpenEmbedded refer to Git and GitPhraseBook.
The OpenEmbedded project resides in a Git repository. You can find it at git://git.openembedded.org/openembedded.
Web interface is: http://cgit.openembedded.org/
To obtain Openembedded:
Install git
Go to the base directory of your OpenEmbedded environment
$ cd /stuff/
Checkout the repository
$ git clone git://git.openembedded.org/openembedded
or for the firewall challenged try
$ git clone http://repo.or.cz/r/openembedded.git
This is the data you'll be using for all the work.
Updating OpenEmbedded
The .dev branch of OE is updated very frequently (as much as several times an hour). The distro branches are not updated as much but still fairly often. It seems good practice to update your OE tree at least daily. To do this, run
$ git pull --rebase
(note: this must be done in the directory created by the checkout of openembedded. On this page, this directory is /stuff/openembedded, but my checkout generated a directory /stuff/openembedded. Check the name of your subdir, and use the name on your machine in the following examples)
Create local configuration

It's now time to create your local configuration. While you could copy the default local.conf.sample like this:
$ cd /stuff/
$ cp openembedded/conf/local.conf.sample build/conf/local.conf
$ vi build/conf/local.conf
It is actually recommended to start smaller and keep local.conf.sample in the background and add entries from there step-by-step as you understand and need them. Please, do not just edit build/conf/local.conf.sample but actually READ it (read it and then edit).
For building a .dev branch, in your local.conf file, you should have at least the following three entries. Example for the Angstrom distribution and the Openmoko gta01 machine:
BBFILES = "/stuff/openembedded/recipes/**/**.bb"
DISTRO = "angstrom-2010.x"
MACHINE = "beagleboard"
If you choose to install OE in your home directory, modify local.conf to refer to the OE paths as /home/

&lt;username&gt;

/ rather than ~/. It does not find the **.bb packages otherwise.
Setup the environment**

One of the four command sets below will need to be run every time you open a terminal for development. (You can automate this in ~/.profile, /etc/profile, or perhaps use a script to set the necessary variables for using BitBake.)
If you followed the recommendation above to use BitBake from Git:
$ export BBPATH=/stuff/build:/stuff/openembedded
$ export PATH=/stuff/bitbake/bin:$PATH
If you installed BitBake:
$ export BBPATH=/stuff/build:/stuff/openembedded
Alternative syntax for those using the tcsh shell (e.g FreeBSD):
$ setenv PATH "/stuff/bitbake/bin:"$PATH
$ setenv BBPATH "/stuff/build:/stuff/openembedded:"$BBPATH
Start building

The primary interface to the build system is the bitbake command (see the bitbake users manual). bitbake will download and patch stuff from the network, so it helps if you are on a well connected machine.
Note that you should issue all bitbake commands from inside of the build/ directory, or you should override TMPDIR to point elsewhere (by default it goes to tmp/ relative to the directory you run the tools in).
Here are some example invocations:
Building a single package (e.g. nano):
$ bitbake nano
Building package sets (e.g. task-base):
$ bitbake task-base
Special note for task-base: you may need additional setup for building this very one task. More details in ZaurusKernels
Building a group of packages and deploying them into a rootfs image:
GPE:
$ bitbake x11-gpe-image
X11:
$ bitbake x11-image
OPIE:
$ bitbake opie-image
(NOTE: kergoth says it will take around 10GB of disk space to build an opie or gpe image for one architecture.
sledge says: You can reduce it to ~4GB by INHERIT += "rm\_work")
(NOTE: if you are using your custom kernel - set "Use the ARM EABI to compile the kernel (AEABI)" option in "Kernel Features")
See the /stuff/openembedded/recipes/meta/ directory if you're curious about what meta/task and image targets exist.
Building a single package, bypassing the long parse step (and therefore its dependencies--use with care):
$ bitbake -b /stuff/openembedded/recipes/blah/blah.bb
See Useful targets for a description of some of the more useful meta-packages. You will typically need at least one of the base images (bootstrap-image, opie-image or gpe-image), and if and only if you're building for an OpenZaurus target requiring an installer image (such as C3000), an additional pivotboot-image.
Output of the build process (temporary files, log files and the binaries) all ends up in the tmp directory. Most interesting is probably the tmp/work/ directory. Just have a look around the DirectoryStructure.
Images generated by building package groups like opie-image or pivotboot-image are placed in the tmp/deploy/images/ directory. Individual ipkg packages are put in tmp/deploy/ipk.
Adding Packages

Create bbfile.
Try building it locally.
Fix eventual problems.
Send .bbfile or an OePatch to the openembedded-devel mailing list. Please note that changes should comply with the commit policy.
Problems

Try to solve problems first by checking that you have done everything right, that nothing has changed from this description and that you have the latest code (see GitPhraseBook). Look also in the log file (referenced in any error message you will receive). If you still have problems, try checking PossibleFailures and common build errors. Above links are dead, you can try the Category:FAQ. If problems persist, ask on IRC or in the openembedded mailing list.
The Openembedded metadata is changing constantly. This implies several things:
Once you have a "known good" version that works well on your system, keep it! To update, clone a new copy; don't overwrite that working version until it's known to be safe.
To resolve build problems, "git pull" is your good friend. Many times, the issues will already be fixed in the current tree.
Not all metadata updates cause the local caches to update correctly. Sometimes you'll need to remove the ".../tmp" work directory and rebuild from scratch.
Similar issues apply to the package sources you download.
Portability issues

Make sure to set TARGET\_OS to something other than linux in local.conf if your host isn't linux.
GNU extensions to tools are often required. Symlink GNU patch, make, and cp (from fileutils), chmod, sed, find, tar, awk into your OE development path.
FreeBSD 4 users: Perl 5.0 is too old. A more recent perl must be available as /usr/bin/perl. Unfortunately just having more recent perl in the path isn't good enough. Some scripts are hard-coded for /usr/bin/perl. You can test for which perl you're using by typing perl -v. see /usr/ports/UPDATING for instructions on updating perl. Don't forget to do a use.perl port as instructed in /usr/ports/UPDATING
FreeBSD users: Set BUILD\_OS in local.conf to freebsdN where N is your major version number. At least the cross gcc wants this.
FreeBSD users: The build process of glibc uses a very long command line at some places. Increase ARG\_MAX to at least 131072, by editing /usr/sys/sys/syslimits.h and recompile your kernel (and reboot).
Productivity notes

Use the interactive bitbake mode ("bitbake -i") to speed up work when debugging or developing .bb files. Remember to run "parse" at the prompt first. Go!
If you want to save some compile time or are interested in additional tweaks to local.conf take a look at the Advanced configuration page.