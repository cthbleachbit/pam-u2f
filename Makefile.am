#  Copyright (C) 2014-2015 Yubico AB - See COPYING

SUBDIRS = po . pamu2fcfg tests

ACLOCAL_AMFLAGS = -I m4

AM_CFLAGS = $(CWFLAGS) $(CFLAGS)
AM_CPPFLAGS = -I$(srcdir)/.. -I$(builddir)/.. $(LIBU2FHOST_CFLAGS) $(LIBU2FSERVER_CFLAGS)

libdir = $(PAMDIR)

lib_LTLIBRARIES = pam_u2f.la

pam_u2f_la_SOURCES = pam-u2f.c
pam_u2f_la_SOURCES += util.c util.h

pam_u2f_la_LIBADD = -lpam
pam_u2f_la_LIBADD += $(LIBU2FHOST_LIBS) $(LIBU2FSERVER_LIBS)
pam_u2f_la_LDFLAGS = -module -avoid-version

DEFS = -DDEBUG_PAM -DPAM_DEBUG @DEFS@

if ENABLE_MAN
dist_man8_MANS = $(top_builddir)/man/pam_u2f.8
DISTCLEANFILES = $(dist_man8_MANS)

MANSOURCES = $(top_builddir)/man/pam_u2f.8.txt
EXTRA_DIST = build-aux/config.rpath  $(MANSOURCES)

SUFFIXES = .8.txt .8

.8.txt.8:
	$(A2X) --format=manpage -L -a revdate="Version $(VERSION)" $<
endif

# Release

install-exec-hook:
	rm -f $(PAMDIR)/pam_u2f.la
	chmod -f 644 $(PAMDIR)/pam_u2f.so || true

indent:
	clang-format -i *.c *.h pamu2fcfg/pamu2fcfg.c

ChangeLog:
	cd $(srcdir) && git2cl > ChangeLog

PROJECT = pam-u2f

release:
	@if test ! -d "$(YUBICO_WWW_REPO)"; then \
		echo "WWW repo not found!"; \
		echo "Make sure that YUBICO_WWW_REPO is set"; \
		exit 1; \
	fi
	@if test -z "$(KEYID)"; then \
		echo "Try this instead:"; \
		echo "  make release KEYID=[PGPKEYID]"; \
		echo "For example:"; \
		echo "  make release KEYID=2117364A"; \
		exit 1; \
	fi
	@head -5 $(srcdir)/NEWS | \
		grep -q "Version $(VERSION) .released `date -I`" || \
		(echo 'error: Update date/version in $(srcdir)/NEWS.'; exit 1)
	rm -f $(srcdir)/ChangeLog
	sudo make ChangeLog distcheck
	gpg --detach-sign --default-key $(KEYID) $(PACKAGE)-$(VERSION).tar.gz
	gpg --verify $(PACKAGE)-$(VERSION).tar.gz.sig
	cd $(srcdir) && git push
	cd $(srcdir) && git tag -u $(KEYID) -m $(VERSION) $(PACKAGE)-$(VERSION)
	cd $(srcdir) && git push --tags
	$(YUBICO_WWW_REPO)/publish $(PROJECT) $(VERSION) $(PACKAGE)-$(VERSION).tar.gz*
