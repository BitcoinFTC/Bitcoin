# Copyright (c) 2013-2016 The Bitcoin Core developers
# Copyright (c) 2019 Bitcoin Association
# Distributed under the Open BSV software license, see the accompanying file LICENSE.

ACLOCAL_AMFLAGS = -I build-aux/m4
SUBDIRS = src
if ENABLE_MAN
SUBDIRS += doc/man
endif

export PYTHONPATH

.PHONY: deploy FORCE

GZIP_ENV="-9n"

if BUILD_BITCOIN_LIBS
pkgconfigdir = $(libdir)/pkgconfig
pkgconfig_DATA = libbitcoinconsensus.pc
endif

BITCOIND_BIN=$(top_builddir)/src/$(BITCOIN_DAEMON_NAME)$(EXEEXT)
BITCOIN_CLI_BIN=$(top_builddir)/src/$(BITCOIN_CLI_NAME)$(EXEEXT)
BITCOIN_WIN_INSTALLER=$(PACKAGE)-$(PACKAGE_VERSION)-win$(WINDOWS_BITS)-setup$(EXEEXT)

empty :=
space := $(empty) $(empty)

DIST_DOCS = $(wildcard doc/*.md) $(wildcard doc/release-notes/*.md)
DIST_CONTRIB = $(top_srcdir)/contrib/bitcoin-cli.bash-completion \
	       $(top_srcdir)/contrib/bitcoin-tx.bash-completion \
	       $(top_srcdir)/contrib/bitcoind.bash-completion \
	       $(top_srcdir)/contrib/init

BIN_CHECKS=$(top_srcdir)/contrib/devtools/symbol-check.py \
           $(top_srcdir)/contrib/devtools/security-check.py

BUILD_FILES=$(top_srcdir)/share/genbuild.sh \
            $(top_srcdir)/share/on_prod_tag.sh

WINDOWS_PACKAGING = $(top_srcdir)/share/pixmaps/bitcoin.ico \
  $(top_srcdir)/share/pixmaps/nsis-header.bmp \
  $(top_srcdir)/share/pixmaps/nsis-wizard.bmp \
  $(top_srcdir)/doc/README_windows.txt

COVERAGE_INFO = baseline_filtered_combined.info baseline.info \
  leveldb_baseline.info test_bitcoin_filtered.info total_coverage.info \
  baseline_filtered.info functional_test.info functional_test_filtered.info \
  leveldb_baseline_filtered.info test_bitcoin_coverage.info test_bitcoin.info

dist-hook:
	-$(GIT) archive --format=tar HEAD -- src/clientversion.cpp | $(AMTAR) -C $(top_distdir) -xf -

$(BITCOIN_WIN_INSTALLER): all-recursive
	$(MKDIR_P) $(top_builddir)/release
	STRIPPROG="$(STRIP)" $(INSTALL_STRIP_PROGRAM) $(BITCOIND_BIN) $(top_builddir)/release
	STRIPPROG="$(STRIP)" $(INSTALL_STRIP_PROGRAM) $(BITCOIN_CLI_BIN) $(top_builddir)/release
	@test -f $(MAKENSIS) && $(MAKENSIS) -V2 $(top_builddir)/share/setup.nsi || \
	  echo error: could not build $@
	@echo built $@

$(if $(findstring src/,$(MAKECMDGOALS)),$(MAKECMDGOALS), none): FORCE
	$(MAKE) -C src $(patsubst src/%,%,$@)

if TARGET_WINDOWS
deploy: $(BITCOIN_WIN_INSTALLER)
endif

$(BITCOIND_BIN): FORCE
	$(MAKE) -C src $(@F)

$(BITCOIN_CLI_BIN): FORCE
	$(MAKE) -C src $(@F)

if USE_LCOV

baseline.info:
	$(MAKE) -C src/
	$(LCOV) -c -i -d $(abs_builddir)/src -o $@

baseline_filtered.info: baseline.info
	$(LCOV) -r $< "/usr/include/*" -o $@

leveldb_baseline.info: baseline_filtered.info
	$(LCOV) -c -i -d $(abs_builddir)/src/leveldb -b $(abs_builddir)/src/leveldb -o $@

leveldb_baseline_filtered.info: leveldb_baseline.info
	$(LCOV) -r $< "/usr/include/*" -o $@

baseline_filtered_combined.info: leveldb_baseline_filtered.info baseline_filtered.info
	$(LCOV) -a leveldb_baseline_filtered.info -a baseline_filtered.info -o $@

test_bitcoin.info: baseline_filtered_combined.info
	$(MAKE) -C src/ check
	$(LCOV) -c -d $(abs_builddir)/src -t test_bitcoin -o $@
	$(LCOV) -z -d $(abs_builddir)/src
	$(LCOV) -z -d $(abs_builddir)/src/leveldb

test_bitcoin_filtered.info: test_bitcoin.info
	$(LCOV) -r $< "/usr/include/*" -o $@

functional_test.info: test_bitcoin_filtered.info
	-@TIMEOUT=15 test/functional/test_runner.py $(EXTENDED_FUNCTIONAL_TESTS)
	$(LCOV) -c -d $(abs_builddir)/src --t functional-tests -o $@
	$(LCOV) -z -d $(abs_builddir)/src
	$(LCOV) -z -d $(abs_builddir)/src/leveldb

functional_test_filtered.info: functional_test.info
	$(LCOV) -r $< "/usr/include/*" -o $@

test_bitcoin_coverage.info: baseline_filtered_combined.info test_bitcoin_filtered.info
	$(LCOV) -a baseline_filtered.info -a leveldb_baseline_filtered.info -a test_bitcoin_filtered.info -o $@

total_coverage.info: baseline_filtered_combined.info test_bitcoin_filtered.info functional_test_filtered.info
	$(LCOV) -a baseline_filtered.info -a leveldb_baseline_filtered.info -a test_bitcoin_filtered.info -a functional_test_filtered.info -o $@ | $(GREP) "\%" | $(AWK) '{ print substr($$3,2,50) "/" $$5 }' > coverage_percent.txt

test_bitcoin.coverage/.dirstamp:  test_bitcoin_coverage.info
	$(GENHTML) -s $< -o $(@D)
	@touch $@

total.coverage/.dirstamp: total_coverage.info
	$(GENHTML) -s $< -o $(@D)
	@touch $@

cov: test_bitcoin.coverage/.dirstamp total.coverage/.dirstamp

endif

dist_noinst_SCRIPTS = autogen.sh

EXTRA_DIST = $(BUILD_FILES) test/functional/test_runner.py test/functional $(DIST_CONTRIB) $(DIST_DOCS) $(WINDOWS_PACKAGING) $(BIN_CHECKS)

EXTRA_DIST += \
	test/util/bitcoin-util-test.py \
	test/util/data/bitcoin-util-test.json \
	test/util/data/blanktxv1.hex \
	test/util/data/blanktxv1.json \
	test/util/data/blanktxv2.hex \
	test/util/data/blanktxv2.json \
	test/util/data/tt-delin1-out.hex \
	test/util/data/tt-delin1-out.json \
	test/util/data/tt-delout1-out.hex \
	test/util/data/tt-delout1-out.json \
	test/util/data/tt-locktime317000-out.hex \
	test/util/data/tt-locktime317000-out.json \
	test/util/data/tx394b54bb.hex \
	test/util/data/txcreate1.hex \
	test/util/data/txcreate1.json \
	test/util/data/txcreate2.hex \
	test/util/data/txcreate2.json \
	test/util/data/txcreatedata1.hex \
	test/util/data/txcreatedata1.json \
	test/util/data/txcreatedata2.hex \
	test/util/data/txcreatedata2.json \
	test/util/data/txcreatedata_seq0.hex \
	test/util/data/txcreatedata_seq0.json \
	test/util/data/txcreatedata_seq1.hex \
	test/util/data/txcreatedata_seq1.json \
	test/util/data/txcreatemultisig1.hex \
	test/util/data/txcreatemultisig1.json \
	test/util/data/txcreateoutpubkey1.hex \
	test/util/data/txcreateoutpubkey1.json \
	test/util/data/txcreatescript1.hex \
	test/util/data/txcreatescript1.json \
	test/util/data/txcreatesignv1.hex \
	test/util/data/txcreatesignv1.json \
	test/util/data/txcreatesignv2.hex

CLEANFILES = $(BITCOIN_WIN_INSTALLER)

.INTERMEDIATE: $(COVERAGE_INFO)

DISTCHECK_CONFIGURE_FLAGS = --enable-man

clean-local:
	rm -rf coverage_percent.txt test_bitcoin.coverage/ total.coverage/ test/tmp/ cache/ 
	rm -rf test/functional/__pycache__
