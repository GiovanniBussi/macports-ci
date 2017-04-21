[![Build Status](https://travis-ci.org/GiovanniBussi/macports-ci.svg?branch=master)](https://travis-ci.org/GiovanniBussi/macports-ci)

# Travis-CI MacPorts

[Travis-ci](https://travis-ci.org) is great and allows you to test your [GitHub](https://github.com) projects on OSX.
However, OSX images on Travis-ci only come with Homebrew installed.
If, like me, you want to use [MacPorts](https://www.macports.org/),
you should install it on the fly everytime.
For a few projects I am working on I copied around scripts installing MacPorts. I now decided to share here a generic script called `macports-ci` that could be used in multiple projects.
The script is checked frequently on [Travis-ci](https://travis-ci.org/GiovanniBussi/macports-ci),
so you should be able to see from the status image above whether it is working or not.

Installing MacPorts
-------------------

Put the following commands in your `.travis.yml` file:

     - export COLUMNS=80
     - wget https://raw.githubusercontent.com/GiovanniBussi/macports-ci/master/macports-ci
     - chmod +x ./macports-ci
     - ./macports-ci install
     - PATH="/opt/local/bin:$PATH"

If you prefer, you can directly include the `./macports-ci` file in your github repository. However, I recommend always using `wget` to obtain
the latest version. Setting the number of columns with `export COLUMNS=80` is required for MacPorts to be able to download some distfiles.

The `./macports-ci install` script can be run with a few options:

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
         Specify sync mode. Mode can be either `tarball` (default),
         `rsync`, or `github`.
 
 Notice that the `./macports-ci install` script will take care of a number of things, including:
 
 - Finding which version of OSX you have so as to identify the correct MacPorts image
   when using binary installation from pkg.
 - Fixing the configuration file so as to make `port selfupdate` more reliable on Travis-ci (current default: `tarball`).
 
**It will not fix your executation path**, which should be adjusted by hand with `PATH="/opt/local/bin:$PATH"`
     
      

Using your local Portfiles
-------------------------------

In case you just need ports to install tools that are used in your project,
you are done. If you want to also setup a 
[local Portfile repository](https://guide.macports.org/chunked/development.local-repositories.html) to test your own Portfiles you might do the following:

     - ./macports-ci localports path/to/local/port/dir

After this command, you will be able to install packages described in your local Portfiles using commands such as `port install portname`.
Notice that local Portfiles will take the precedence with respect to official Portfiles.

Feedbacks
---------

I would like to keep this script up to date and to make it as robust as possible so that other projects can just use it without too much worries.
In case you find any problem or you think you can improve this script, please open an
[issue](https://github.com/GiovanniBussi/macports-ci/issues/new)
or a [pull request](https://github.com/GiovanniBussi/macports-ci/pulls).

Todo
----

There are a few additional improvements that could be implemented:

- Making the `port sync` procedure used within `./macports-ci install` more robust. I occasionally find problems with MacPorts mirrors. The current choice (sync from tarball) seems the most robust, but perhaps the could be improved trying multiple approaches until one of them works.
- Allow users to cache the `/opt/local` directory. This could be made by giving to `./macports-ci install` the path to a [cached directory](https://docs.travis-ci.com/user/caching) that could be used to restore the tree. `./macports-ci install` could then use `tar cjf $cachedir/macports.tbz2 $MACPORTS_PREFIX`. I should first check whether the `/opt/local` directory is small enough for this procedure to give some benefit.
- Make use of `sudo` command optional. Inside `./macports-ci`, need of `sudo` could be detected trying to touch a file in the prefix path.