#
# GNUmakefile - Yet Another Teensy Makefile.
#
# Copyright (c) 2019, 2020 Jens Schmidt.
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.
#
# See https://github.com/farblos/yet-another-teensy-makefile/README.md
# for documentation on variables and targets.
#

# force the use of Bash on Debian systems
SHELL :=	/bin/bash

#
# installation-time variables.
#
# These are normally instantiated by target "install".  Ensure to
# remove all trailing comments when you configure these manually.
#

TEENSY_BASE_DIR :=	### Teensy base directory

PROJECT_BASE_DIR :=	### Project base directory

NO_TEENSY_GCC :=	### Teensy-gcc deselection (empty, "avr", "arm", "all")

UPLOAD_TOOL_DEFAULT :=	### Upload tool default ("tycmd", "tlcli", "tlgui")

MONITOR_TOOL_DEFAULT :=	### Monitor tool default ("tycmd", "teensy")

ARDUINO_VERSION :=	### Ardunio version (sans periods)

TEENSYDUINO_VERSION :=	### Teensyduino version (sans periods)

#
# top-level configuration variables.
#
# Configuration variable TEENSY is so top-level that it is not
# even defined here.  See README.md for more information.
#

ifeq ($(origin LIBRARIES), undefined)
LIBRARIES :=	@included-libs@
endif

ifeq ($(origin UPLOAD_TOOL), undefined)
UPLOAD_TOOL :=	$(UPLOAD_TOOL_DEFAULT)
endif

ifeq ($(origin MONITOR_TOOL), undefined)
MONITOR_TOOL :=	$(MONITOR_TOOL_DEFAULT)
endif

ifeq ($(origin DEF_TARGET), undefined)
DEF_TARGET :=	build
endif

ifeq ($(origin USER_LIB_PATH), undefined)
USER_LIB_PATH :=libraries
endif

ifeq ($(origin DEP_MODEL), undefined)
DEP_MODEL :=	makefiles
endif

ifeq ($(origin SILENT), undefined)
SILENT :=	@
endif

# base name of the generated ELF object and hex file
ifeq ($(origin TARGET), undefined)
TARGET :=	$(notdir $(CURDIR))
endif

# build directory for intermediate objects and libraries, best
# specified relatively to the current working directory.  Target
# clean completely removes this directory.
ifeq ($(origin BUILD_DIR), undefined)
BUILD_DIR :=	build
endif

# variables influencing build results, required for dependency
# model "variables" only.  Any changes in the values of the
# variables listed here trigger a complete rebuild of all objects
# and libraries.  Must be defined as deferred variable.
VAR_DEPS ?=	$(TEENSY) $(T_USB) $(T_SPEED) $(T_OPT) $(T_KEYS)

#
# menu-level configuration variables.
#

T_USB ?=	$($(TEENSY).menu.usb.default)

T_SPEED ?=	$($(TEENSY).menu.speed.default)

T_OPT ?=	$($(TEENSY).menu.opt.default)

T_KEYS ?=	$($(TEENSY).menu.keys.default)

#
# low-level configuration variables.
#

# governed by TEENSY.  The global optimization flags in T_GLBLOPT
# are not used since T_OPTIMIZE is more specific.
T_COMMON ?=	$(call xtv,build.flags.common)
T_DEP ?=	$(call xtv,build.flags.dep)
T_GLBLOPT ?=	$(call xtv,build.flags.optimize)
T_CPU ?=	$(call xtv,build.flags.cpu)
T_DEFS ?=	$(call xtv,build.flags.defs)
T_C ?=		$(call xtv,build.flags.c)
T_CPP ?=	$(call xtv,build.flags.cpp)
T_S ?=		$(call xtv,build.flags.S)
T_LD ?=		$(call xtv,build.flags.ld)
T_LIBS ?=	$(call xtv,build.flags.libs)

# governed by T_USB
T_USBTYPE ?=	$(call xuv,build.usbtype)

# governed by T_SPEED
T_FCPU ?=	$(call xsv,build.fcpu)

# governed by T_OPT
T_OPTIMIZE ?=	$(call xov,build.flags.optimize)
T_LDSPECS ?=	$(call xov,build.flags.ldspecs)

# governed by T_KEYS
T_KEYLAYOUT ?=	$(call xkv,build.keylayout)

#
# compiler flags and options.
#

CPPFLAGS ?=	$(T_DEFS) -DARDUINO=$(ARDUINO_VERSION)				\
		-DF_CPU=$(T_FCPU) -D$(T_USBTYPE) -DLAYOUT_$(T_KEYLAYOUT)	\
		$(DEFINES)							\
		$(INCLUDES) -I. -I$(TCORE_DIR) $(addprefix -I, $(INC_PATH))

CFLAGS ?=	$(T_OPTIMIZE) $(T_COMMON) $(T_DEP) $(T_C) $(T_CPU)

CXXFLAGS ?=	$(T_OPTIMIZE) $(T_COMMON) $(T_DEP) $(T_CPP) $(T_CPU)

ASFLAGS ?=	$(T_OPTIMIZE) $(T_COMMON) $(T_DEP) $(T_S) $(T_CPU)

LDFLAGS ?=	$(T_OPTIMIZE) $(T_LD) $(T_CPU) $(T_LDSPECS)

#
# compiler and other executables.
#
# Some of these get default values assigned by make, so a plain
# conditional assignment with "?=" is not sufficient for these.
#

# Teensy gcc directory.  Must end in a slash if non-empty.
ifeq ($(NO_TEENSY_GCC),)
  T_GCC_DIR =	$(TOOLS_DIR)/$(call xtv,build.toolchain)
else ifeq ($(NO_TEENSY_GCC), arm)
  T_GCC_DIR =	$(filter-out %/arm/bin/, $(TOOLS_DIR)/$(call xtv,build.toolchain))
else ifeq ($(NO_TEENSY_GCC), avr)
  T_GCC_DIR =	$(filter-out %/avr/bin/, $(TOOLS_DIR)/$(call xtv,build.toolchain))
else
  T_GCC_DIR =
endif

ifeq ($(filter-out undefined default, $(origin CC)),)
CC =		$(T_GCC_DIR)$(call xtv,build.command.gcc)
endif

ifeq ($(filter-out undefined default, $(origin CXX)),)
CXX =		$(T_GCC_DIR)$(call xtv,build.command.g_plus_plus)
endif

ifeq ($(filter-out undefined default, $(origin AR)),)
AR =		$(T_GCC_DIR)$(call xtv,build.command.ar)
endif

OBJCOPY ?=	$(T_GCC_DIR)$(call xtv,build.command.objcopy)

LD ?=		$(T_GCC_DIR)$(call xtv,build.command.linker)

SIZE ?=		$(T_GCC_DIR)$(call xtv,build.command.size)

#
# auxilliary stuff.
#

BOARDS_TXT :=	$(TEENSY_BASE_DIR)/hardware/teensy/avr/boards.txt

# seconds since the epoch.  Required to define the RTC symbol in
# T_LD.
TIME_LOCAL =	$(shell date +'%s')

# awkified properties platform.txt/recipe.size.regex and
# platform.txt/recipe.size.regex.data
SIZE_RE_TEXT :=	^(\.text|\.text\.progmem|\.text\.itcm|\.data)[\t ]+([0-9]+).*
SIZE_RE_DATA :=	^(\.usbdescriptortable|\.dmabuffers|\.usbbuffers|\.data|\.bss|\.bss\.dma|\.noinit|\.text\.itcm)[\t ]+([0-9]+).*

# non-clean-or-install target predicate.  This variable is
# non-empty if the current make goal as specified on the
# commandline is different from clean, realclean, or install (or
# combinations thereof).  More clearly:
#
#   ifeq  ($(NCITP),)  <=>  "if   clean-or-install-target-p"
#   ifneq ($(NCITP),)  <=>  "if ! clean-or-install-target-p"
NCITP :=	$(strip						\
		  $(if $(MAKECMDGOALS),				\
		       $(filter-out clean realclean install,	\
		                    $(MAKECMDGOALS)),		\
		       all))

# resolves any special "@included-libs@" marker in the specified
# parameters.  Caller must strip the result.
#
# As an example, this function resolves includes
#
#   #include <TimeLib.h>
#   #include <Bounce.h>
#   #include <Snooze.h>
#
# from the current sources to the list "Time Bounce Snooze"
# because of the files
#
#   Time/TimeLib.h
#   Bounce/Bounce.h
#   Snooze/src/Snooze.h
#
# being present below the Teensy platform library directory.
#
# The call to "sort" is required to strip potential duplicate
# libraries present both in the user library path and in the
# Teensy platform library directory.  The reference to
# "/dev/null" is required to not let "sed" wait on STDIN when
# there are no source files present.
define rslv_inc_libs
  $(subst								\
    @included-libs@,							\
    $(foreach								\
      inc, $(shell sed -ne 's/^ *\# *include *[<"]\(.*\)\.h[>"]/\1/p'	\
                           $(SRC_FILES) /dev/null),			\
      $(sort								\
        $(foreach							\
          dir, $(USER_LIB_PATH) $(TPLTF_DIR),				\
          $(or $(patsubst $(dir)/%/src/$(inc).h, %,			\
                          $(wildcard $(dir)/*/src/$(inc).h)),		\
               $(patsubst $(dir)/%/$(inc).h, %,				\
                          $(wildcard $(dir)/*/$(inc).h)))))),		\
    $(1))
endef

# returns the first set of existing include directories from the
# specified list of library base directories.  Distinguishes
# between new-style and old-style libraries.  Caller must strip
# the result.
#
# Here are some examples assuming that the user library path
# equals "libraries", which contains one user library "Bounce",
# and the Teensy platform library directory equals "/teensylibs".
#
# Then this function converts (potentially absent) library base
# directories to (existing) include directories as follows:
#
#   libraries/Time   /teensylibs/Time   => /teensylibs/Time
#   libraries/Bounce /teensylibs/Bounce => libraries/Bounce
#   libraries/Snooze /teensylibs/Snooze => /teensylibs/Snooze/src
define lib_inc_dirs
  $(if
    $(strip $(1)),
    $(or
      $(if $(and $(wildcard $(firstword $(1))/src), $(wildcard $(firstword $(1))/library.properties)),
           $(firstword $(1))/src),
      $(if $(wildcard $(firstword $(1))),
           $(firstword $(1)) $(wildcard $(firstword $(1))/utility)),
      $(call lib_inc_dirs, $(wordlist 2, $(words $(1)), $(1)))))
endef

# returns the first set of existing library directories from the
# specified list of library base directories.  Distinguishes
# between new-style and old-style libraries.  Caller must strip
# the result.
#
# In the context of the above example, this function converts
# (potentially absent) library base directories to (existing)
# library directories as follows:
#
#   libraries/Time   /teensylibs/Time   => /teensylibs/Time
#   libraries/Bounce /teensylibs/Bounce => libraries/Bounce
#   libraries/Snooze /teensylibs/Snooze => /teensylibs/Snooze/src
#                                          /teensylibs/Snooze/src/hal
#                                          /teensylibs/Snooze/src/hal/...
define lib_lib_dirs
  $(if
    $(strip $(1)),
    $(or
      $(if $(and $(wildcard $(firstword $(1))/src), $(wildcard $(firstword $(1))/library.properties)),
           $(shell find "$(firstword $(1))/src" -type d)),
      $(if $(wildcard $(firstword $(1))),
           $(firstword $(1)) $(wildcard $(firstword $(1))/utility)),
      $(call lib_lib_dirs, $(wordlist 2, $(words $(1)), $(1)))))
endef

# exactly-one-teensy check
define EOTC
		@case $$( $(TOOLS_DIR)/teensy_ports -L | wc -l ) in				\
		  (1) : ;;									\
		  (0) echo "No Teensy board connected." 1>&2; exit 1 ;;				\
		  ([2-9]) echo "More than one Teensy board connected." 1>&2; exit 1 ;;		\
		  (*) echo "Unreasonable number of Teensy boards connected." 1>&2; exit 1 ;;	\
		esac
endef

# default target
.PHONY:		all
all:		$(DEF_TARGET)

# force target
.PHONY:		force
force:

# dump the value of a variable
.PHONY:		echo.%
echo.%:
		@echo '<$($*)>'

#
# verify configuration variable TEENSY and determine T_CORE.
#

ifeq ($(NCITP),)
  # no-op
else ifeq ($(MAKECMDGOALS), list-teensy)
  # no-op
else ifeq ($(origin TEENSY), undefined)
  $(error Undefined configuration variable TEENSY)
else ifeq ($(shell test -f "$(BOARDS_TXT)" && echo found),)
  $(error Inaccessible boards.txt file "$(BOARDS_TXT)")
else ifeq ($(shell grep '^$(TEENSY)\.name=' "$(BOARDS_TXT)" 2>/dev/null),)
  $(error Invalid configuration variable TEENSY ("$(TEENSY)" not found in boards.txt))
else
  # use function "shell" only this one time and the user-defined
  # boards.txt accessor functions for all other instances
  T_CORE :=	$(shell sed -n 's/^$(TEENSY)\.build\.core=\(.*\)$$/\1/p' "$(BOARDS_TXT)")
endif

#
# dependency handling.
#
# The only tricky dependency model here is "variables" since this
# makefile heavily relies on deferred variables.  With these the
# makefile can decide only in a recipe whether variable values
# have changed such that a rebuild is required.
#
# But the only (?) way to influence dependencies from a recipe is
# to relate the recipe to an included makefile, since GNU make
# restarts its dependency tracking only when an included makefile
# changes.  Therefore we do not only need file "vardeps" to track
# the variable values, but also that dummy makefile "vardeps.mk".
#

# duplicate the instructions from the dummy makefile recipe here.
# During a "make clean all", for example, both files get rebuilt
# through this target, and not through the dummy makefile target.
$(BUILD_DIR)/vardeps:
		@mkdir -p "$(BUILD_DIR)"
		@echo '$(VAR_DEPS)' | md5sum 1>$(BUILD_DIR)/vardeps
		@touch "$(BUILD_DIR)/vardeps.mk"

ifneq ($(NCITP),)
-include $(BUILD_DIR)/vardeps.mk
endif

.PHONY:		$(BUILD_DIR)/vardeps.mk
$(BUILD_DIR)/vardeps.mk:
		@set -e; set -o pipefail;					\
		vdsumact=$$( echo '$(VAR_DEPS)' | md5sum );			\
		vdsumnom=$$( cat $(BUILD_DIR)/vardeps 2>/dev/null || : );	\
		if test "$$vdsumact" != "$$vdsumnom"; then			\
		  mkdir -p "$(BUILD_DIR)";					\
		  echo "$$vdsumact" 1>$(BUILD_DIR)/vardeps;			\
		  touch "$(BUILD_DIR)/vardeps.mk";				\
		fi

# define dependency hook depending on selected dependency model
ifeq ($(DEP_MODEL), makefiles)
  DPH :=	$(filter-out $(BUILD_DIR)/%, $(MAKEFILE_LIST))
else ifeq ($(DEP_MODEL), variables)
  DPH :=	$(BUILD_DIR)/vardeps
else ifeq ($(DEP_MODEL), always)
  DPH :=	force
else
  DPH :=
endif

#
# derived paths and directories.
#

# local source files
INO_FILES :=	$(wildcard *.ino)
C_FILES :=	$(wildcard *.c)
CPP_FILES :=	$(wildcard *.cpp)
SRC_FILES :=	$(INO_FILES) $(C_FILES) $(CPP_FILES)

# Teensy core library directory
TCORE_DIR :=	$(TEENSY_BASE_DIR)/hardware/teensy/avr/cores/$(T_CORE)

# Teensy platform library directory
TPLTF_DIR :=	$(TEENSY_BASE_DIR)/hardware/teensy/avr/libraries

# list of include directories required to use the Teensyduino
# libraries referenced by the user
INC_PATH :=	$(strip							\
		  $(foreach						\
		    lib, $(call rslv_inc_libs, $(LIBRARIES)),		\
		    $(call lib_inc_dirs,				\
		           $(foreach					\
		             dir, $(USER_LIB_PATH) $(TPLTF_DIR),	\
		             $(dir)/$(lib)))))

# list of library directories required to use the Teensyduino
# libraries referenced by the user
LIB_PATH :=	$(strip							\
		  $(foreach						\
		    lib, $(call rslv_inc_libs, $(LIBRARIES)),		\
		    $(call lib_lib_dirs,				\
		           $(foreach					\
		             dir, $(USER_LIB_PATH) $(TPLTF_DIR),	\
		             $(dir)/$(lib)))))

# toolchain executable directory
TOOLS_DIR :=	$(TEENSY_BASE_DIR)/hardware/tools

#
# build variables and targets.
#

define compile-s
		@echo -e "[AS]\t$<"
		@mkdir -p "$(dir $@)"
		$(SILENT)$(CC) $(CPPFLAGS) $(ASFLAGS) -o $@ -c $<
endef

define compile-c
		@echo -e "[CC]\t$<"
		@mkdir -p "$(dir $@)"
		$(SILENT)$(CC) $(CPPFLAGS) $(CFLAGS) -o $@ -c $<
endef

define compile-cpp
		@echo -e "[CXX]\t$<"
		@mkdir -p "$(dir $@)"
		$(SILENT)$(CXX) $(CPPFLAGS) $(CXXFLAGS) -o $@ -c $<
endef

define compile-ino
		@echo -e "[CXX]\t$<"
		@mkdir -p "$(dir $@)"
		$(SILENT)$(CXX) $(CPPFLAGS) $(CXXFLAGS) -include Arduino.h -x c++ -o $@ -c $<
endef

# source files of the Teensy core library
TCS_FILES :=	$(wildcard $(TCORE_DIR)/*.S)
TCC_FILES :=	$(wildcard $(TCORE_DIR)/*.c)
TCCPP_FILES :=	$(wildcard $(TCORE_DIR)/*.cpp)

# object files of the Teensy core library
TCOBJ_FILES :=	$(patsubst $(TEENSY_BASE_DIR)/%.S,   $(BUILD_DIR)/teensy/%.o, $(TCS_FILES))	\
		$(patsubst $(TEENSY_BASE_DIR)/%.c,   $(BUILD_DIR)/teensy/%.o, $(TCC_FILES))	\
		$(patsubst $(TEENSY_BASE_DIR)/%.cpp, $(BUILD_DIR)/teensy/%.o, $(TCCPP_FILES))

# source files of all Teensyduino libraries referenced by the
# user
TLS_FILES :=	$(foreach tldir, $(LIB_PATH), $(wildcard $(tldir)/*.S))
TLC_FILES :=	$(foreach tldir, $(LIB_PATH), $(wildcard $(tldir)/*.c))
TLCPP_FILES :=	$(foreach tldir, $(LIB_PATH), $(wildcard $(tldir)/*.cpp))

# all non-core object files
OBJ_FILES :=	$(patsubst   %.ino,                    $(BUILD_DIR)/%.o, $(INO_FILES))		\
		$(patsubst   %.c,                      $(BUILD_DIR)/%.o, $(C_FILES))		\
		$(patsubst   %.cpp,                    $(BUILD_DIR)/%.o, $(CPP_FILES))		\
		$(patsubst   %.S,                      $(BUILD_DIR)/userlib/%.o,		\
		  $(patsubst $(TEENSY_BASE_DIR)/%.S,   $(BUILD_DIR)/teensy/%.o, $(TLS_FILES)))	\
		$(patsubst   %.c,                      $(BUILD_DIR)/userlib/%.o,		\
		  $(patsubst $(TEENSY_BASE_DIR)/%.c,   $(BUILD_DIR)/teensy/%.o, $(TLC_FILES)))	\
		$(patsubst   %.cpp,                    $(BUILD_DIR)/userlib/%.o,		\
		  $(patsubst $(TEENSY_BASE_DIR)/%.cpp, $(BUILD_DIR)/teensy/%.o, $(TLCPP_FILES)))

$(BUILD_DIR)/teensy/%.o: $(TEENSY_BASE_DIR)/%.S $(DPH)
		$(compile-s)

$(BUILD_DIR)/teensy/%.o: $(TEENSY_BASE_DIR)/%.c $(DPH)
		$(compile-c)

$(BUILD_DIR)/teensy/%.o: $(TEENSY_BASE_DIR)/%.cpp $(DPH)
		$(compile-cpp)

$(BUILD_DIR)/userlib/%.o: %.S $(DPH)
		$(compile-s)

$(BUILD_DIR)/userlib/%.o: %.c $(DPH)
		$(compile-c)

$(BUILD_DIR)/userlib/%.o: %.cpp $(DPH)
		$(compile-cpp)

$(BUILD_DIR)/%.o: %.ino $(DPH)
		$(compile-ino)

$(BUILD_DIR)/%.o: %.c $(DPH)
		$(compile-c)

$(BUILD_DIR)/%.o: %.cpp $(DPH)
		$(compile-cpp)

$(BUILD_DIR)/libcore.a: $(TCOBJ_FILES)
		@echo -e "[AR]\t$< ..."
		@for tcobjfile in $^; do			\
		  $(AR) rcs $@ "$$tcobjfile" || exit $$?;	\
		done

$(TARGET).elf:	$(OBJ_FILES) $(BUILD_DIR)/libcore.a
		@echo -e "[LD]\t$@"
		$(SILENT)$(CC) $(LDFLAGS) -o $@ $(OBJ_FILES) $(BUILD_DIR)/libcore.a $(T_LIBS) $(LDLIBS)

$(TARGET).hex:	$(TARGET).elf
		@echo -e "[OC]\t$@"
		@$(SIZE) $<
		@-$(SIZE) --format=sysv $< |				\
		awk 'BEGIN { ts = 0; ds = 0; }				\
		     /$(SIZE_RE_TEXT)/ { ts += $$2; }			\
		     /$(SIZE_RE_DATA)/ { ds += $$2; }			\
		     END {						\
		       mts = "$(call xtv,upload.maximum_size)";		\
		       if ( mts !~ /^[0-9]+$$/ ) mts = 0;		\
		       mds = "$(call xtv,upload.maximum_data_size)";	\
		       if ( mds !~ /^[0-9]+$$/ ) mds = 0;		\
		       if ( mts > 0 && mds > 0 )			\
		         printf( "Used program storage: %d (%.1f%%), "	\
		                 "dynamic memory: %d (%.1f%%)\n",	\
		                 ts, ts/mts*100.0,			\
		                 ds, ds/mds*100.0 );			\
		     }'
		$(SILENT)$(OBJCOPY) -O ihex -R .eeprom $< $@

.PHONY:		build
build:		$(TARGET).hex

.PHONY:		upload
upload:		$(TARGET).hex
ifeq ($(UPLOAD_TOOL), tycmd)
		$(EOTC)
		tycmd upload $<
else ifeq ($(UPLOAD_TOOL), tlcli)
		$(EOTC)
		teensy_loader_cli --mcu "$(call xtv,build.mcu)" -v -w $<
else ifeq ($(UPLOAD_TOOL), tlgui)
		$(EOTC)
		$(TOOLS_DIR)/teensy_post_compile -file=$(TARGET) -path=$(CURDIR) -tools=$(TOOLS_DIR)
		$(TOOLS_DIR)/teensy_reboot
else
		$(UPLOAD_TOOL)
endif

.PHONY:		monitor
monitor:
ifeq ($(MONITOR_TOOL), tycmd)
		$(EOTC)
		tycmd monitor
else ifeq ($(MONITOR_TOOL), teensy)
		$(EOTC)
		$(TOOLS_DIR)/teensy_serialmon "$$( $(TOOLS_DIR)/teensy_ports -L | awk '{ print $$1 }' )"
else
		$(MONITOR_TOOL)
endif

clean::
		rm -rf $(BUILD_DIR)
		rm -f $(TARGET).elf
		rm -f $(TARGET).hex

# include dependency files
ifneq ($(NCITP),)
-include $(TCOBJ_FILES:.o=.d) $(OBJ_FILES:.o=.d)
endif

#
# boards.txt functions and targets.
#

# functions to access board and menu variables from boards.txt
xtv =		$($(TEENSY).$(1))
xuv =		$($(TEENSY).menu.usb.$(T_USB).$(1))
xsv =		$($(TEENSY).menu.speed.$(T_SPEED).$(1))
xov =		$($(TEENSY).menu.opt.$(T_OPT).$(1))
xkv =		$($(TEENSY).menu.keys.$(T_KEYS).$(1))

ifneq ($(NCITP),)
-include $(PROJECT_BASE_DIR)/boards.mk
endif

# convert boards.txt into a makefile.  The awk script converts
# the first line of a new menu block
#
#   <teensy-id>.menu.<menu-id>.<menu-entry-id>=<value>
#
# into
#
#   <teensy-id>.menu.<menu-id>.default=<menu-entry-id>
#
# and the sed script does some minor make-compatibility-clean-up.
$(PROJECT_BASE_DIR)/boards.mk: $(BOARDS_TXT)
		@set -e; set -o pipefail;					\
		cat $< |							\
		awk 'BEGIN             { pmenu = "none"; menu = "none";  }	\
		     /\.menu\.usb\./   { pmenu = menu; 	 menu = "usb";   }	\
		     /\.menu\.speed\./ { pmenu = menu; 	 menu = "speed"; }	\
		     /\.menu\.opt\./   { pmenu = menu; 	 menu = "opt";   }	\
		     /\.menu\.keys\./  { pmenu = menu; 	 menu = "keys";  }	\
		     /^ *#/ { next; }						\
                     { print; }							\
                     /\.menu\./ && (menu != pmenu) {				\
                       sub( /=.*$$/, "", $$0 );   key = $$0;			\
                       sub( /^.*\./, "", $$0 );   def = $$0;			\
                       sub( /[^.]+$$/, "", key );				\
		       print( key "default=" def );				\
                     }' |							\
		sed -e 's/\.build\.command\.g++/.build.command.g_plus_plus/'	\
		    -e 's/{extra\.time\.local}/$$(TIME_LOCAL)/'			\
		    -e 's/{build\.core\.path}/$$(TCORE_DIR)/' 1>$@

.PHONY:		realclean
realclean::	clean
		rm -f $(PROJECT_BASE_DIR)/boards.mk

# targets to list possible values for the top-level and
# menu-level configuration variables
.PHONY:		list-teensy
list-teensy:
		@echo "Valid values for variables TEENSY:"
		@sed -n -e '/^ *#/d; s/^\([^.]*\)\.name=\(.*\)$$/\2:\n    TEENSY = \1/p' "$(BOARDS_TXT)"

.PHONY:		list-usb
list-usb:
		@echo "Valid values for variable T_USB on $(TEENSY):"
		@sed -n 's/^$(TEENSY)\.menu\.usb\.\([^.]*\)=\(.*\)$$/\2:\n    T_USB = \1/p' "$(BOARDS_TXT)"

.PHONY:		list-speed
list-speed:
		@echo "Valid values for variable T_SPEED on $(TEENSY):"
		@sed -n 's/^$(TEENSY)\.menu\.speed\.\([^.]*\)=\(.*\)$$/\2:\n    T_SPEED = \1/p' "$(BOARDS_TXT)"

.PHONY:		list-opt
list-opt:
		@echo "Valid values for variable T_OPT on $(TEENSY):"
		@sed -n 's/^$(TEENSY)\.menu\.opt\.\([^.]*\)=\(.*\)$$/\2:\n    T_OPT = \1/p' "$(BOARDS_TXT)"

.PHONY:		list-keys
list-keys:
		@echo "Valid values for variable T_KEYS on $(TEENSY):"
		@sed -n 's/^$(TEENSY)\.menu\.keys\.\([^.]*\)=\(.*\)$$/\2:\n    T_KEYS = \1/p' "$(BOARDS_TXT)"

#
# installation target.
#

.PHONY:		install
install:
#		perform some basic sanity checks
		@for varspec in "ARDUINO_BASE_DIR=$(ARDUINO_BASE_DIR)"			\
		                "TEENSY_BASE_DIR=$(TEENSY_BASE_DIR)"			\
		                "PROJECT_BASE_DIR=$(PROJECT_BASE_DIR)"; do		\
		  varname=$${varspec%%=*}; varval=$${varspec#*=};			\
		  if [[ "$$varspec" != *_BASE_DIR=/* ]]; then				\
		    echo "Invalid $$varname = \"$$varval\" (non-absolute)." 1>&2;	\
		    exit 1;								\
		  fi;									\
		  if [[ "$$varspec" = *[\ \	]* ]]; then				\
		    echo "Invalid $$varname = \"$$varval\" (contains ws)." 1>&2;	\
		    exit 1;								\
		  fi;									\
		  if test "$$varname" = "ARDUINO_BASE_DIR" &&				\
		     ! test -f "$$varval/hardware/teensy/avr/boards.txt"; then		\
		    echo "Invalid $$varname = \"$$varval\" (no boards.txt)." 1>&2;	\
		    exit 1;								\
		  fi;									\
		done
#		(re)create the Teensy base directory
		rm -rf "$(TEENSY_BASE_DIR)"
		mkdir -p "$(TEENSY_BASE_DIR)"
		@echo 'find ... | cpio -pdm "$(TEENSY_BASE_DIR)"';	\
		set -e; set -o pipefail;				\
		( cd "$(ARDUINO_BASE_DIR)" &&				\
		  find examples/Teensy					\
		       hardware/teensy/avr				\
		       hardware/tools/teensy* -depth -print0 |		\
                  cpio -pdm --null "$(TEENSY_BASE_DIR)" );		\
		if test "$(NO_TEENSY_GCC)" != "arm" &&			\
		   test "$(NO_TEENSY_GCC)" != "all"; then		\
		  ( cd "$(ARDUINO_BASE_DIR)" &&				\
		    find hardware/tools/arm -depth -print0 |		\
                    cpio -pdm --null "$(TEENSY_BASE_DIR)" );		\
		fi;							\
		if test "$(NO_TEENSY_GCC)" != "avr" &&			\
		   test "$(NO_TEENSY_GCC)" != "all"; then		\
		  ( cd "$(ARDUINO_BASE_DIR)" &&				\
		    find hardware/tools/avr -depth -print0 |		\
                    cpio -pdm --null "$(TEENSY_BASE_DIR)" );		\
		fi
#		prepare the project base directory
		mkdir -p "$(PROJECT_BASE_DIR)"
		rm -f "$(PROJECT_BASE_DIR)/boards.mk"
		@if test -f "$(PROJECT_BASE_DIR)/GNUmakefile"; then					\
		  echo 'cp -p "$(PROJECT_BASE_DIR)/GNUmakefile" "$(PROJECT_BASE_DIR)/GNUmakefile.bak"';	\
		  cp -p "$(PROJECT_BASE_DIR)/GNUmakefile" "$(PROJECT_BASE_DIR)/GNUmakefile.bak";	\
		fi
		@echo 'cat ... | sed ... 1>"$(PROJECT_BASE_DIR)/GNUmakefile"';		\
		set -e; set -o pipefail;						\
		upltldef="";								\
		mtrtldef="";								\
		if ( tycmd ) 1>/dev/null 2>&1; then					\
		  upltldef="tycmd";							\
		  mtrtldef="tycmd";							\
		elif ( teensy_loader_cli || : ) 2>&1 | grep -q 'www\.pjrc\.com'; then	\
		  upltldef="tlcli";							\
		  mtrtldef="teensy";							\
		else									\
		  upltldef="tlgui";							\
		  mtrtldef="teensy";							\
		fi;									\
		avers=$$( head -1 "$(ARDUINO_BASE_DIR)/revisions.txt" );		\
		if [[ "$$avers" =~ ^ARDUINO\ ([0-9]+)\.([0-9]+)\.([0-9]+) ]]; then	\
		  avers=$$( printf '%d%02d%02d' $${BASH_REMATCH[1]}			\
		                                $${BASH_REMATCH[2]}			\
		                                $${BASH_REMATCH[3]} );			\
		else									\
		  avers=10000;								\
		fi;									\
		tvers=$$( head -1 "$(ARDUINO_BASE_DIR)/lib/teensyduino.txt" );		\
		if [[ "$$tvers" =~ ^([0-9]+)\.([0-9]+) ]]; then				\
		  tvers=$$( printf '%d%02d' $${BASH_REMATCH[1]} $${BASH_REMATCH[2]} );	\
		else									\
		  tvers=100;								\
		fi;									\
		cat "$(strip $(MAKEFILE_LIST))" |					\
		sed -e 's@[#]## Teensy base directory$$@$(TEENSY_BASE_DIR)@'		\
		    -e 's@[#]## Project base directory$$@$(PROJECT_BASE_DIR)@'		\
		    -e 's@[#]## Teensy-gcc deselection.*$$@$(NO_TEENSY_GCC)@'		\
		    -e 's@[#]## Upload tool default.*$$@'"$$upltldef"'@'		\
		    -e 's@[#]## Monitor tool default.*$$@'"$$mtrtldef"'@'		\
		    -e 's@[#]## Ardunio version.*$$@'"$$avers"'@'			\
		    -e 's@[#]## Teensyduino version.*$$@'"$$tvers"'@'			\
		    1>"$(PROJECT_BASE_DIR)/GNUmakefile"
