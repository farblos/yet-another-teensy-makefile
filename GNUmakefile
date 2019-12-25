#
# GNUmakefile - Yet Another Teensy Makefile.
#

# force the use of Bash on Debian systems
SHELL :=	/bin/bash

##################################################################
# installation-time variables.  Instantiated by target "install".
##################################################################

TEENSY_BASE_DIR :=	### Teensy base directory

PROJECT_BASE_DIR :=	### Project base directory

NO_TEENSY_TOOLS :=	### No-Teensy-tools flag ("0" or "1")

ARDUINO_VERSION :=	### Ardunio version (sans periods)

TEENSYDUINO_VERSION :=	### Teensyduino version (sans periods)

##################################################################
# non-customizable variables and targets
##################################################################

.PHONY:		install
install:
#		perform some basic sanity checks
		@for dir in "$(ARDUINO_BASE_DIR)"				\
		            "$(TEENSY_BASE_DIR)"				\
		            "$(PROJECT_BASE_DIR)"; do				\
		  [[ "$$dir" = /* ]] ||						\
		  { echo "Invalid non-absolute directory \"$$dir\"" 1>&2;	\
		    exit 1; };							\
		done
#		(re)create the Teensy base directory
		rm -rf "$(TEENSY_BASE_DIR)"
		mkdir -p "$(TEENSY_BASE_DIR)"
		@echo 'find ... | cpio -pdm "$(TEENSY_BASE_DIR)"';		\
		( cd "$(ARDUINO_BASE_DIR)" &&					\
		  find examples/Teensy						\
		       hardware/teensy/avr -depth -print0 |			\
                  cpio -pdm --null "$(TEENSY_BASE_DIR)" );			\
		if test "$(NO_TEENSY_TOOLS)" != 1; then				\
		  ( cd "$(ARDUINO_BASE_DIR)" &&					\
		    find hardware/tools/arm					\
		         hardware/tools/teensy* -depth -print0 |		\
                    cpio -pdm --null "$(TEENSY_BASE_DIR)" );			\
		fi
#		prepare the project base directory
		mkdir -p "$(PROJECT_BASE_DIR)"
		@echo 'cat ... | sed ... 1>"$(PROJECT_BASE_DIR)/GNUmakefile"';		\
		if test "$(NO_TEENSY_TOOLS)" != 1; then					\
		  nttp=0;								\
		else									\
		  nttp=1;								\
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
		sed -e 's@### Teensy base directory$$@$(TEENSY_BASE_DIR)@'		\
		    -e 's@### Project base directory$$@$(PROJECT_BASE_DIR)@'		\
		    -e 's@### No-Teensy-tools flag.*$$@'"$$nttp"'@'			\
		    -e 's@### Ardunio version.*$$@'"$$avers"'@'				\
		    -e 's@### Teensyduino version.*$$@'"$$tvers"'@'			\
		    1>"$(PROJECT_BASE_DIR)/GNUmakefile"
