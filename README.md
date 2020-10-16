[![Build Status](https://travis-ci.org/GiovanniBussi/macports-ci.svg?branch=master)](https://travis-ci.org/GiovanniBussi/macports-ci)
[![CI](https://github.com/GiovanniBussi/macports-ci/workflows/CI/badge.svg)](https://github.com/GiovanniBussi/macports-ci/actions?query=workflow%3ACI)

# Travis-CI MacPorts

[Travis-ci](https://travis-ci.org) is great and allows you to test your [GitHub](https://github.com) projects on OSX.
However, OSX images on Travis-ci only come with HomeBrew installed.
If, like me, you want to use [MacPorts](https://www.macports.org/),
you should install it on the fly everytime.
For a few projects I am working on I copied around scripts installing MacPorts. I now decided to share here a generic script called `macports-ci` that could be used in multiple projects.
The script is checked frequently on [Travis-ci](https://travis-ci.org/GiovanniBussi/macports-ci),
so you should be able to see from the status image above whether it is working or not.
I recommend using `curl -LO` as explained below to always use the latest version from the `master` branch.
In case I have to test some feature, I will do it on separate branches so as to keep `master` branch stable.

Installing MacPorts
-------------------

Put the following commands in your `.travis.yml` file:

     - curl -LO https://raw.githubusercontent.com/GiovanniBussi/macports-ci/master/macports-ci
     - source ./macports-ci install

If you prefer, you can directly include the `macports-ci` file in your github repository. However, I recommend always using `curl -LO` to obtain
the latest version.

Notice that in the original version of this script it was necessary to also set some enviroment variables (`COLUMNS` and `PATH`). In case you execute the script in the old way (`./macports-ci`) or in case you execute it in a separate shell you might have to take care of these variables.
See notes below.
Also notice that as of July 15, 2020, when used in a GitHub workflow, the script also exports the variable using [::set-env](https://docs.github.com/en/actions/reference/workflow-commands-for-github-actions#setting-an-environment-variable). This means that you can install MacPorts in one step and use it in the following steps. As of October 17, this has been fixed to use [environment files](https://docs.github.com/en/free-pro-team@latest/actions/reference/workflow-commands-for-github-actions#environment-files) (see [issue 14](https://github.com/GiovanniBussi/macports-ci/issues/14)).

The `source macports-ci install` script can be run with a few options:

     --prefix=/other/prefix
         Install MacPorts in a different location. Default is `/opt/local`.
     --source
         Install MacPorts from source (slower). It is implicit when
         using `--prefix` with a prefix different from `/opt/local`.
     --binary
         Install MacPorts from a pkg (faster, this is the default).
     --version=2.4.1
         Require a specific MacPorts version. Notice that
         MacPorts is anyway updated with a `selfupdate` command.
     --sync=rsync
         Specify sync mode. Mode can be either `rsync`,
         `tarball` (default), or `github`.
     --keep-brew
         Do not uninstall HomeBrew (currently this is the default)
     --remove-brew
         Uninstall HomeBrew (experimental)
 
 Notice that the `source macports-ci install` script will take care of a number of things, including:
 
 - Uninstalling HomeBrew (only if `--remove-brew` option is used). This is an experimental feature. Notice that
   travis relies on some library installed by HomeBrew so some feature might not work (I noticed a problem in saving caches).
 - Finding which version of OSX you have so as to identify the correct MacPorts image
   when using binary installation from pkg.
 - Trying to use `port selfupdate` multiple times until it succeeds.
 - Fixing the environment setting `COLUMNS` to the appropriate value (except if you execute the script in the old way, see below).
 - Fixing the environment prepending `/opt/local/bin` to the `PATH` (except if you execute the script in the old way, see below). This will make sure that the `port` command is in the path.
 

Using your local Portfiles
-------------------------------

In case you just need ports to install tools that are used in your project,
you are done. If you want to also setup a 
[local Portfile repository](https://guide.macports.org/chunked/development.local-repositories.html) to test your own Portfiles you might do the following:

     - source macports-ci localports path/to/local/port/dir

After this command, you will be able to install packages described in your local Portfiles using commands such as `port install portname`.
Notice that local Portfiles will take the precedence with respect to official Portfiles.

Enabling ccache
---------------

Ccache could be used to significantly speed up your builds, especially if you build similar sources multiple times. However, notice that it will add an initial slowdown to install ccache itself. This would be more significant if you install on a non-default prefix, such that ccache itself (and its required libraries) are installed from source. Thus, using ccache is not recommended when building on non-default prefix.

In case you want to enable ccache to be available to `port install` commands, you should modify your `.travis.yml` file following these instructions. First, add the following command at the beginning of the configuration file

````
cache:
  directories:
  - $HOME/.macports-ci-ccache
````    

This will inform Travis-ci that there is a directory to be cached. Then, add the following command after you installed MacPorts and before executing `port install` commands:

    - source macports-ci ccache

This will install ccache, retrieve the cache, and make sure ccache is used for compiling further packages. 
Finally, add the following command after you executed the relevant `port install` commands:

    - source macports-ci ccache --save

This will store the ccache cache in a place where it can be seen by Travis-ci. 

Notice that caching the `$HOME/.ccache` directory as explained in 
[this page](https://docs.travis-ci.com/user/caching/) is not required nor sufficient.

Executing the script in the old way
-------------------

Originally this script was designed to be executed as `./macports-ci` (rather than `source macports-ci`). Executing the script has some limitations. In particular, you should adjust the environment variables `COLUMNS` and `PATH` for the next commands to work correctly. A typical set of instructions was:

````
- export COLUMNS=80
- curl -LO https://raw.githubusercontent.com/GiovanniBussi/macports-ci/master/macports-ci
- chmod +x ./macports-ci
- ./macports-ci install
- export PATH="/opt/local/bin:$PATH"
````

As of April 30, 2018 it is possible to source the script rather than executing it:
     
     - curl -LO https://raw.githubusercontent.com/GiovanniBussi/macports-ci/master/macports-ci
     - source ./macports-ci install

As of July 24, 2018, sourcing the script set both the `COLUMNS` and the `PATH` variables to the correct values. At this point, sourcing the script rather then executing it is recommended. **Executing the script in the old way will still be supported.**

Notice that if instead of running the source command directly in your `.travis.yml` file you decide to include it in some external auxiliary scripts, you might still need to set the environment variables `COLUMNS` and `PATH` in the `.travis.yml` file.


Real life usage
---------------

You can find some sample usage in the `.travis.yml` files of these repositories:

- [plumed/plumed2](http://github.com/plumed/plumed2)
- [plumed/ports](http://github.com/plumed/ports)

Feedbacks
---------

I would like to keep this script up to date and to make it as robust as possible so that other projects can just use it without too much worries.
In case you find any problem or you think you can improve this script, please open an
[issue](https://github.com/GiovanniBussi/macports-ci/issues/new)
or a [pull request](https://github.com/GiovanniBussi/macports-ci/pulls).

Todo
----

There are a few additional improvements that could be implemented:

- Allow users to cache the `/opt/local` directory. I am afraid this is not feasible, since MacPorts also does a number of other things under the hood that are difficult to track (e.g. it adds a new user).
- Make use of `sudo` command optional. Inside `./macports-ci`, need of `sudo` could be detected trying to touch a file in the prefix path.
- Personalize ccache, e.g. allowing the user to specify cache size.

