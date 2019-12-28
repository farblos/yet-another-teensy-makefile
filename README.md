# yet-another-teensy-makefile

Title says it all.

Unique selling points and other facts:

- Supports GNU/Linux only and requires GNU Make and GNU Bash

- Extracts and uses compiler flags, linker flags, and other
  useful information directly from Teensyduino's `board.txt`

- Keeps all that information overridable at different levels of
  granularity

- Comes with instructions and procedures to set up a minimum
  Teensy development environment from a full Arduino and
  Teensyduino installation

- Initially based on and inspired by
  https://github.com/apmorton/teensy-template and the makefile
  that comes with Teensyduino itself

- Tested with Arduino 1.8.10, Teensyduino 1.48, and Teensy 3.5

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
        hardware/tools/arm          # disable with NO_TEENSY_TOOLS=1
        hardware/tools/avr          # disable with NO_TEENSY_TOOLS=1
        hardware/tools/teensy*      # disable with NO_TEENSY_TOOLS=1
    ```

  In addition, target `install` copies the YAT makefile to the
  project base directory, overwriting any other file named
  `GNUmakefile` there, and configures the installation-time
  variables in it.

- (If you have specified `NO_TEENSY_TOOLS=1` above:)  
  Install Debian packages `gcc-arm-none-eabi`,
  `libnewlib-arm-none-eabi`, `libstdc++-arm-none-eabi-newlib` (or
  comparable packages on other distros).

- (If you have specified `NO_TEENSY_TOOLS=1` above:)  
  Build `teensy_loader_cli` from
  https://github.com/PaulStoffregen/teensy_loader_cli and copy it
  to `<teensy-base-dir>/hardware/tools`.

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
          TEENSY =		teensy40
        Teensy 3.6:
          TEENSY =		teensy36
    ```

From this variable, all following configuration variables are
configured to reasonable default values (at least as reasonable
as determined by Teensyduino's `board.txt`).  You can override
these default values freely and at all levels.  Remember, though,
that the compilation of the Teensy core libraries depends on your
board and on the menu-level configuration variables.  If you
change these, you must execute `make clean` to remove all
intermediate objects and libraries.

The remaining top-level configuration variables are:

- `LIBRARIES`  
  List of additional Teensyduino libraries to use in the project,
  like `LiquidCrystal` or `Wire`.  For example:

    ```make
        LIBRARIES = LiquidCrystal Wire
    ```

- `TARGET`  
  The base name of the generated ELF object and hex file.
  Defaults to the base name of the current working directory.

- `BUILD_DIR`  
  Build directory for intermediate objects and libraries.  Can be
  specified relatively to the current working directory.
  Defaults to value `build`.

- `SILENT`  
  Echo-suppressing at-sign.  Set to an empty string to echo the
  most important compiler and linker commands.  For example:

    ```
        make SILENT= all
    ```

#### Menu-Level Configuration Variables

Setting the menu-level configuration variables to some value
corresponds to choosing a Teensyduino-related menu entry in the
Arduino IDE:

- `T_USB`: USB type
- `T_SPEED`: CPU speed
- `T_OPT`: Optimization level
- `T_KEYS`: Keyboard layout

For every `T_SOMEMENU` menu-level variable described above there
is a makefile target `list-somemenu` that you can execute to get
a list of possible values for that variable.  For example:

```
    [foo]$ make list-opt | head -5
    Valid values for variable T_OPT on teensy35:
    Faster:
      T_OPT =		o2std
    Faster with LTO:
      T_OPT =		o2lto
```

The first entry from each of these lists is choosen as default
value for the corresponding menu-level configuration variable.

#### Low-Level Configuration Variables, Compiler Flags, and Executables

The low-level configuration variables are initialized by the YAT
makefile depending on the selected board and menu entries.  Each
of these could be overridden separately, resulting in a rather
fine-grained control over the build process.  See the VAT
makefile itself for a list of these.

From the low-level configuration variables the usual compiler and
linker flags are assembled: `CPPFLAGS`, `CFLAGS`, `LDFLAGS`, etc.
Since it is a bit tedious to redefine these completely just to
add, say, some own define or include path, there are a number of
hook variables for the most needed (?) items to be added to the
flags:

- `DEFINES`, `INCLUDES`  
  Included in `CPPFLAGS` and, as such, in all other compiler and
  assembler command lines.
  
- `LDLIBS`  
  Included in the final linker call.

Finally, there is as set of configuration variables for compiler
and other executables, such as `CC`, `CXX`, `LD`, etc.  These get
default values based on your board and whether you have specified
`NO_TEENSY_TOOLS` during installation.

#### General Notes on Configuration Variables

To dump the value of some configuration variable you can execute
`make echo.<config-var-name>` as shown in the following example:

```
    [foo]$ make list-opt | tail -2
    Smallest Code with LTO:
      T_OPT =   oslto

    [foo]$ make T_OPT=oslto echo.T_OPTIMIZE
    -Os -flto -fno-fat-lto-objects --specs=nano.specs
```

If you directly modify the YAT makefile, ensure to keep the type
("immediate" or "deferred") of the makefile variables that you
modify unchanged.  As a rule of thumb, all variables related to
files, directories, and paths should be defined as immediate
variables (`:=`), while all variables related to compiler flags,
etc. should be defined as deferred variables (`=`, `?=`).

### Targets

The YAT makefile provides the following top-level targets:

- `all` (equivalent to `$(TARGET).hex`)  
  Compiles the Teensy core library, all required additional
  Arduino libraries, the sources of you project.  Links them and
  converts them to a `hex` file.
  
- `upload` (depending on `$(TARGET).hex`)  
  Uploads the `hex` file to your Teensy board.

- `clean`  
  Removes any intermediate objects, libraries, and other results.

- `list-teensy`  
  List the IDs of the supported Teensy boards.

- `list-usb`, `list-speed`, `list-opt`, `list-keys`  
  List the IDs of the supported menu entries for your Teeny
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
