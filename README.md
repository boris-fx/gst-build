# gst-build

GStreamer [meson](http://mesonbuild.com/) based repositories aggregator

You can build GStreamer and all its modules at once using
meson and its [subproject](https://github.com/mesonbuild/meson/wiki/Subprojects) feature.

## Getting started

### Install git and python 3.5+

If you're on Linux, you probably already have these. On macOS, you can use the
[official Python installer](https://www.python.org/downloads/mac-osx/).

You can find [instructions for Windows below](#windows-prerequisites-setup).

### Install meson and ninja

Meson 0.54 or newer is required.

On Linux and macOS you can get meson through your package manager or using:

  $ pip3 install --user meson

This will install meson into `~/.local/bin` which may or may not be included
automatically in your PATH by default.

You should get `ninja` using your package manager or download the [official
release](https://github.com/ninja-build/ninja/releases) and put the `ninja`
binary in your PATH.

You can find [instructions for Windows below](#windows-prerequisites-setup).

### Build GStreamer and its modules

You can get all GStreamer built running:

```
meson build/
ninja -C build/
```

This will automatically create the `build` directory and build everything
inside it.

NOTE: On Windows, you *must* run this from inside the Visual Studio command
prompt of the appropriate architecture and version.

# Development environment

## Building the Qt5 QML plugin

If `qmake` is not in `PATH` and pkgconfig files are not available, you can
point the `QMAKE` env var to the Qt5 installation of your choosing before
running `meson` as shown above.

The plugin will be automatically enabled if possible, but you can ensure that
it is built by passing `-Dgst-plugins-good:qt5=enabled` to `meson`. This will
cause Meson to error out if the plugin could not be enabled. This also works
for all plugins in all GStreamer repositories.

## Uninstalled environment

gst-build also contains a special `uninstalled` target that lets you enter an
uninstalled development environment where you will be able to work on GStreamer
easily. You can get into that environment running:

```
ninja -C build/ uninstalled
```

If your operating system handles symlinks, built modules source code will be
available at the root of `gst-build/` for example GStreamer core will be in
`gstreamer/`. Otherwise they will be present in `subprojects/`. You can simply
hack in there and to rebuild you just need to rerun `ninja -C build/`.

NOTE: In the uninstalled environment, a fully usable prefix is also configured
in `gst-build/prefix` where you can install any extra dependency/project.

## Update git subprojects

We added a special `update` target to update subprojects (it uses `git pull
--rebase` meaning you should always make sure the branches you work on are
following the right upstream branch, you can set it with `git branch
--set-upstream-to origin/master` if you are working on `gst-build` master
branch).

Update all GStreamer modules and rebuild:

```
ninja -C build/ update
```

Update all GStreamer modules without rebuilding:

```
ninja -C build/ git-update
```

## Custom subprojects

We also added a meson option, `custom_subprojects`, that allows the user
to provide a comma-separated list of subprojects that should be built
alongside the default ones.

To use it:

```
cd subprojects
git clone my_subproject
cd ../build
rm -rf * && meson .. -Dcustom_subprojects=my_subproject
ninja
```

## Run tests

You can easily run the test of all the components:

```
meson test -C build
```

To list all available tests:

```
meson test -C build --list
```

To run all the tests of a specific component:

```
meson test -C build --suite gst-plugins-base
```

Or to run a specific test file:

```
meson test -C build/ --suite gstreamer gst_gstbuffer
```

Run a specific test from a specific test file:

```
GST_CHECKS=test_subbuffer meson test -C build/ --suite gstreamer gst_gstbuffer
```

## Optional Installation

`gst-build` has been created primarily for [uninstalled usage](#uninstalled-environment),
but you can also install everything that is built into a predetermined prefix like so:

```
meson --prefix=/path/to/install/prefix build/
ninja -C build/
meson install -C build/
```

Note that the installed files have `RPATH` stripped, so you will need to set
`LD_LIBRARY_PATH`, `DYLD_LIBRARY_PATH`, or `PATH` as appropriate for your
platform for things to work.

## Checkout another branch using worktrees

If you need to have several versions of GStreamer coexisting (e.g. `master` and `1.14`),
you can use the `checkout-branch-worktree` script provided by `gst-build`. It allows you
to create a new `gst-build` environment with new checkout of all the GStreamer modules as
[git worktrees](https://git-scm.com/docs/git-worktree).

For example to get a fresh checkout of `gst-1.14` from a `gst-build` in master already
built in a `build` directory you can simply run:

```
./checkout-branch-worktree ../gst-1.14 1.14 -C build/
```

## Add information about GStreamer development environment in your prompt line

### Bash prompt

We automatically handle `bash` and set `$PS1` accordingly.

If the automatic `$PS1` override is not desired (maybe you have a fancy custom prompt), set the `$GST_BUILD_DISABLE_PS1_OVERRIDE` environment variable to `TRUE` and use `$GST_ENV` when setting the custom prompt, for example with a snippet like the following:

```bash
...
if [[ -n "${GST_ENV-}" ]];
then
  PS1+="[ ${GST_ENV} ]"
fi
...
```

### Using powerline

In your powerline theme configuration file (by default in
`{POWERLINE INSTALLATION DIR}/config_files/themes/shell/default.json`)
you should add a new environment segment as follow:

```
{
  "function": "powerline.segments.common.env.environment",
  "args": { "variable": "GST_ENV" },
  "priority": 50
},
```

## Windows Prerequisites Setup

On Windows, some of the components may require special care.

### Git for Windows

Use the [Git for Windows](https://gitforwindows.org/) installer. It will
install a `bash` prompt with basic shell utils and up-to-date git binaries.

During installation, when prompted about `PATH`, you should select the
following option:

![Select "Git from the command line and also from 3rd-party software"](/data/images/git-installer-PATH.png)

### Python 3.5+ on Windows

Use the [official Python installer](https://www.python.org/downloads/windows/).
You must ensure that Python is installed into `PATH`:

![Enable Add Python to PATH, then click Customize Installation](/data/images/py-installer-page1.png)

You may also want to customize the installation and install it into
a system-wide location such as `C:\PythonXY`, but this is not required.

### Ninja on Windows

The easiest way to install Ninja on Windows is with `pip3`, which will download
the compiled binary and place it into the `Scripts` directory inside your
Python installation:

```
pip3 install ninja
```

You can also download the [official release](https://github.com/ninja-build/ninja/releases)
and place it into `PATH`.

### Meson on Windows

**IMPORTANT**: Do not use the Meson MSI installer since it is experimental and known to not
work with `gst-build`.

You can use `pip3` to install Meson, same as Ninja above:

```
pip3 install meson
```

Note that Meson is written entirely in Python, so you can also run it as-is
from the [git repository](https://github.com/mesonbuild/meson/) if you want to
use the latest master branch for some reason.
