# MinGW Makefile for binkd (GNU make)
#
# Run it from the binkd source root:
#
#   mingw32-make -f mkfls/nt95-mingw/Makefile [options]
#
# Options: [DEBUG=1] [DEBUGCHILD=1] [BINKD9X=1] [IPV6=1] [FTS5004=1]
#          [BW_LIM=1] [AF_FORCE=1] [NTLM=0] [PERL=1] [PERLDL=1]
#          [ZLIB=1] [BZLIB2=1] [ZLIBDL=1]
#
# On Windows the commands are run by cmd.exe, MSYS is not required.
# On other systems (cross-compiling) POSIX shell commands are used, e.g.:
#
#   make -f mkfls/nt95-mingw/Makefile CC=i686-w64-mingw32-gcc \
#        WINDRES=i686-w64-mingw32-windres
#
# Targets: all (default), clean, distclean

#PERL_BASE=c:/Perl
#PERL_LIB=perl58

#ZLIB_BASE=../zlib
#ZLIB_LIB=z
#BZLIB2_BASE=../bzlib2
#BZLIB2_LIB=bz2

# =============================================================================

ifeq ($(OS),Windows_NT)
  SHELL := cmd.exe
  winpath = $(subst /,\,$(1))
  MKDIR   = if not exist "$(call winpath,$(1))" mkdir "$(call winpath,$(1))"
  RMDIR   = if exist "$(call winpath,$(1))" rmdir /s /q "$(call winpath,$(1))"
  Q       = "
else
  MKDIR   = mkdir -p $(1)
  RMDIR   = rm -rf $(1)
  Q       = '
endif

CC      = gcc
WINDRES = windres

NTLM ?= 1

OUTDIR    = bin/mingw32
BINKDTYPE = mingw32

CFLAGS  += -mthreads -pipe -funsigned-char -Wall -Wno-char-subscripts -Wno-comment

CDEFS += -DWIN32 -DHAVE_THREADS -DHAVE_INTTYPES_H -DHAVE_INTMAX_T -DHAVE_IO_H \
         -DHAVE_DOS_H -DHAVE_STDARG_H -DHAVE_SNPRINTF -DHAVE_VSNPRINTF       \
         -DHTTPS -DAMIGADOS_4D_OUTBOUND -DHASATTRIBUTE -DNONAMELESSUNION

SRCS = binkd.c readcfg.c tools.c ftnaddr.c ftnq.c client.c server.c protocol.c \
       bsy.c inbound.c branch.c ftndom.c ftnnode.c srif.c pmatch.c readflo.c   \
       prothlp.c iptools.c rfc2553.c run.c binlog.c exitproc.c getw.c xalloc.c \
       setpttl.c https.c md5b.c crypt.c getopt.c snprintf.c                    \
       nt/breaksig.c nt/getfree.c nt/sem.c nt/TCPErr.c nt/WSock.c             \
       nt/w32tools.c nt/tray.c

RES = nt/binkdres.rc

# NTLM proxy authorization +++++++++++++++++++++++++++++++++++++++++++++++++++
ifeq ($(NTLM),1)
CDEFS += -DNTLM
SRCS  += ntlm/des_enc.c ntlm/helpers.c ntlm/ecb_enc.c ntlm/md4_dgst.c ntlm/set_key.c
else
OPTDIR += -nontlm
endif

# binkd9x / binkd (NT service) +++++++++++++++++++++++++++++++++++++++++++++++
ifdef BINKD9X
SRCS      += nt/win9x.c
CDEFS     += -DBINKD9X
LDFLAGS   += -mwindows
LIBS      += -lwsock32
BINKDNAME  = binkd9x-mingw
OUTDIR    := $(OUTDIR)-binkd9x
BINKDTYPE := $(BINKDTYPE), binkd9x
else
SRCS      += nt/service.c
ifdef IPV6
CDEFS     += -DIPV6
LIBS      += -lws2_32
BINKDNAME  = binkd-mingw-ipv6
OUTDIR    := $(OUTDIR)-binkd-ipv6
else
LIBS      += -lwsock32
BINKDNAME  = binkd-mingw
OUTDIR    := $(OUTDIR)-binkd
endif
ifdef FTS5004
CDEFS     += -DWITH_FTS5004
SRCS      += srv_gai.c
LIBS      += -ldnsapi
OPTDIR    += -fts5004
endif
endif

# debug ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
ifdef DEBUG
CFLAGS    += -g
CDEFS     += -DDEBUG -D_DEBUG
BINKDNAME := $(BINKDNAME)-debug
OUTDIR    := $(OUTDIR)-debug
BINKDTYPE := $(BINKDTYPE), debug
else
CFLAGS    += -O2
LDFLAGS   += -s
endif

ifdef DEBUGCHILD
CDEFS     += -DDEBUGCHILD
BINKDNAME := $(BINKDNAME)-debugchild
OUTDIR    := $(OUTDIR)-debugchild
BINKDTYPE := $(BINKDTYPE), debugchild
endif

# bandwidth limiting, soft AF force ++++++++++++++++++++++++++++++++++++++++++
ifdef BW_LIM
CDEFS  += -DBW_LIM=1
OPTDIR += -bwlim
endif

ifdef AF_FORCE
CDEFS  += -DAF_FORCE=1
OPTDIR += -afforce
endif

# perl support +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
# PERLDL implies PERL
ifdef PERLDL
PERL = 1
endif

ifdef PERL
PERL_BASE ?= $(shell perl -MConfig -e $(Q)print $$Config{prefix}$(Q))
PERL_INCL  = $(PERL_BASE)/lib/CORE

CDEFS  += -DWITH_PERL -DNDEBUG -DNO_STRICT -DHAVE_DES_FCRYPT \
          -DPERL_IMPLICIT_CONTEXT -DPERL_IMPLICIT_SYS -DPERL_MSVCRT_READFIX
SRCS   += perlhooks.c
C_DIRS += -idirafter "$(PERL_INCL)"

ifdef PERLDL
CDEFS     += -DPERLDL
CFLAGS    += -fno-strict-aliasing
BINKDNAME := $(BINKDNAME)-perldl
OUTDIR    := $(OUTDIR)-perldl
BINKDTYPE := $(BINKDTYPE), perldl
else
# libperlNNN.a or perlNNN.lib -> perlNNN
PERL_LIB  ?= $(shell perl -MConfig -e $(Q)($$_ = $$Config{libperl}) =~ s/^lib//; s/\.(a|lib)$$//; print$(Q))
L_DIRS    += -L"$(PERL_INCL)"
LIBS      += -l$(PERL_LIB)
BINKDNAME := $(BINKDNAME)-perl
OUTDIR    := $(OUTDIR)-perl
BINKDTYPE := $(BINKDTYPE), perl
endif
endif

# zlib, bzlib2 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
ifdef ZLIB
CDEFS += -DWITH_ZLIB
ifdef ZLIB_BASE
C_DIRS += -I"$(ZLIB_BASE)/include"
L_DIRS += -L"$(ZLIB_BASE)/lib"
endif
ifndef ZLIBDL
ZLIB_LIB  ?= z
LIBS      += -l$(ZLIB_LIB)
BINKDNAME := $(BINKDNAME)-zlib
OUTDIR    := $(OUTDIR)-zlib
BINKDTYPE := $(BINKDTYPE), zlib
else
BINKDNAME := $(BINKDNAME)-zlibdl
OUTDIR    := $(OUTDIR)-zlibdl
BINKDTYPE := $(BINKDTYPE), zlibdl
endif
endif

ifdef BZLIB2
CDEFS += -DWITH_BZLIB2
ifdef BZLIB2_BASE
C_DIRS += -I"$(BZLIB2_BASE)/include"
L_DIRS += -L"$(BZLIB2_BASE)/lib"
endif
ifndef ZLIBDL
BZLIB2_LIB ?= bz2
LIBS       += -l$(BZLIB2_LIB)
BINKDNAME  := $(BINKDNAME)-bzlib2
OUTDIR     := $(OUTDIR)-bzlib2
BINKDTYPE  := $(BINKDTYPE), bzlib2
else
BINKDNAME  := $(BINKDNAME)-bzlib2dl
OUTDIR     := $(OUTDIR)-bzlib2dl
BINKDTYPE  := $(BINKDTYPE), bzlib2dl
endif
endif

ifneq ($(ZLIB)$(BZLIB2),)
SRCS += compress.c
ifdef ZLIBDL
CDEFS += -DZLIBDL
SRCS  += zlibdl.c
endif
endif

# =============================================================================

# Options which don't change the exe name still need their own output dir
OUTDIR  := $(OUTDIR)$(subst $(eval) ,,$(OPTDIR))

BINKDEXE = $(BINKDNAME).exe
OBJDIR   = $(OUTDIR)/obj
OBJS     = $(addprefix $(OBJDIR)/,$(SRCS:.c=.o))
RESOBJS  = $(addprefix $(OBJDIR)/,$(RES:.rc=.o))
OBJDIRS  = $(OBJDIR) $(OBJDIR)/nt $(OBJDIR)/ntlm

.PHONY: all printinfo clean distclean

all: printinfo $(OUTDIR)/$(BINKDEXE)

printinfo:
	@echo -----------------------------------------------------------
	@echo binkd type : $(BINKDTYPE)
	@echo output dir : $(OUTDIR)
	@echo binkd exe  : $(BINKDEXE)
	@echo -----------------------------------------------------------

$(OBJDIRS):
	@$(call MKDIR,$@)

$(OBJS) $(RESOBJS): | $(OBJDIRS)

$(OUTDIR)/$(BINKDEXE): $(OBJS) $(RESOBJS)
	@echo Linking $(BINKDEXE)...
	@$(CC) $(CFLAGS) $(LDFLAGS) $(L_DIRS) -o $@ $(OBJS) $(RESOBJS) $(LIBS)

$(OBJDIR)/%.o: %.c
	@echo Compiling $<...
	@$(CC) -c -MMD $(CFLAGS) $(CDEFS) $(C_DIRS) -o $@ $<

$(OBJDIR)/%.o: %.rc
	@echo Compiling $<...
	@$(WINDRES) $(CDEFS) -D"BINKDNAME=$(BINKDNAME)" -D"BINKDEXE=$(BINKDEXE)" $< $@

clean:
	@echo Cleanup...
	@$(call RMDIR,$(OBJDIR))

distclean:
	@echo Full cleanup...
	@$(call RMDIR,$(OUTDIR))

-include $(OBJS:.o=.d)
