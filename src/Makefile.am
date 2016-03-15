AM_CPPFLAGS = @VMOD_INCLUDES@ -Wall -Werror

vmoddir = @VMOD_DIR@
vmod_LTLIBRARIES = libvmod_bofh.la

libvmod_bofh_la_LDFLAGS = -module -export-dynamic -avoid-version -shared

libvmod_bofh_la_SOURCES = \
	vmod_bofh.c

nodist_libvmod_bofh_la_SOURCES = \
	vcc_if.c \
	vcc_if.h

dist_man_MANS = vmod_bofh.3

vmod_bofh.3: vmod_bofh.man.rst
	${RST2MAN} $< $@


vmod_bofh.lo vmod_bofh.man.rst: vcc

vcc: @VMODTOOL@ $(top_srcdir)/src/vmod_bofh.vcc
	@PYTHON@ @VMODTOOL@ $(top_srcdir)/src/vmod_bofh.vcc

VMOD_TESTS = $(top_srcdir)/src/tests/*.vtc
.PHONY: $(VMOD_TESTS) vcc

$(top_srcdir)/src/tests/*.vtc: libvmod_bofh.la
	@VARNISHTEST@ -Dvarnishd=@VARNISHD@ -Dvmod_topbuild=$(abs_top_builddir) $@

check: $(VMOD_TESTS)

EXTRA_DIST = \
	vmod_bofh.vcc \
	$(VMOD_TESTS)

CLEANFILES = \
	$(builddir)/vcc_if.c \
	$(builddir)/vcc_if.h \
	$(builddir)/vmod_bofh.rst \
	$(builddir)/vmod_bofh.man.rst \
	$(dist_man_MANS)
