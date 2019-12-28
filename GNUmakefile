#
# GNUmakefile - Yet Another Teensy Makefile.
#
# Copyright (c) 2019 Jens Schmidt.
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
# for documentation on variable and targets.
#

# force the use of Bash on Debian systems
SHELL :=	/bin/bash

#
# installation-time variables.
#
# These are instantiated by target "install".
#

TEENSY_BASE_DIR :=	### Teensy base directory

PROJECT_BASE_DIR :=	### Project base directory

NO_TEENSY_TOOLS :=	### No-Teensy-tools flag (non-empty for true)

ARDUINO_VERSION :=	### Ardunio version (sans periods)

TEENSYDUINO_VERSION :=	### Teensyduino version (sans periods)

#
# top-level configuration variables.
#

# configuration variable TEENSY is so top-level that it is not
# even mentioned here.  See README.md for more information.

ifeq ($(origin LIBRARIES), undefined)
LIBRARIES :=
endif

ifeq ($(origin TARGET), undefined)
TARGET :=	$(notdir $(CURDIR))
endif

ifeq ($(origin BUILD_DIR), undefined)
BUILD_DIR :=	build
endif

ifeq ($(origin SILENT), undefined)
SILENT :=	@
endif

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

# controlled by TEENSY
T_COMMON ?=	$(call xtv,build.flags.common)
T_DEP ?=	$(call xtv,build.flags.dep)
T_GOPT ?=	$(call xtv,build.flags.optimize)
T_CPU ?=	$(call xtv,build.flags.cpu)
T_DEFS ?=	$(call xtv,build.flags.defs)
T_C ?=		$(call xtv,build.flags.c)
T_CPP ?=	$(call xtv,build.flags.cpp)
T_S ?=		$(call xtv,build.flags.S)
T_LD ?=		$(call xtv,build.flags.ld)
T_LIBS ?=	$(call xtv,build.flags.libs)

# controlled by T_USB
T_USBTYPE ?=	$(call xuv,build.usbtype)

# controlled by T_SPEED
T_FCPU ?=	$(call xsv,build.fcpu)

# controlled by T_OPT
T_OPTIMIZE ?=	$(call xov,build.flags.optimize)
T_LDSPECS ?=	$(call xov,build.flags.ldspecs)

# controlled by T_KEYS
T_KEYLAYOUT ?=	$(call xkv,build.keylayout)

#
# compiler flags and options.
#

CPPFLAGS ?=	-D$(T_USBTYPE) -DF_CPU=$(T_FCPU) -DLAYOUT_$(T_KEYLAYOUT) $(T_DEFS)	\
		-DARDUINO=$(ARDUINO_VERSION) $(DEFINES)					\
		-I. -I$(TCORE_DIR) $(addprefix -I, $(TLIB_PATH)) $(INCLUDES)

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

# compiler executable directory.  Ends in slash if non-empty.
TCEXE_DIR =	$(if $(NO_TEENSY_TOOLS),,$(TOOLS_DIR)/$(call xtv,build.toolchain))

ifeq ($(filter-out undefined default, $(origin CC)),)
CC =		$(TCEXE_DIR)$(call xtv,build.command.gcc)
endif

ifeq ($(filter-out undefined default, $(origin CXX)),)
CXX =		$(TCEXE_DIR)$(call xtv,build.command.g_plus_plus)
endif

ifeq ($(filter-out undefined default, $(origin AR)),)
AR =		$(TCEXE_DIR)$(call xtv,build.command.ar)
endif

OBJCOPY ?=	$(TCEXE_DIR)$(call xtv,build.command.objcopy)

LD ?=		$(TCEXE_DIR)$(call xtv,build.command.linker)

SIZE ?=		$(TCEXE_DIR)$(call xtv,build.command.size)

#
# verify configuration variable TEENSY and determine T_CORE.
#

BOARDS_TXT :=	$(TEENSY_BASE_DIR)/hardware/teensy/avr/boards.txt

ifeq ($(filter-out install list-teensy,$(MAKECMDGOALS)),)
  # no-op
else ifeq ($(origin TEENSY), undefined)
  $(error Undefined configuration variable TEENSY)
else ifeq ($(shell test -f '$(BOARDS_TXT)' && echo found),)
  $(error Inaccessible boards.txt file "$(BOARDS_TXT)")
else ifeq ($(shell grep '^$(TEENSY)\.name=' '$(BOARDS_TXT)' 2>/dev/null),)
  $(error Invalid configuration variable TEENSY ("$(TEENSY)" not found in boards.txt))
else
  # use function "shell" only this one time and the user-defined
  # boards.txt accessor functions for all other instances
  T_CORE :=	$(shell sed -n 's/^$(TEENSY)\.build\.core=\(.*\)$$/\1/p' '$(BOARDS_TXT)')
endif

#
# derived paths and directories.
#

# Teensy core library
TCORE_DIR :=	$(TEENSY_BASE_DIR)/hardware/teensy/avr/cores/$(T_CORE)

# Teensy additional libraries
TLIB_DIR :=	$(TEENSY_BASE_DIR)/hardware/teensy/avr/libraries

# user-required libraries with directory prepended
TLIB_PATH :=	$(addprefix $(TLIB_DIR)/, $(LIBRARIES))

# toolchain executable directory
TOOLS_DIR :=	$(TEENSY_BASE_DIR)/hardware/tools

#
# main variables and targets.
#

# source files of the Teensy core library
TCS_FILES :=	$(wildcard $(TCORE_DIR)/*.S)
TCC_FILES :=	$(wildcard $(TCORE_DIR)/*.c)
TCCPP_FILES :=	$(wildcard $(TCORE_DIR)/*.cpp)

# object files of the Teensy core library
TCOBJ_FILES :=	$(patsubst $(TEENSY_BASE_DIR)/%.S, $(BUILD_DIR)/%.o, $(TCS_FILES))		\
		$(patsubst $(TEENSY_BASE_DIR)/%.c, $(BUILD_DIR)/%.o, $(TCC_FILES))		\
		$(patsubst $(TEENSY_BASE_DIR)/%.cpp, $(BUILD_DIR)/%.o, $(TCCPP_FILES))

# source files of the Teensy additional libraries
TLC_FILES :=	$(foreach tldir, $(TLIB_PATH), $(wildcard $(tldir)/*.c))
TLCPP_FILES :=	$(foreach tldir, $(TLIB_PATH), $(wildcard $(tldir)/*.cpp))

# local source files
C_FILES :=	$(wildcard *.c)
CPP_FILES :=	$(wildcard *.cpp)
INO_FILES :=	$(wildcard *.ino)

OBJ_FILES :=	$(patsubst $(TEENSY_BASE_DIR)/%.c, $(BUILD_DIR)/%.o, $(TLC_FILES))		\
		$(patsubst $(TEENSY_BASE_DIR)/%.cpp, $(BUILD_DIR)/%.o, $(TLCPP_FILES))		\
		$(patsubst %.c, $(BUILD_DIR)/%.o, $(C_FILES))					\
		$(patsubst %.cpp, $(BUILD_DIR)/%.o, $(CPP_FILES))				\
		$(patsubst %.ino, $(BUILD_DIR)/%.o, $(INO_FILES))

.PHONY:		all
all:		$(TARGET).hex

$(BUILD_DIR)/%.o: $(TEENSY_BASE_DIR)/%.S
		@echo -e "[AS]\t$<"
		@mkdir -p "$(dir $@)"
		$(SILENT)$(CC) $(CPPFLAGS) $(ASFLAGS) -o $@ -c $<

$(BUILD_DIR)/%.o: $(TEENSY_BASE_DIR)/%.c
		@echo -e "[CC]\t$<"
		@mkdir -p "$(dir $@)"
		$(SILENT)$(CC) $(CPPFLAGS) $(CFLAGS) -o $@ -c $<

$(BUILD_DIR)/%.o: $(TEENSY_BASE_DIR)/%.cpp
		@echo -e "[CXX]\t$<"
		@mkdir -p "$(dir $@)"
		$(SILENT)$(CXX) $(CPPFLAGS) $(CXXFLAGS) -o $@ -c $<

$(BUILD_DIR)/%.o: %.c
		@echo -e "[CC]\t$<"
		@mkdir -p "$(dir $@)"
		$(SILENT)$(CC) $(CPPFLAGS) $(CFLAGS) -o $@ -c $<

$(BUILD_DIR)/%.o: %.cpp
		@echo -e "[CXX]\t$<"
		@mkdir -p "$(dir $@)"
		$(SILENT)$(CXX) $(CPPFLAGS) $(CXXFLAGS) -o $@ -c $<

$(BUILD_DIR)/%.o: %.ino
		@echo -e "[INO]\t$<"
		@mkdir -p "$(dir $@)"
		$(SILENT)$(CXX) $(CPPFLAGS) $(CXXFLAGS) -include Arduino.h -x c++ -o $@ -c $<

$(BUILD_DIR)/core.a: $(TCOBJ_FILES)
		@echo -e "[AR]\t$< ..."
		@for tcobj in $^; do		\
		  $(AR) rcs $@ "$$tcobj";	\
		done

$(TARGET).elf:	$(BUILD_DIR)/core.a $(OBJ_FILES)
		@echo -e "[LD]\t$@"
		$(SILENT)$(CC) $(LDFLAGS) -o $@ $(OBJ_FILES) $(BUILD_DIR)/core.a $(T_LIBS) $(LDLIBS)

$(TARGET).hex:	$(TARGET).elf
		@echo -e "[HEX]\t$@"
		@$(SIZE) $<
		@$(SIZE) $< |						\
		awk '/^ *[0-9]+[ \t]+[0-9]+[ \t]+[0-9]+/		\
		       { ts = $$1; ds = $$2; bs= $$3; }			\
		     END {						\
		       mts = "$(call xtv,upload.maximum_size)";		\
		       if ( mts !~ /^[0-9]+$$/ ) mts = 0;		\
		       mds = "$(call xtv,upload.maximum_data_size)";	\
		       if ( mds !~ /^[0-9]+$$/ ) mds = 0;		\
		       if ( mts > 0 && mds > 0 )			\
		         printf( "Used program storage: %.2f%%, "	\
				 "dynamic memory: %.2f%%\n",		\
		                 (ts+ds)/mts*100.0,			\
		                 (ds+bs)/mds*100.0 );			\
		     }'
		$(SILENT)$(OBJCOPY) -O ihex -R .eeprom $< $@

.PHONY:		upload
upload:		$(TARGET).hex
ifeq ($(NO_TEENSY_TOOLS),)
		$(TOOLS_DIR)/teensy_post_compile -file=$(TARGET) -path=$(CURDIR) -tools=$(TOOLS_DIR)
		$(TOOLS_DIR)/teensy_reboot
else
		$(TOOLS_DIR)/teensy_loader_cli --mcu "$(call xtv,build.mcu)" -w -v $<
endif

clean::
		rm -rf $(BUILD_DIR)
		rm -f $(TARGET).elf
		rm -f $(TARGET).hex

# include dependency files
ifneq ($(MAKECMDGOALS),install)
-include $(OBJ_FILES:.o=.d)
endif

#
# auxilliary variables and targets.
#

# seconds since the epoch, required to set RTC on upload
TIME_LOCAL =	$(shell date +'%s')

# dump the value of a variable
.PHONY:		echo.%
echo.%:
		@echo $($*)

#
# boards.txt functions and targets.
#

# functions to access board and menu variables from boards.txt
xtv =		$($(TEENSY).$(1))
xuv =		$($(TEENSY).menu.usb.$(T_USB).$(1))
xsv =		$($(TEENSY).menu.speed.$(T_SPEED).$(1))
xov =		$($(TEENSY).menu.opt.$(T_OPT).$(1))
xkv =		$($(TEENSY).menu.keys.$(T_KEYS).$(1))

ifneq ($(MAKECMDGOALS),install)
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
		@cat $< |							\
		awk 'BEGIN             { pmenu = "none"; menu = "none";  }	\
		     /\.menu\.usb\./   { pmenu = menu; 	 menu = "usb";   }	\
		     /\.menu\.speed\./ { pmenu = menu; 	 menu = "speed"; }	\
		     /\.menu\.opt\./   { pmenu = menu; 	 menu = "opt";   }	\
		     /\.menu\.keys\./  { pmenu = menu; 	 menu = "keys";  }	\
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
		@sed -n -e 's/^\([^.]*\)\.name=\(.*\)$$/\2:\n  TEENSY =\t\t\1/p' "$(BOARDS_TXT)"

.PHONY:		list-usb
list-usb:
		@echo "Valid values for variable T_USB on $(TEENSY):"
		@sed -n 's/^$(TEENSY)\.menu\.usb\.\([^.]*\)=\(.*\)$$/\2:\n  T_USB =\t\t\1/p' "$(BOARDS_TXT)"

.PHONY:		list-speed
list-speed:
		@echo "Valid values for variable T_SPEED on $(TEENSY):"
		@sed -n 's/^$(TEENSY)\.menu\.speed\.\([^.]*\)=\(.*\)$$/\2:\n  T_SPEED =\t\t\1/p' "$(BOARDS_TXT)"

.PHONY:		list-opt
list-opt:
		@echo "Valid values for variable T_OPT on $(TEENSY):"
		@sed -n 's/^$(TEENSY)\.menu\.opt\.\([^.]*\)=\(.*\)$$/\2:\n  T_OPT =\t\t\1/p' "$(BOARDS_TXT)"

.PHONY:		list-keys
list-keys:
		@echo "Valid values for variable T_KEYS on $(TEENSY):"
		@sed -n 's/^$(TEENSY)\.menu\.keys\.\([^.]*\)=\(.*\)$$/\2:\n  T_KEYS =\t\t\1/p' "$(BOARDS_TXT)"

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
		  if test "$$varname" = "ARDUINO_BASE_DIR" &&				\
		     ! test -f "$$varval/hardware/teensy/avr/boards.txt"; then		\
		    echo "Invalid $$varname = \"$$varval\" (no boards.txt)." 1>&2;	\
		    exit 1;								\
		  fi;									\
		done
#		(re)create the Teensy base directory
		rm -rf "$(TEENSY_BASE_DIR)"
		mkdir -p "$(TEENSY_BASE_DIR)"
		@echo 'find ... | cpio -pdm "$(TEENSY_BASE_DIR)"';		\
		( cd "$(ARDUINO_BASE_DIR)" &&					\
		  find examples/Teensy						\
		       hardware/teensy/avr -depth -print0 |			\
                  cpio -pdm --null "$(TEENSY_BASE_DIR)" );			\
		if test -z "$(NO_TEENSY_TOOLS)"; then				\
		  ( cd "$(ARDUINO_BASE_DIR)" &&					\
		    find hardware/tools/arm					\
		         hardware/tools/avr					\
		         hardware/tools/teensy* -depth -print0 |		\
                    cpio -pdm --null "$(TEENSY_BASE_DIR)" );			\
		fi
#		prepare the project base directory
		mkdir -p "$(PROJECT_BASE_DIR)"
		rm -f "$(PROJECT_BASE_DIR)/boards.mk"
		@if test -f "$(PROJECT_BASE_DIR)/GNUmakefile"; then					\
		  echo 'cp -p "$(PROJECT_BASE_DIR)/GNUmakefile" "$(PROJECT_BASE_DIR)/GNUmakefile.bak"';	\
		  cp -p "$(PROJECT_BASE_DIR)/GNUmakefile" "$(PROJECT_BASE_DIR)/GNUmakefile.bak";	\
		fi
		@echo 'cat ... | sed ... 1>"$(PROJECT_BASE_DIR)/GNUmakefile"';		\
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
		    -e 's@[#]## No-Teensy-tools flag.*$$@$(NO_TEENSY_TOOLS)@'		\
		    -e 's@[#]## Ardunio version.*$$@'"$$avers"'@'			\
		    -e 's@[#]## Teensyduino version.*$$@'"$$tvers"'@'			\
		    1>"$(PROJECT_BASE_DIR)/GNUmakefile"
