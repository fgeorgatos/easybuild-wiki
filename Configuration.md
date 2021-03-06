# EasyBuild configuration

(this page discusses the new style of configuring EasyBuild, which is supported since EasyBuild v1.3.0; for the legacy way of configuring EasyBuild, see [here](https://github.com/hpcugent/easybuild/wiki/Configuration-legacy))

Configuring EasyBuild can be done by:
* calling `eb` with **command line arguments** to tweak the configuration
* setting **environment variables**
* providing a **configuration file**

Of course, combining any of these types of configuration works too (and is even fairly common).

The order of preference for the different configuration types is a listed above, i.e., environment variables override the corresponding entries in the configuration file,
while command line arguments in turn override the corresponding environment variables _and_ matching entries in the configuration file.

## Available configuration settings

### Mandatory configuration settings

## Different configuration options

### Legacy configuration options (deprecated)

In EasyBuild v1.x, a couple of configuration options other than the ones below are available that follow the **legacy configuration style**, including:
* the `-C` and `--config` command line arguments
* the `$EASYBUILDCONFIG` environment variable
* the default path `$HOME/.easybuild/config.py`
* the legacy fallback path `<installpath>/easybuild/easybuild_config.py`

_**We _strongly_ advise to switch to the new way of configuring EasyBuild as soon as possible, since the legacy style will no longer be supported in EasyBuild v2.x.**_

### Configuration file

The EasyBuild configuration file follows the default Python configuration format as parsed by the `configparser` module (see [http://docs.python.org/2/library/configparser.html](http://docs.python.org/2/library/configparser.html)).

The set of configuration files that will be used by EasyBuild is determined in the following order of preference:
* the path(s) specified via **command line argument `--configfiles`**
* the path(s) specified via the **`$EASYBUILD_CONFIGFILES` environment variable**
* the **default path** for the EasyBuild configuration file, i.e. `$HOME/.easybuild/config.cfg`

On top of this, the command line argument `--ignoreconfigfiles` allows to specify configuration files that should be _ignored_ by EasyBuild (regardless of whether they are specified via any of the options above).

## Environment variables

## Command line arguments


## Configuration variables

The configuration file must define the following five variables: `build_path`, `install_path`, `source_path`, `repository`, and `log_format`.
If one of them is not defined, EasyBuild will complain and exit.


<a name="wiki-build_path">
### Build path (required)

The `build_path` variable specifies the directory in which EasyBuild builds its software packages.

Each software package is (by default) built in a subdirectory of the `build_path` under `<name>/<version>/<toolkit><versionsuffix>`.

Note that the build directories are emptied by EasyBuild when the installation is completed (by default).


<a name="wiki-install_path">
### Install path (required)

The `install_path` variable specifies the directory in which EasyBuild installs software packages and the corresponding module files.

The packages themselves are installed under `install_path/software` in their own subdirectory aptly named `<name>/<version>-<toolkit><versionsuffix>`
(by default), where name is the package name. The corresponding module files are installed under `install_path/modules`.

**Setting MODULEPATH**

After the configuration, you need to make sure that `MODULEPATH` environment variable is extended with the `modules/all` subdirectory of the `install_path`, i.e.:

```bash
export MODULEPATH=<install_path>/modules/all:$MODULEPATH
```

It is probably a good idea to add this to your (favourite) shell .rc file, e.g.,  `.bashrc`, and/or the `.profile` login scripts,
so you do not need to adjust the `MODULEPATH` variable every time you start a new session.


<a name="wiki-source_path">
### Source path (required)

The `source_path` variable specifies the directory in which EasyBuild looks for software source and install files.

Similarly to the configuration file lookup, EasyBuild looks for the installation files as given by the `sources` variable
in the .eb easyconfig file, in the following order of preference:

* `<source_path>/<name>`: a subdirectory determined by the name of the software package
* `<source_path>/<letter>/<name>`: in the style of the `easyblocks`/`easyconfigs` directories:
  in a subdirectory determined by the first letter (in lower case) of the software package and by its full `name`
* `<source_path>`: directly in the source path

Note that these locations are also used when EasyBuild looks for patch files in addition to the various `easybuild/easyconfigs`
directories that are listed in the PYTHONPATH.


<a name="wiki-repository">
### Easyconfigs repository (required)

EasyBuild has support for keeping track of (tested) .eb easyconfigs. These files are build specification files for software package installation.
After successfully installing a software package using EasyBuild, the corresponding .eb file is uploaded to a repository defined by the `repository` configuration variable.

Currently, EasyBuild supports the following repository types:

* `FileRepository('path', 'subdir/path'))`: a plain flat file repository; `path` is the path where files will be stored.
* `GitRepository('path', 'path/in/repository'`: a _non-empty_ **bare** git repository (created with `git init --bare` or `git clone --bare`);
   `path` is the path to the git repository (can also be a URL), `path/in/repository` is a path inside the repository where to save the files
* `SvnRepository('path')`: an SVN repository; `path` contains the subversion repository location, again, this can be a directory or a URL

You need to set the `repository` variable inside the config like so:
```python
repository = FileRepository(path)
```

Or, optionally a subdir argument can be specified:

```python
repository = FileRepository(repositoryPath, subdir)
```

You don't have to worry about importing these classes, EasyBuild will make them available to the config file.

Using `git` requires the `GitPython` Python modules, using `svn` requires the `pysvn` Python module (see [[Dependencies]]).

If access to the easyconfigs repository fails for some reason (e.g., no network or a required Python module), EasyBuild will
issue a warning. The software package will still be installed, but the (successful) easyconfig will not be automatically added to the repository.


<a name="wiki-log_format">
### Log format (required)

The `log_format` variable contains a tuple specifying a log directory name and a template string. In both of these values, using the following fields is supported:

* `name`: the name of the software package to install
* `version`: the version of the software package to install
* `date`: the date on which the installation was performed (in `YYYYMMDD` format, e.g. `20120324`)
* `time`: the time at which the installation was started (in `HHMMSS` format, e.g. `214359`)

Example :

```python
log_format = ("easylog", "easybuild-%(name)s.log")
```

<a name="wiki-install_suffixes">
### Software and modules install path suffixes

(supported since v1.1.0)

By default, EasyBuild will use the install path suffix `software` for the software installations, and the suffix `modules` for the generated module files.

These defaults can be adjusted by defining the `software_install_suffix` and/or `modules_install_suffix` variables in the configuration file, e.g.:

```python
import os
software_install_suffix = os.path.join('easybuild', 'installs')
modules_install_suffix = os.path.join('easybuild', 'modules')
```

Note: EasyBuild will still use the additional `all` and `base` suffixes for the module install paths, along with a directory for every module class that's being used.

<a name="wiki-example_config">
## Example configuration

This is a simple example configuration file, that specifies the user's home directory as the `install_path` and  that uses /tmp/easybuild as the `build_path`.
Additionally, it states that the .eb repository resides in $HOME/ebfiles and that it stores flat files:

```python
import os

home = os.getenv('HOME')

build_path = "/tmp/easybuild"
install_path = home
source_path = os.path.join(home, "sources")

repository_path = os.path.join(home, "ebfiles")
repository = FileRepository(repository_path)

log_format = ("easybuildlogs", "%(name)s-%(version)s.log")

module_classes = ['base', 'bio', 'chem', 'compiler', 'lib', 'phys', 'tools']
```


## Default configuration

The default configuration file that comes with EasyBuild (`easybuild/easybuild_config.py`) can be used as a starting point if you create your own configuration file,
see [here](https://github.com/hpcugent/easybuild-framework/tree/master/easybuild/easybuild_config.py).


### Paths

In the default configuration file, the prefix for the build, install and source directories is obtained from the `EASYBUILDPREFIX` environment variable.
If this variable was not defined, the prefix is set to $HOME/.local/easybuild/

If for some reason `HOME` is not defined, the configuration file falls back to using `/tmp/easybuild` as the prefix for the various paths.

These paths are then defined as follows:

* `build_path`: `<prefix>/easybuild/build`
* `install_path`: `<prefix>/easybuild`
* `source_path`: `<prefix>/easybuild/sources`


### Easyconfigs repository

The `repository_path` is specified as `<prefix>/ebfiles_repo`, and
`repository` is set to `FileRepository`


### Log format

The default log format uses all fields available:

```python
("easybuildlog", "easybuild-%(name)s-%(version)s-%(date)s.%(time)s.log")
```

## Reconfiguration using environment variables

You can (temporarily) override both the `build_path`, `source_path` and `install_path` settings by defining the `EASYBUILDBUILDPATH`,
`EASYBUILDSOURCEPATH` and `EASYBUILDINSTALLPATH` environment variables.

Overriding the configuration file is commonly done when testing new easyblocks.
