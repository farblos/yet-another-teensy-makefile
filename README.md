# yet-another-teensy-makefile

Title says it all.

Unique selling points and other facts:

- Supports GNU/Linux only and requires GNU Make and GNU Bash

- Extracts and uses compiler flags, linker flags, and other
  useful information directly from Teensyduino's `board.txt`

- Keeps all that information overridable at different levels of
  granularity

- Handles the assembler files that come in recent Teensy core
  libraries

- Natively supports upload with `tycmd` from
  [Koromix/tytools](https://github.com/Koromix/tytools) (no big
  deal, but looks nice in this list, anyway)

- Comes with instructions and procedures to set up a minimum
  Teensy development environment from a full Arduino and
  Teensyduino installation

- Initially based on and inspired by
  [apmorton/teensy-template](https://github.com/apmorton/teensy-template)
  and the makefile that comes with Teensyduino itself

- Tested with Arduino 1.8.10, Teensyduino 1.48, and Teensy 3.5,
  but should be usable for all Teensy boards declared in
  Teensyduino's `boards.txt`

- Tested on Debian "stretch"

# Installation

### Directory Structure

The installation process below assume the following directory
structure, where the first one is required as input and the
remaining two are configured as output of the process:

- `arduino-base-dir` contains a full Arduino and Teensyduino
  installation
  
- `teensy-base-dir` contains a minimum Teensy development
  environment consisting of some Teensyduino-specific tools and
  sources

- `project-base-dir` contains your Teensy project sources

You can use one directory both as Teensy base directory and
project base directory, but should keep in mind that the
installation instructions below completely recreate
`teensy-base-dir`, discarding any previous contents.

If you have these directories already set up in some different
fashion, feel free to skip below installation instructions and
update the variables tagged "installation-time variables" in the
YAT makefile directly as needed.

### Installation Instructions

- Install Arduino and Teensyduino into the Arduino base directory
  `arduino-base-dir` according to their respective installation
  instructions.

- Create the Teensy base directory `teensy-base-dir` and prepare
  the project base directory `project-base-dir` with the
  following make invocation:
  
    ```
        make -f <path-to-the-yat-makefile>            \
                ARDUINO_BASE_DIR=<arduino-base-dir>   \
                TEENSY_BASE_DIR=<teensy-base-dir>     \
                PROJECT_BASE_DIR=<project-base-dir>   \
                install
    ```

  where all `*-base-dir` placeholders must be specified as
  absolute directory paths, like in the following example:
  
    ```
        make -f ~/git/yet-another-teensy-makefile/GNUmakefile \
                ARDUINO_BASE_DIR=~/arduino-1.8.10             \
                TEENSY_BASE_DIR=~/teensy-1.48                 \
                PROJECT_BASE_DIR=~/teensy-projects            \
                install
    ```

  Target `install` copies the following files and directories
  from the Arduino base directory to the newly created Teensy
  base directory:
  
    ```
        examples/Teensy
        hardware/teensy/avr
        hardware/tools/teensy*
        hardware/tools/arm          # disable with NO_TEENSY_GCC=arm or ...=all
        hardware/tools/avr          # disable with NO_TEENSY_GCC=avr or ...=all
    ```

  In addition, target `install` copies the YAT makefile to the
  project base directory, renaming any other file named
  `GNUmakefile` there to `GNUmakefile.bak`, and configures the
  installation-time variables in it.

- (If you have specified `NO_TEENSY_GCC=arm` or `...=all` above
  and want to compile for a Teensy 3.x or 4.x board):  
  Install Debian packages `gcc-arm-none-eabi`,
  `libnewlib-arm-none-eabi`, `libstdc++-arm-none-eabi-newlib`.
  Other distros may provide similar packages.
  
- (If you have specified `NO_TEENSY_GCC=avr` or `...=all` above
  and want to compile for a Teensy 2.x board):  
  Install Debian packages `gcc-avr`, `avr-libc`.  Other distros
  may provide similar packages.

- After executing above steps, the Arduino base directory is no
  longer required for using the YAT makefile and can be removed.

# Usage

To use the YAT makefile in one of your projects you must
configure the variables described in the following sections.  You
can do so either by directly modifying them in the YAT makefile
or by defining them in your own project-specific makefile that,
as a last statement, includes the YAT makefile.

The latter approach is somewhat cleaner and hopefully more
scalable, in particular if used in a directory structure like
this:

```
    <project-base-dir>
    <project-base-dir>/GNUmakefile      # the YAT makefile

    <project-base-dir>/foo              # project foo
    <project-base-dir>/foo/GNUmakefile  # foo-specific makefile
    <project-base-dir>/foo/bar.ino

    <project-base-dir>/baz              # project baz
    <project-base-dir>/baz/GNUmakefile  # baz-specific makefile
    <project-base-dir>/baz/buz.ino
```
  
where the project-specific makefiles are structured like this:

```make
    TEENSY = teensy35
    # ... more configuration variables ...
    include ../GNUmakefile
```

If you decide to directly modify the YAT makefile, keep in mind
that target `install` overwrites any previously present YAT
makefile in your project base directory.  (It keeps one backup of
the old YAT makefile, though.)

### Configuration Variables

There are three levels of configuration variables: top-level,
menu-level, and low-level configuration variables.  From all
these, you really must configure only one variable:

- `TEENSY`  
  The ID of your Teensy board.  Execute `make list-teensy` to get
  a list of supported values.  For example:

    ```
        [foo]$ make list-teensy | head -5
        Valid values for variables TEENSY:
        Teensy 4.0:
            TEENSY = teensy40
        Teensy 3.6:
            TEENSY = teensy36
    ```

From this variable, all following variables related to the Teensy
configuration are preset to reasonable default values (at least
as reasonable as determined by Teensyduino's `board.txt`).  You
can override these default values freely and at all levels, but
you should not forget that it may be required to rebuild the
Teensy core library and your project when changing Teensy
configuration.  See also configuration variable `DEP_MODEL`.

The remaining top-level configuration variables are:

- `LIBRARIES`  
  List of additional Teensyduino libraries to use in a project,
  like `LiquidCrystal` or `Wire`.  For example:

    ```make
        LIBRARIES = LiquidCrystal Wire
    ```

- `UPLOAD_TOOL`  
  The tool used to upload the hex file to your Teensy board.  One
  of `tycmd`, `tlcli` (for `teensy_loader_cli` from
  [PaulStoffregen/teensy_loader_cli](https://github.com/PaulStoffregen/teensy_loader_cli)),
  or `tlgui` (the vanilly Teensy loader).  To use one of the
  former two you must have the respective tool available in your
  execution path.  The default for this variable is based on what
  is found in the execution path during execution of `make
  install`.

- `DEF_TARGET`  
  The default target triggered by a plain `make` or a `make all`.
  Reasonable values are `build` (the default) and `upload`.

- `DEP_MODEL`  
  The dependency model used by the YAT makefile.
  
  As mentioned above, the choice of your Teensy board and the
  values of the menu-level configuration variables described
  below influence the compiled result.  You can configure this
  variable to let the YAT makefile detect changes in your choice
  and recompile sources as required:
  
  Dependency model `makefiles` (the default) triggers a complete
  rebuild if any of the makefiles of the project changes.  This
  model is suitable if you completely maintain your project
  configuration in the YAT makefile or some project-specific
  makefile.  Whenever you update any of these to change, say, the
  USB type of your project, all sources will be recompiled.

  Dependency model `variables` triggers a complete rebuild if the
  value of any of the variables listed in configuration variable
  `VAR_DEPS` changes.  (Which, by default, comprises variable
  `TEENSY` and all menu-level configuration variables.)  This
  model is suitable if you frequently change configuration
  variables, probably even at command line level as in `make
  T_USB=serialhid all`.

  Dependency model `always` always forces a complete rebuild.
  
  Dependency model `traditional` does not introduce any
  additional artificial dependencies and, hence, lets the YAT
  makefile rely on the usual object-source-header dependencies
  only.

- `SILENT`  
  Echo-suppressing at-sign.  Set to an empty string to echo the
  most important compiler and linker commands during their
  execution.  For example:

    ```
        make SILENT= all
    ```

There are more top-level configuration variables than those
described above, but these are less significant.  See the section
tagged "top-level configuration variables" in the YAT makefile
itself for a list of these.

#### Menu-Level Configuration Variables

Setting the menu-level configuration variables to some value
corresponds to choosing a Teensyduino-related menu entry in the
Arduino IDE:

| Variable  | Menu Entry         |
|:----------|:-------------------|
| `T_USB`   | USB type           |
| `T_SPEED` | CPU speed          |
| `T_OPT`   | Optimization level |
| `T_KEYS`  | Keyboard layout    |

For every `T_SOMEMENU` menu-level variable listed above there is
a makefile target `list-somemenu` that you can execute to get a
list of possible values for that variable.  For example:

```
    [foo]$ make list-opt | head -5
    Valid values for variable T_OPT on teensy35:
    Faster:
        T_OPT = o2std
    Faster with LTO:
        T_OPT = o2lto
```

The first entry from each of these lists is choosen as default
value for the corresponding menu-level configuration variable.

#### Low-Level Configuration Variables, Compiler Flags, and Executables

The low-level configuration variables are initialized by the YAT
makefile depending on the selected board and menu entries.  Each
of these could be overridden separately, resulting in a rather
fine-grained control over the build process.  See the section
tagged "low-level configuration variables" in the YAT makefile
itself for a list of these.

From the low-level configuration variables the usual compiler and
linker flags are assembled: `CPPFLAGS`, `CFLAGS`, `LDFLAGS`, etc.
Since it is a bit tedious to redefine these completely just to
add, say, some own define or include path, there are a number of
hook variables for the most needed (?) items to be added to these
flags:

- `DEFINES`, `INCLUDES`  
  Included in `CPPFLAGS` and, as such, in all other compiler and
  assembler command lines.
  
- `LDLIBS`  
  Included in the final linker call.

Finally, there is as set of configuration variables for compiler
and other executables, such as `CC`, `CXX`, `LD`, etc.  These get
default values based on your board and whether you have specified
`NO_TEENSY_GCC` during installation.

#### General Notes on Configuration Variables

To dump the value of some configuration variable you can execute
`make echo.<config-var-name>` as shown in the following example:

```
    [foo]$ make list-opt | tail -2
    Smallest Code with LTO:
        T_OPT = oslto

    [foo]$ make T_OPT=oslto echo.T_OPTIMIZE
    -Os -flto -fno-fat-lto-objects --specs=nano.specs
```

Regardless whether you directly modify the YAT makefile or
whether you include it from a project-specific one, ensure to
keep the type ("immediate" or "deferred") of the makefile
variables that you modify or define unchanged.  As a rule of
thumb, all variables related to files, directories, and paths
should be defined as immediate variables (`:=`), while all
variables related to compiler flags, etc. should be defined as
deferred variables (`=`, `?=`).

### Targets

The default (first) target in the YAT makefile is target `all`,
which by default is equivalent to target `build` described below.
You can change that default by configuring variable `DEF_TARGET`,
for example, to value `upload`.

The YAT makefile provides the following top-level targets:

- `build` (equivalent to `$(TARGET).hex`)  
  Compiles the Teensy core library, all required additional
  Teensyduino libraries, the sources of your project.  Links them
  and converts them to a hex file.
  
- `upload` (depending on `$(TARGET).hex`)  
  Uploads the hex file to your Teensy board.

- `clean`  
  Removes any intermediate objects, libraries, and other results.

- `list-teensy`  
  Lists the IDs of the supported Teensy boards.

- `list-usb`, `list-speed`, `list-opt`, `list-keys`  
  Lists the IDs of the supported menu entries for your Teensy
  board.  See also section
  [Menu-Level Configuration Variables](#menu-level-configuration-variables).

- `realclean` (depending on `clean`)  
  Removes in addition `boards.mk`, which is generated from the
  `boards.txt` file.

- `echo.<config-var-name>`  
  See section
  [General Notes on Configuration Variables](#general-notes-on-configuration-variables).

- `install`  
  Install a minimum Teensy development environment from a full
  Arduino and Teensyduino installation.  See section
  [Installation Instructions](#installation-instructions).

# Motivation and History

I dislike IDEs other than the One, so I started looking for
ready-made, Teensyduino-replacing makefiles.  But those I came
across were out-of-date and produced hex files which were larger
and different in structure compared to those generated by the
IDE.  So after some fiddeling I decided to use `strace` on the
Arduino IDE, extract the command lines from that, and put some
makefile around the command lines.

Later in that process I discovered that all that information on
compiler flags, etc. is actually contained in `boards.txt` and
restructured the makefile to extract the information directly
from there.

Still later I came across
[fkmclane/teensy-makefile](https://github.com/fkmclane/teensy-makefile),
which also accesses `boards.txt` to do its job, but not for all
Teensyduino's menus.
