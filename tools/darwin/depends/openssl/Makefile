include ../Makefile.include
include ../config.site.mk
DEPS= Makefile

# lib name, version
LIBNAME=openssl
VERSION=1.0.2h
SOURCE=$(LIBNAME)-$(VERSION)
ARCHIVE=$(SOURCE).tar.gz

# configuration settings
ifeq ($(DARWIN), ios)
  # No darwin-arm-cc so use darwin-i386-cc and patch files after configure
  CONFIGURE=./Configure darwin-i386-cc zlib no-asm no-krb5 shared --openssldir=$(PREFIX)
else
  ifeq ($(ARCH), x86_64)
    CONFIGURE=./Configure darwin64-$(ARCH)-cc zlib no-asm no-krb5 shared --openssldir=$(PREFIX)
  else
    CONFIGURE=./Configure darwin-$(ARCH)-cc zlib no-asm no-krb5 shared --openssldir=$(PREFIX)
  endif
endif

LIBDYLIB=$(SOURCE)/libssl.dylib

all: .installed

$(TARBALLS_LOCATION)/$(ARCHIVE):
	$(RETRIEVE_TOOL) $(RETRIEVE_TOOL_FLAGS) $(BASE_URL)/$(ARCHIVE)

$(SOURCE): $(TARBALLS_LOCATION)/$(ARCHIVE) $(DEPS)
	-rm -rf $(SOURCE)
	$(ARCHIVE_TOOL) $(ARCHIVE_TOOL_FLAGS) $(TARBALLS_LOCATION)/$(ARCHIVE)
	echo $(SOURCE) > .gitignore
	cd $(SOURCE); $(CONFIGURE)
	if test "$(DARWIN)" = "osx"; then \
		sed -ie "s|CC= /usr/bin/gcc-4.2|CC= /usr/bin/gcc|" "$(SOURCE)/Makefile"; \
	fi
	if test "$(DARWIN)" = "ios"; then \
		sed -ie "s|CFLAG= |CFLAG=-arch armv7 -isysroot ${platform_sdk_path} |" "$(SOURCE)/Makefile"; \
		sed -ie "s|-arch i386|-arch armv7|" "$(SOURCE)/Makefile"; \
		sed -ie "s|static volatile sig_atomic_t intr_signal;|static volatile intr_signal;|" "$(SOURCE)/crypto/ui/ui_openssl.c"; \
	fi

$(LIBDYLIB): $(SOURCE)
	make -j 1 -C $(SOURCE)

.installed: $(LIBDYLIB)
	make -C $(SOURCE) install_sw
	touch $@

clean:
	make -C $(SOURCE) clean
	rm -f .installed

distclean::
	rm -rf $(SOURCE) .installed
