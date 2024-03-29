VERSION = $(shell test -d .git && git describe --tags --dirty 2>/dev/null)
ifeq "$(VERSION)" ""
VERSION = 31
endif

# paths
PREFIX ?= /usr/local

# includes and libs
PKGCONF_DEPS := freetype2 harfbuzz libopenjp2 libjpeg x11 zlib
CPPFLAGS += -D_GNU_SOURCE -DFFP
CFLAGS += -g -std=c99 -pedantic -Wall -Wextra -Wshadow $(shell pkg-config --cflags $(PKGCONF_DEPS))
LDLIBS = -lpthread -lmupdf -lmupdf-third -ljbig2dec $(shell pkg-config --libs $(PKGCONF_DEPS))

# ocaml
CAMLOPT = ocamlopt
CAMLFLAGS = -g -w +a -safe-string -I +lablGL

VPATH = wsi/x11

C_SRC = cutils.c keysym2ucs.c link.c xlib.c version.c
C_OBJ = $(C_SRC:.c=.o)
OCAML_SRC = utils.ml wsi.ml confstruct.ml parser.ml config.ml ffi.ml glutils.ml help.ml keys.ml utf8syms.ml listview.ml main.ml
OCAML_OBJ = $(OCAML_SRC:.ml=.cmx)
MOD = unix.cmxa str.cmxa lablgl.cmxa
SRCMANS = $(wildcard adoc/*.adoc)
MANS = $(SRCMANS:.adoc=.1)

DISTFILES := Makefile $(OCAML_SRC) link.c glfont.c keysym2ucs.c wsi.mli
DISTFILES += $(wildcard *.sh) KEYS README BUILDING
DISTFILES += misc/ adoc/

all: llpp $(MANS)

# dependency ordering
config.cmx: wsi.cmi parser.cmx utils.cmx confstruct.cmx
confstruct.cmx: wsi.cmx utils.cmx
ffi.cmx: config.cmx
glutils.cmx: ffi.cmx
help.cmx: help.cmi config.cmx utils.cmx
listview.cmx: utils.cmx glutils.cmx config.cmx utf8syms.cmx
main.cmx: main.ml main.cmi utils.cmx config.cmx glutils.cmx listview.cmx ffi.cmx keys.cmx wsi.cmx
main.cmx: CAMLFLAGS += -thread
parser.cmx: utils.cmx
wsi.cmi: utils.cmx keys.cmx
wsi.cmx: wsi.cmi

link.o: glfont.c
version.o: CPPFLAGS += -DLLPP_VERSION=$(VERSION)

# ordinary targets
llpp: $(OCAML_OBJ) $(C_OBJ)
	$(CAMLOPT) -o $@ $(CAMLFLAGS) $(C_OBJ) -ccopt '$(LDFLAGS)' -cclib '$(LDLIBS)' $(MOD) $(OCAML_OBJ)


confstruct.ml: genconfstr.sh
	sh $< >$@

# pattern rules
%.o: %.c
	$(CAMLOPT) -c -o $@ $(CAMLFLAGS) -cc $(CC) -ccopt '$(CFLAGS) $(CPPFLAGS)' $<

%.cmx: %.ml
	$(CAMLOPT) -c -o $@ $(CAMLFLAGS) $<

%.cmi: %.mli
	$(CAMLOPT) -c -o $@ $(CAMLFLAGS) $<

%.1: %.adoc adoc/asciidoc.conf
	a2x -d manpage -f manpage --asciidoc-opts="-f adoc/asciidoc.conf --out-file=$@.xml" $<

# special targets
clean:
	$(RM) llpp link.o help.ml $(OCAML_OBJ) $(OCAML_OBJ:.cmx=.cmi) $(OCAML_OBJ:.cmx=.o) $(MANS) $(MANS:.1=.xml)

dist: clean
	mkdir llpp-$(VERSION)
	cp -r $(DISTFILES) llpp-$(VERSION)
	tar czf llpp-$(VERSION).tar.gz llpp-$(VERSION)
	rm -rf llpp-$(VERSION)

install:
	install -Dm755 llpp "$(DESTDIR)"$(PREFIX)/bin/llpp
	install -Dm644 -t "$(DESTDIR)"$(PREFIX)/share/man/man1  $(MANS)
	install -Dm755 misc/llppac "$(DESTDIR)"$(PREFIX)/bin/llppac
	install -Dm755 misc/llpp.inotify "$(DESTDIR)"$(PREFIX)/bin/llpp.inotify
	install -Dm755 misc/llpphtml "$(DESTDIR)"$(PREFIX)/bin/llpphtml
	install -Dm644 misc/llpp.desktop "$(DESTDIR)"$(PREFIX)/share/applications/llpp.desktop
	install -Dm644 README "$(DESTDIR)"$(PREFIX)/share/licenses/llpp/README

.PHONY: all clean dist install
