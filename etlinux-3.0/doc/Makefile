#
# Makefile for the documentation directory
#
# Copyright 1994,2000   Alessandro Rubini <rubini@linux.it>
#
#################
#
# BE CAREFUL in editing:
#	due to the large number of index files, and my use of a non standard
#	info input file, any file $(TARGET).* is removed by "make clean"
#
#	I chose to use a prefix for the input file ("doc.$(TARGET)"), to ease
#	makeing clean and applying my own rules.
#
###################################################################
#
# Assume makeinfo can do images and --html.
# In any case, MAKEINFO can be specified on the commandline

MAKEINFO = makeinfo

##############################################


%.texinfo: doc.%
	@rm -f $@ 2> /dev/null
	sed -f ./infofilter $< > $@
	chmod 400 $@


# This rule is somewhat a rewrite of texi2dvi. I like make more than sh :-)

%.dvi: %.texinfo

# create a spurious  index file to please silly sh (bash will work anyway)
	touch $*.oo

# get the index list
	if test "x`ls $*.?? $*.aux`" != "x"; then \
	  for i in `ls $*.?? $*.aux`; do \
	    cp $$i $$i~; \
	  done; \
	fi
	tex $<

#check the file list, the file and if needed run TeX again
	old="`ls $*.??~ $*.aux~ | sed 's/~//g'`"; \
	new="`ls $*.?? $*.aux`"; \
	need="n"; \
	if test "$$old" != "$$new"; then need="y"; \
	else \
	  for i in  `ls $*.?? $*.aux`; do \
	    cmp -s $$i $$i~; if test $$? -ne 0; then need="y" break; fi; \
	  done; \
	fi; \
	if test "$$need" = "y"; then \
	  texindex $*.?? && tex $<; \
	fi


%.ps: %.dvi
	dvips -f $< > $@

%.pdf: %.ps
	ps2pdf $< > $@

%.lj: %.dvi
	dvilj -e- $< > $@

%.info: %.texinfo
	$(MAKEINFO) $< -o $@

%.html: %.texinfo
	$(MAKEINFO) --html --no-split -o $@ $<

#%.man: doc.%
#		manpages are created by the toplevel Makefile

%doc.txt: %.texinfo
	$(MAKEINFO) --no-headers $< > $@

##############################################


TARGET = etlinux
ALL    = $(TARGET).info $(TARGET).ps $(TARGET)doc.txt $(TARGET).html \
		$(TARGET).pdf
# shortcuts
PS = $(TARGET).ps
PDF = $(TARGET).pdf

all: images $(ALL) terse

images::
	if [ -d images ]; then $(MAKE) -C images || exit 1; fi

info: $(TARGET).info

check: _err.ps
	gs -sDEVICE=linux -r320x200x16 $<

gs: $(TARGET).ps
	gs -sDEVICE=linux -r640x480x2 $<

gv: $(TARGET).ps
	ghostview $(TARGET).ps -magstep -1 -a4

mpage: all
	mv -f $(TARGET).ps $(TARGET)1.ps
	mpage -2AP $(TARGET)1.ps > $(TARGET)2.ps && rm $(TARGET)1.ps

terse:
	rm -f *~ *.dvi *.log *.aux $(TARGET).*.bak $(TARGET).texinfo
# preserve the pdf copy
	if [ -f $(PDF) ]; then \
		mv -f $(PDF) PDF && rm -f $(TARGET).??? && mv PDF $(PDF); \
	else rm -f  $(TARGET).???; fi
# preserve the ps copy
	if [ -f $(PS) ]; then \
		mv -f $(PS) PS && rm -f $(TARGET).?? && mv PS $(PS); \
	else rm -f  $(TARGET).??; fi
	if [ -d images ]; then $(MAKE) -C images terse; fi

clean: terse
	rm -f $(ALL) $(TARGET).info*
	if [ -d images ]; then $(MAKE) -C images clean; fi

install:
	# nothing, it's installed from above
