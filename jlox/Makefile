##
# Project name - used to set the jar's file name
##
PROJ_NAME := atqa

##
# source directory
##
SRC_DIR := src/prod

##
# test source directory
##
TST_SRC_DIR := src/test

##
# overall output directory
##
OUT_DIR := out

##
# output directory for production files
##
OUT_DIR_PROD := $(OUT_DIR)/prod

##
# output directory for test files
##
OUT_DIR_TEST := $(OUT_DIR)/test

##
# the library of stable jars used by our program
##
LIB := lib

##
# the utilties
##
UTILS := utils

##
# sources
##
SRCS := $(shell find ${SRC_DIR} -type f -name '*.java' -print)

##
# test sources
##
TST_SRCS := $(shell find ${TST_SRC_DIR} -type f -name '*.java' -print)

##
# build classpath options - the classpaths needed to build
##
BUILD_CP := "$(SRC_DIR)/:$(LIB)/*"

##
# build classpath for the tests
##
TEST_BUILD_CP := "$(SRC_DIR)/:$(LIB)/*:$(TST_SRC_DIR)/:$(OUT_DIR_PROD)/:$(OUT_DIR_TEST)/"

##
# run classpath options - the classpaths needed to run the program
##
RUN_CP := "$(OUT_DIR_PROD):$(LIB)/*"

##
# run classpath for tests
##
TST_RUN_CP := "$(OUT_DIR_PROD):$(LIB)/*:$(OUT_DIR_TEST)"

##
# classes
##
CLS := $(SRCS:$(SRC_DIR)/%.java=$(OUT_DIR_PROD)/%.class)

##
# test classes
##
TST_CLS := $(TST_SRCS:$(TST_SRC_DIR)/%.java=$(OUT_DIR_TEST)/%.class)

# If Java home is defined (either from command-line
# argument or environment variable), add /bin/ to it
# to access the proper location of the java binaries
#
# otherwise, it will just remain an empty string
ifneq ($(JAVA_HOME),)
  JAVA_HOME := $(JAVA_HOME)/bin/
endif

# the name of our Java compiler
JC = $(JAVA_HOME)javac

# the name of the java runner
JAVA = $(JAVA_HOME)java

# the directory where we store the code coverage report
COV_DIR = out/coveragereport

##
# suffixes
##
.SUFFIXES: .java

##
# targets that do not produce output files
##
.PHONY: all clean run test testcov rundebug testdebug jar classes testclasses

##
# default target(s)
##
all: classes

LIST:=
TEST_LIST:=

# note that putting an @ in front of a command in a makefile
# will cause that command not to echo out when running Make.

classes: $(CLS)
	    @if [ ! -z "$(LIST)" ] ; then \
	        $(JC) -Xlint:unchecked -Werror -g -d $(OUT_DIR_PROD)/ -cp $(BUILD_CP) $(LIST) ; \
	    fi

testclasses: $(TST_CLS)
	    @if [ ! -z "$(TEST_LIST)" ] ; then \
	        $(JC) -Xlint:unchecked -Werror -g -d $(OUT_DIR_TEST)/ -cp $(TEST_BUILD_CP) $(TEST_LIST) ; \
	    fi

# here is the target for the application code
$(CLS): $(OUT_DIR_PROD)/%.class: $(SRC_DIR)/%.java
	   $(eval LIST+=$$<)

# here is the target for the test code
$(TST_CLS): $(OUT_DIR_TEST)/%.class: $(TST_SRC_DIR)/%.java
	    $(eval TEST_LIST+=$$<)

#: clean up any output files
clean:
	    rm -fr $(OUT_DIR)

#: jar up the application (See Java's jar command)
jar: all
	    cd $(OUT_DIR_PROD) && jar --create --file $(PROJ_NAME).jar -e primary.Main *

#: run the application
run: all
	    $(JAVA) -cp $(RUN_CP) com.craftinginterpreters.lox.Lox

#: run the application and open a port for debugging.
rundebug: all
	    $(JAVA) -agentlib:jdwp=transport=dt_socket,server=y,address=8000,suspend=y -cp $(RUN_CP) com.craftinginterpreters.lox.Lox

#: run the tests
test: all testclasses
	    $(JAVA) -cp $(TST_RUN_CP) primary.Tests

#: run the tests and open a port for debugging.
testdebug: all testclasses
	    $(JAVA) -agentlib:jdwp=transport=dt_socket,server=y,address=8000,suspend=y -cp $(TST_RUN_CP) primary.Tests

#: If you want to obtain code coverage from running the tests
testcov: all testclasses
	    $(JAVA) -javaagent:$(UTILS)/jacocoagent.jar=destfile=$(COV_DIR)/jacoco.exec -cp $(TST_RUN_CP) primary.Tests
	    $(JAVA) -jar $(UTILS)/jacococli.jar report $(COV_DIR)/jacoco.exec --html ./$(COV_DIR) --classfiles $(OUT_DIR_PROD) --sourcefiles $(SRC_DIR)

# a handy debugging tool.  If you want to see the value of any
# variable in this file, run something like this from the
# command line:
#
#     make print-CLS
#
# and you'll get something like: CLS = out/logging/ILogger.class out/logging/Logger.class out/primary/Main.class out/utils/ActionQueue.class
print-%:
	    @echo $* = $($*)

# This is a handy helper.  This prints a menu of items
# from this file - just put hash+colon over a target and type
# the description of that target.  Run this from the command
# line with "make help"
help:
	    @grep -B1 -E "^[a-zA-Z0-9_-]+\:([^\=]|$$)" Makefile \
     | grep -v -- -- \
     | sed 'N;s/\n/###/' \
     | sed -n 's/^#: \(.*\)###\(.*\):.*/\2###\1/p' \
     | column -t  -s '###'
