# Copyright (c) 2013-2016 The Bitcoin Core developers
# Distributed under the MIT software license, see the accompanying
# file COPYING or http://www.opensource.org/licenses/mit-license.php.

ACLOCAL_AMFLAGS = -I build-aux/m4
SUBDIRS = src
if ENABLE_MAN
SUBDIRS += doc/man
endif
.PHONY: deploy FORCE

GZIP_ENV="-9n"
export PYTHONPATH

if BUILD_CATCOIN_LIBS
pkgconfigdir = $(libdir)/pkgconfig
pkgconfig_DATA = libcatcoinconsensus.pc
endif

CATCOIND_BIN=$(top_builddir)/src/$(CATCOIN_DAEMON_NAME)$(EXEEXT)
CATCOIN_QT_BIN=$(top_builddir)/src/qt/$(CATCOIN_GUI_NAME)$(EXEEXT)
CATCOIN_CLI_BIN=$(top_builddir)/src/$(CATCOIN_CLI_NAME)$(EXEEXT)
CATCOIND_WIN_INSTALLER=$(PACKAGE)-$(PACKAGE_VERSION)-win$(WINDOWS_BITS)-setup$(EXEEXT)

empty :=
space := $(empty) $(empty)

OSX_APP=Catcoin-Qt.app
OSX_VOLNAME = $(subst $(space),-,$(PACKAGE_NAME))
OSX_DMG = $(OSX_VOLNAME).dmg
OSX_DSSTORE_GEN=$(top_srcdir)/contrib/macdeploy/custom_dsstore.py
OSX_DEPLOY_SCRIPT=$(top_srcdir)/contrib/macdeploy/macdeployqtplus
OSX_FANCY_PLIST=$(top_srcdir)/contrib/macdeploy/fancy.plist
OSX_INSTALLER_ICONS=$(top_srcdir)/src/qt/res/icons/catcoin.icns
OSX_PLIST=$(top_builddir)/share/qt/Info.plist #not installed
OSX_QT_TRANSLATIONS = da,de,es,hu,ru,uk,zh_CN,zh_TW

DIST_DOCS = $(wildcard doc/*.md) $(wildcard doc/release-notes/*.md)
DIST_CONTRIB = $(top_srcdir)/contrib/catcoin-cli.bash-completion \
		$(top_srcdir)/contrib/catcoin-tx.bash-completion \
		$(top_srcdir)/contrib/catcoind.bash-completion \
		$(top_srcdir)/contrib/init \
		$(top_srcdir)/contrib/rpm

BIN_CHECKS=$(top_srcdir)/contrib/devtools/symbol-check.py \
		$(top_srcdir)/contrib/devtools/security-check.py

WINDOWS_PACKAGING = $(top_srcdir)/share/pixmaps/catcoin.ico \
	$(top_srcdir)/share/pixmaps/nsis-header.bmp \
	$(top_srcdir)/share/pixmaps/nsis-wizard.bmp \
	$(top_srcdir)/doc/README_windows.txt

OSX_PACKAGING = $(OSX_DEPLOY_SCRIPT) $(OSX_FANCY_PLIST) $(OSX_INSTALLER_ICONS) \
	$(top_srcdir)/contrib/macdeploy/background.png \
	$(OSX_DSSTORE_GEN) \
	$(top_srcdir)/contrib/macdeploy/detached-sig-apply.sh \
	$(top_srcdir)/contrib/macdeploy/detached-sig-create.sh

COVERAGE_INFO = baseline_filtered_combined.info baseline.info \
	leveldb_baseline.info test_catcoin_filtered.info total_coverage.info \
	baseline_filtered.info functional_test.info functional_test_filtered.info \
	leveldb_baseline_filtered.info test_catcoin_coverage.info test_catcoin.info

dist-hook:
	-$(GIT) archive --format=tar HEAD -- src/clientversion.cpp | $(AMTAR) -C $(top_distdir) -xf -

$(CATCOIND_WIN_INSTALLER): all-recursive
	$(MKDIR_P) $(top_builddir)/release
	STRIPPROG="$(STRIP)" $(INSTALL_STRIP_PROGRAM) $(CATCOIND_BIN) $(top_builddir)/release
	STRIPPROG="$(STRIP)" $(INSTALL_STRIP_PROGRAM) $(CATCOIN_QT_BIN) $(top_builddir)/release
	@test -f $(MAKENSIS) && $(MAKENSIS) -V2 $(top_builddir)/share/setup.nsi || \
		echo error: could not build $@
	@echo built $@

$(OSX_APP)/Contents/PkgInfo:
	$(MKDIR_P) $(@D)
	@echo "APPL????" > $@

$(OSX_APP)/Contents/Resources/empty.lproj:
	$(MKDIR_P) $(@D)
	@touch $@ 

$(OSX_APP)/Contents/Info.plist: $(OSX_PLIST)
	$(MKDIR_P) $(@D)
	$(INSTALL_DATA) $< $@

$(OSX_APP)/Contents/Resources/catcoin.icns: $(OSX_INSTALLER_ICONS)
	$(MKDIR_P) $(@D)
	$(INSTALL_DATA) $< $@

$(OSX_APP)/Contents/MacOS/Catcoin-Qt: $(CATCOIN_QT_BIN)
	$(MKDIR_P) $(@D)
	STRIPPROG="$(STRIP)" $(INSTALL_STRIP_PROGRAM)  $< $@

$(OSX_APP)/Contents/Resources/Base.lproj/InfoPlist.strings:
	$(MKDIR_P) $(@D)
	echo '{	CFBundleDisplayName = "$(PACKAGE_NAME)"; CFBundleName = "$(PACKAGE_NAME)"; }' > $@

OSX_APP_BUILT=$(OSX_APP)/Contents/PkgInfo $(OSX_APP)/Contents/Resources/empty.lproj \
	$(OSX_APP)/Contents/Resources/catcoin.icns $(OSX_APP)/Contents/Info.plist \
	$(OSX_APP)/Contents/MacOS/Catcoin-Qt $(OSX_APP)/Contents/Resources/Base.lproj/InfoPlist.strings

osx_volname:
	echo $(OSX_VOLNAME) >$@

if BUILD_DARWIN
$(OSX_DMG): $(OSX_APP_BUILT) $(OSX_PACKAGING)
	$(PYTHON) $(OSX_DEPLOY_SCRIPT) $(OSX_APP) -add-qt-tr $(OSX_QT_TRANSLATIONS) -translations-dir=$(QT_TRANSLATION_DIR) -dmg -fancy $(OSX_FANCY_PLIST) -verbose 2 -volname $(OSX_VOLNAME)

deploydir: $(OSX_DMG)
else
APP_DIST_DIR=$(top_builddir)/dist
APP_DIST_EXTRAS=$(APP_DIST_DIR)/.background/background.png $(APP_DIST_DIR)/.DS_Store $(APP_DIST_DIR)/Applications

$(APP_DIST_DIR)/Applications:
	@rm -f $@
	@cd $(@D); $(LN_S) /Applications $(@F)

$(APP_DIST_EXTRAS): $(APP_DIST_DIR)/$(OSX_APP)/Contents/MacOS/Catcoin-Qt

$(OSX_DMG): $(APP_DIST_EXTRAS)
	$(GENISOIMAGE) -no-cache-inodes -D -l -probe -V "$(OSX_VOLNAME)" -no-pad -r -dir-mode 0755 -apple -o $@ dist

$(APP_DIST_DIR)/.DS_Store: $(OSX_DSSTORE_GEN)
	$(PYTHON) $< "$@" "$(OSX_VOLNAME)"

$(APP_DIST_DIR)/$(OSX_APP)/Contents/MacOS/Catcoin-Qt: $(OSX_APP_BUILT) $(OSX_PACKAGING)
	INSTALLNAMETOOL=$(INSTALLNAMETOOL)  OTOOL=$(OTOOL) STRIP=$(STRIP) $(PYTHON) $(OSX_DEPLOY_SCRIPT) $(OSX_APP) -translations-dir=$(QT_TRANSLATION_DIR) -add-qt-tr $(OSX_QT_TRANSLATIONS) -verbose 2

deploydir: $(APP_DIST_EXTRAS)
endif

if TARGET_DARWIN
appbundle: $(OSX_APP_BUILT)
deploy: $(OSX_DMG)
endif
if TARGET_WINDOWS
deploy: $(CATCOIND_WIN_INSTALLER)
endif

$(CATCOIN_QT_BIN): FORCE
	$(MAKE) -C src qt/$(@F)

$(CATCOIND_BIN): FORCE
	$(MAKE) -C src $(@F)

$(CATCOIN_CLI_BIN): FORCE
	$(MAKE) -C src $(@F)

if USE_LCOV

baseline.info:
	$(LCOV) -c -i -d $(abs_builddir)/src -o $@

baseline_filtered.info: baseline.info
	$(LCOV) -r $< "/usr/include/*" -o $@

leveldb_baseline.info: baseline_filtered.info
	$(LCOV) -c -i -d $(abs_builddir)/src/leveldb -b $(abs_builddir)/src/leveldb -o $@

leveldb_baseline_filtered.info: leveldb_baseline.info
	$(LCOV) -r $< "/usr/include/*" -o $@

baseline_filtered_combined.info: leveldb_baseline_filtered.info baseline_filtered.info
	$(LCOV) -a leveldb_baseline_filtered.info -a baseline_filtered.info -o $@

test_catcoin.info: baseline_filtered_combined.info
	$(MAKE) -C src/ check
	$(LCOV) -c -d $(abs_builddir)/src -t test_catcoin -o $@
	$(LCOV) -z -d $(abs_builddir)/src
	$(LCOV) -z -d $(abs_builddir)/src/leveldb

test_catcoin_filtered.info: test_catcoin.info
	$(LCOV) -r $< "/usr/include/*" -o $@

functional_test.info: test_catcoin_filtered.info
	-@TIMEOUT=15 python test/functional/test_runner.py $(EXTENDED_FUNCTIONAL_TESTS)
	$(LCOV) -c -d $(abs_builddir)/src --t functional-tests -o $@
	$(LCOV) -z -d $(abs_builddir)/src
	$(LCOV) -z -d $(abs_builddir)/src/leveldb

functional_test_filtered.info: functional_test.info
	$(LCOV) -r $< "/usr/include/*" -o $@

test_catcoin_coverage.info: baseline_filtered_combined.info test_catcoin_filtered.info
	$(LCOV) -a baseline_filtered.info -a leveldb_baseline_filtered.info -a test_catcoin_filtered.info -o $@

total_coverage.info: baseline_filtered_combined.info test_catcoin_filtered.info functional_test_filtered.info
	$(LCOV) -a baseline_filtered.info -a leveldb_baseline_filtered.info -a test_catcoin_filtered.info -a functional_test_filtered.info -o $@ | $(GREP) "\%" | $(AWK) '{ print substr($$3,2,50) "/" $$5 }' > coverage_percent.txt

test_catcoin.coverage/.dirstamp:  test_catcoin_coverage.info
	$(GENHTML) -s $< -o $(@D)
	@touch $@

total.coverage/.dirstamp: total_coverage.info
	$(GENHTML) -s $< -o $(@D)
	@touch $@

cov: test_catcoin.coverage/.dirstamp total.coverage/.dirstamp

endif

dist_noinst_SCRIPTS = autogen.sh

EXTRA_DIST = $(top_srcdir)/share/genbuild.sh test/functional/test_runner.py test/functional $(DIST_CONTRIB) $(DIST_DOCS) $(WINDOWS_PACKAGING) $(OSX_PACKAGING) $(BIN_CHECKS)

EXTRA_DIST += \
	test/util/catcoin-util-test.py \
	test/util/data/catcoin-util-test.json \
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
	test/util/data/txcreatemultisig2.hex \
	test/util/data/txcreatemultisig2.json \
	test/util/data/txcreatemultisig3.hex \
	test/util/data/txcreatemultisig3.json \
	test/util/data/txcreatemultisig4.hex \
	test/util/data/txcreatemultisig4.json \
	test/util/data/txcreateoutpubkey1.hex \
	test/util/data/txcreateoutpubkey1.json \
	test/util/data/txcreateoutpubkey2.hex \
	test/util/data/txcreateoutpubkey2.json \
	test/util/data/txcreateoutpubkey3.hex \
	test/util/data/txcreateoutpubkey3.json \
	test/util/data/txcreatescript1.hex \
	test/util/data/txcreatescript1.json \
	test/util/data/txcreatescript2.hex \
	test/util/data/txcreatescript2.json \
	test/util/data/txcreatescript3.hex \
	test/util/data/txcreatescript3.json \
	test/util/data/txcreatescript4.hex \
	test/util/data/txcreatescript4.json \
	test/util/data/txcreatesignv1.hex \
	test/util/data/txcreatesignv1.json \
	test/util/data/txcreatesignv2.hex

CLEANFILES = $(OSX_DMG) $(CATCOIND_WIN_INSTALLER)

.INTERMEDIATE: $(COVERAGE_INFO)

DISTCHECK_CONFIGURE_FLAGS = --enable-man

clean-local:
	rm -rf coverage_percent.txt test_catcoin.coverage/ total.coverage/ test/tmp/ cache/ $(OSX_APP)
	rm -rf test/functional/__pycache__
