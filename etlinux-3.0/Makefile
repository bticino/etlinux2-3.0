#
# Makefile used to build a runtime tree for etlinux. It uses gmake features.
#
# Export all variables.
export
### Uncomment (or pass "X=-x" on the command line) to get verbose compilation
# X = -x



############### Configuration section. Later, a warning section and then code

# use SOURCEDIR to allow building elsewhere; relative pathnames allowed
ifndef SOURCEDIR
   SOURCEDIR = $(shell /bin/pwd)
else
   SOURCEDIR := $(shell $(SOURCEDIR)/tools/abspath $(SOURCEDIR))
endif

BUILDDIR = $(shell /bin/pwd)

TOOLS = $(SOURCEDIR)/tools

# Read config file. This is preferred, otherwise warnings are spit later
-include Etlinux.cfg

# default config is Etlinux
CONFIG ?= Etlinux

# Read config file either in this directory or from sourcedir
# Also $(CONFIGFILE).mk is included later, if it exists
ifeq ($(wildcard ./$(CONFIG)),./$(CONFIG))
   CONFIGFILE = $(BUILDDIR)/$(CONFIG)
else
   CONFIGFILE = $(SOURCEDIR)/Configs/$(CONFIG)
endif

# PLATFORM is included later
PLATFORM ?= default

# TARGETDIR is where the output goes
TARGETDIR ?= $(BUILDDIR)/Trees/$(CONFIG)

# HTML is a stupid html output file
HTML := $(BUILDDIR)/Trees/$(CONFIG).html

################ Since Etlinux.cfg is preferred, warn if it has not been used
HADWARNED :=
ifeq ("$(origin CONFIG)", "command line")
   SAY := $(shell echo "Makefile: warning:" \
	"CONFIG is defined on command line" >& 2)
   HADWARNED += sth
endif
ifeq ("$(origin CONFIG)", "environment")
   SAY := $(shell echo "Makefile: warning:" \
	"CONFIG is defined in environment" >& 2)
   HADWARNED += sth
endif
ifeq ("$(origin PLATFORM)", "command line")
   SAY := $(shell echo "Makefile: warning:" \
	"PLATFORM is defined on command line" >& 2)
   HADWARNED += sth
endif
ifeq ("$(origin PLATFORM)", "environment")
   SAY := $(shell echo "Makefile: warning:" \
	"PLATFORM is defined in environment" >& 2)
   HADWARNED += sth
endif
ifeq ("$(origin CUSTOM)", "command line")
   SAY := $(shell echo "Makefile: warning:" \
	"CUSTOM is defined on command line" >& 2)
   HADWARNED += sth
endif
ifeq ("$(origin CUSTOM)", "environment")
   SAY := $(shell echo "Makefile: warning:" \
	"CUSTOM is defined in environment" >& 2)
   HADWARNED += sth
endif

ifneq ($(HADWARNED),)
   SAY := $(shell echo "Makefile: warning:" \
	"consider writing Etlinux.cfg instead" >& 2)
endif

############### Set up is over, now run

### Include the platform file
include $(SOURCEDIR)/Platforms/$(PLATFORM)
#since it might have changed $(PLATFORM), include it again
include $(SOURCEDIR)/Platforms/$(PLATFORM)

#### And include the configuration stuff too, if it is there
-include $(CONFIGFILE).mk

#### Now, use CROSS_COMPILE if defined (some of these are not used).
# note that AS, LD, CC, may be set by the platform file (to add
# compulsory flags: see arm-cross-snapgear-be for an example)
AS     ?= $(CROSS_COMPILE)as
LD     ?= $(CROSS_COMPILE)ld
CC     ?= $(CROSS_COMPILE)gcc
AR      = $(CROSS_COMPILE)ar
NM      = $(CROSS_COMPILE)nm
STRIP   = $(CROSS_COMPILE)strip
OBJCOPY = $(CROSS_COMPILE)objcopy
OBJDUMP = $(CROSS_COMPILE)objdump


#### extract the list of packages from the build file or from globbing

PACKAGES ?= $(shell PLATFORMPKG="$(PLATFORMPKG)" \
			sh $X $(TOOLS)/extractpackages $(CONFIGFILE))

#### Define the base package
BASE = _base-$(CONFIG)
ifneq ($(SOURCEDIR)/$(BASE).tar.gz, $(wildcard $(SOURCEDIR)/$(BASE).tar.gz))
  BASE = _base
endif

COMPILATIONMARKERS := $(foreach m,$(shell $(TOOLS)/pkg-to-marker $(BASE) $(PACKAGES)), $m)

#CHECK:= $(shell echo Packages: $(PACKAGES) > /dev/tty)

#### Define the html head, mid and tail
HTMLHEAD = Configs/Html-$(CONFIG).head
ifneq ($(SOURCEDIR)/$(HTMLHEAD), $(wildcard $(SOURCEDIR)/$(HTMLHEAD)))
	HTMLHEAD = $(SOURCEDIR)/Html.head
endif

HTMLMID = Configs/Html-$(CONFIG).mid
ifneq ($(SOURCEDIR)/$(HTMLMID), $(wildcard $(SOURCEDIR)/$(HTMLMID)))
	HTMLMID = $(SOURCEDIR)/Html.mid
endif

HTMLTAIL = Configs/Html-$(CONFIG).tail
ifneq ($(SOURCEDIR)/$(HTMLTAIL), $(SOURCEDIR)/$(wildcard $(HTMLTAIL)))
	HTMLTAIL = $(SOURCEDIR)/Html.tail
endif

.PHONY: all build install terse clean nosource cleanest \
	do.build do.install do.terse do.clean do.nosource do.cleanest \
	compilationmarkers checkbuild checkplatform

#### Rules to perform generic operations based on file names

%: $(SOURCEDIR)/%.tar.gz
	@if ! [ -d $* ] ; then \
		$(TOOLS)/untarandpatch $^; \
	fi

tmp/%.compile: $(SOURCEDIR)/%.build
	@if [ '!' -d tmp ]; then mkdir tmp; fi
	$(TOOLS)/extract-build-script $^ > $@

tmp/%.install: $(SOURCEDIR)/%.build
	@if [ '!' -d tmp ]; then mkdir tmp; fi
	$(TOOLS)/extract-install-script $^ > $@

%.bin: tmp/%.built tmp/%.install
	@echo ------ Installing $* to $*.bin
	@if [ ! -r tmp/$*.install ]; then $(MAKE) tmp/$*.install; fi
	@mkdir -p $*.bin
	@export TARGET="$(shell pwd)/$*.bin" && cd `$(TOOLS)/getsource $*` \
		&& sh $X $(BUILDDIR)/tmp/$*.install


#### Actual running instructions

# $(HTML) must be created before installing, as a package may want to import
# it into the /html installation tree.

all: debugprint checkbuild $(HTML) build install

debugprint:
ifdef	DEBUG_MAKEFILE
	@echo SOURCEDIR = $(SOURCEDIR)
	@echo BUILDDIR = $(BUILDDIR)
	@echo TARGETDIR = $(TARGETDIR)
	@echo HTML = $(HTML)
	@echo PLATFORM = $(PLATFORM)
	@echo CONFIG = $(CONFIG)
	@echo CONFIGFILE = $(CONFIGFILE)
	@echo CUSTOM = $(CUSTOM)
	@echo COMPILATIONMARKERS = $(COMPILATIONMARKERS)
endif

#Hmm... looks like I can't use implicit intermediate "%" files.
tmp/%.built: tmp/%.compile 
	if [ ! -d $*.bin ] ; then \
	  if [ ! -d `$(TOOLS)/getsource $*` ]; then \
		$(MAKE) `$(TOOLS)/getsource $*`; fi; \
	  if [ tmp/$*.compile -ot $(SOURCEDIR)/$*.build ]; then \
		$(MAKE) tmp/$*.compile; fi; \
	  export TARGET="$(TARGETDIR)"; \
	  (cd `$(TOOLS)/getsource $*` && sh $X $(BUILDDIR)/tmp/$*.compile) || exit 1; \
	fi
	@touch $@


nocheck: $(HTML) build.nocheck install.nocheck


# This sanity check is used by all rules below.
checkbuild::
	@if [ ! -f $(CONFIGFILE) ]; then \
		echo "Error: $(CONFIGFILE) does not exist"; exit 1; \
	fi
	@if [ "x$(CROSS_COMPILE)" != "x" ] ; then \
	  if echo $(CC) | grep -q '^$(CROSS_COMPILE)'; then \
	    true; else \
	    echo "Error: \$$CC doesn't match \$$CROSS_COMPILE"; exit 1; \
	  fi; \
        fi
	@bash -c '[ "$$EUID" -eq 0 ] || echo "*** You must run this as root."'
	@bash -c '[ "$$EUID" -eq 0 ] || exit 1'
	@for n in $(BASE) $(PACKAGES); do \
		basename=`echo $$n | sed 's/-.*$$//'`; \
		if ! [ -r $(SOURCEDIR)/$$n.build \
		    -o -r $(SOURCEDIR)//$$basename.build ]; then \
			echo "Error: no build file for $$n"; exit 1; \
		 fi; \
	done

# This is used by build below.
checkplatform::
	@if [ -x $(TOOLS)/checkplatform-$(PLATFORM) ]; then \
		echo "==== Mandatory test for platform $(PLATFORM)"; \
			$(TOOLS)/checkplatform-$(PLATFORM) || exit 1; \
		echo "==== Test OK"; \
	fi

# This non-phony rule is used to recreate a makefile from build files
# Other rules must all re-invoke $(MAKE) in order to have
# Makefile.auto included when doing real work
Makefile.auto: $(wildcard *.build)
	CONFIG=$(CONFIG) PACKAGES="$(PACKAGES)" $(TOOLS)/makemakefile > $@ \
		|| (rm -f $@; exit 1)


# Build: extract patch and compile unless a .bin directory is there
# It depends on $(HTML) because sample-html includes that very file
build: Makefile.auto $(HTML)
	@echo ========= Beginning build process
	$(MAKE) do.build

do.build: checkbuild checkplatform Makefile.auto compilationmarkers

# Same, without checks
build.nocheck: Makefile.auto $(HTML)
	@echo ========= "Beginning build process (without sanity checks)"
	$(MAKE) Makefile.auto compilationmarkers

compilationmarkers: $(COMPILATIONMARKERS)


# Install: either copy the .bin directory or run the install script
install: Makefile.auto
	$(MAKE) do.install

install.nocheck: Makefile.auto
	$(MAKE) do.install.nocheck

do.install: checkbuild do.install.nocheck

do.install.nocheck:
	@if [ '!' -d tmp ]; then mkdir tmp; fi
	@echo ========= Installing into $(TARGETDIR)
	@for n in $(BASE) $(PACKAGES); do \
	    if [ -d $$n.bin ] ; then \
		echo ------ Copying $$n.bin; \
		pushd $$n.bin && \
			find . -depth -print | \
			 $(TOOLS)/onlynew $(TARGETDIR) | \
			cpio -pdamvu $(TARGETDIR) && \
		popd; \
	    else \
		echo ------ Installing $$n; \
		if [ -f $(SOURCEDIR)/$$n.build ]; \
		    then build="$(SOURCEDIR)/$$n.build"; \
		    else build="$(SOURCEDIR)/`echo $$n | \
				sed 's/-.*$$//'`.build"; \
		fi; \
		$(TOOLS)/extract-install-script $$build > tmp/$$n.install; \
		export TARGET="$(TARGETDIR)"; \
		(cd `$(TOOLS)/getsource $$n` && sh $X $(BUILDDIR)/tmp/$$n.install); \
	    fi; \
	done
	@$(TOOLS)/runcustom $(CFG_CUSTOM) $(CUSTOM)

$(HTML): $(HTMLHEAD) $(HTMLTAIL) $(HTMLMID) \
		$(wildcard $(SOURCEDIR)/*.build $(SOURCEDIR)/Configs/*)
	@echo Creating html file $@
	@-mkdir `dirname $(HTML)` 2> /dev/null || true
	cat $(HTMLHEAD) > $@
	@for n in $(BASE) $(PACKAGES); do \
	    if [ -f $(SOURCEDIR)/$$n.build ]; \
		then bf="$(SOURCEDIR)/$$n.build"; \
		else bf="$(SOURCEDIR)/`echo $$n | sed 's/-.*$$//'`.build"; \
	    fi; \
	    echo "Extracting html description from   `basename $$bf`"; \
	    $(TOOLS)/extract-htmldesc $$bf | \
		sed -e "s/__PACKAGE__/$$n/" -e "s/__CONFIG__/$(CONFIG)/">>$@; \
	done
	cat $(HTMLMID) >> $@
	@for n in $(BASE) $(PACKAGES); do \
	    if [ -f $(SOURCEDIR)/$$n.build ]; then \
		bf="$(SOURCEDIR)/$$n.build"; \
		else bf="$(SOURCEDIR)/`echo $$n | sed 's/-.*$$//'`.build"; \
	fi; \
	    echo "Extracting html listings from   `basename $$bf`"; \
	    $(TOOLS)/extract-htmllist $$bf | \
		sed -e "s/__PACKAGE__/$$n/" -e "s/__CONFIG__/$(CONFIG)/">>$@; \
	done
	cat $(HTMLTAIL) >> $@


terse: Makefile.auto
	$(MAKE) do.terse

do.terse: checkbuild
	rm -rf $(BASE) $(PACKAGES) $(BUILDDIR)/tmp
	rm -f *~

clean: Makefile.auto
	$(MAKE) do.clean
	rm -f Makefile.auto
	rm -f $(BUILDDIR)/tmp/*.built

do.clean: do.terse
	rm -rf $(TARGETDIR) $(HTML)
	#Remove binary files, if source is there
	for n in *.bin; do \
		pkg=`basename $$n .bin`; \
		source=`$(TOOLS)/getsource $$pkg`; \
		if [ -f  $(SOURCEDIR)/$$source.tar.gz ]; then \
			echo "rm -rf $$n"; rm -rf $$n; \
		fi; \
	done
	#Remove extracted trees, if source is there
	@for n in *.tar.gz; do \
		d=`basename $$n .tar.gz`; \
		if [ -d $(SOURCEDIR)/$$d ]; then \
			echo "rm -rf $$d"; \
			rm -rf $$d; \
		fi; \
	done

nosource: Makefile.auto
	$(MAKE) do.nosource

do.nosource:
	# Remove .tar.gz and .patch, whereever .bin is there
	@for n in *.bin; do \
		t=`basename $$n .bin`.tar.gz; \
		p=`basename $$n .bin`.patch; \
		if [ -r $(SOURCEDIR)/$$t ]; then echo "rm  $(SOURCEDIR)/$$t"; rm $(SOURCEDIR)/$$t; fi; \
		if [ -r $(SOURCEDIR)/$$p ]; then echo "rm  $(SOURCEDIR)/$$p"; rm $(SOURCEDIR)/$$p; fi; \
	done

cleanest: Makefile.auto
	$(MAKE) do.cleanest
	rm -f Makefile.auto

do.clanest: do.clean
	@for n in *.tar.gz; do \
		if [ -d `basename $$n .tar.gz` ]; then \
			rm -rf `basename $$n .tar.gz`; \
		fi; \
	done

ifeq (Makefile.auto,$(wildcard Makefile.auto))
include Makefile.auto
endif
