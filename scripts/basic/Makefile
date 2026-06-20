# SPDX-License-Identifier: GPL-2.0-only
#
# fixdep: used to generate dependency information during build process

hostprogs-always-y	+= fixdep

# randstruct: the seed is needed before building the gcc-plugin or
# before running a Clang kernel build.
gen-randstruct-seed	:= $(srctree)/scripts/gen-randstruct-seed.sh
quiet_cmd_create_randstruct_seed = GENSEED $@
cmd_create_randstruct_seed = \
	$(CONFIG_SHELL) $(gen-randstruct-seed) \
		$@ $(objtree)/include/generated/randstruct_hash.h
$(obj)/randstruct.seed: $(gen-randstruct-seed) FORCE
	$(call if_changed,create_randstruct_seed)
always-$(CONFIG_RANDSTRUCT) += randstruct.seed

# integer-wrap: if the .scl file changes, we need to do a full rebuild.
$(obj)/../../include/generated/integer-wrap.h: $(srctree)/scripts/integer-wrap-ignore.scl FORCE
	$(call if_changed,touch)
always-$(CONFIG_UBSAN_INTEGER_WRAP) += ../../include/generated/integer-wrap.h

# rustc-wrapper: rustc's `-Clinker=` expects a single executable path, not a
# command line.  `HOSTCC` may be a multi-word command when wrapped (e.g.
# "ccache gcc"), which would otherwise be split by the shell and mis-parsed by
# rustc.  To work around this, we generate a script that invokes `HOSTRUSTC_LD`
# with the linker arguments appended so such commands can be used safely.
#
# Set `HOSTRUSTC_LD` for a different rustc linker command than `HOSTCC`
HOSTRUSTC_LD ?= $(HOSTCC)

quiet_cmd_rustc-wrapper = GEN     $@
      cmd_rustc-wrapper = \
	      printf '%s\n' '\#!/bin/sh' '$(addsuffix $(space),$(HOSTRUSTC_LD))"$$@"' >$@; \
	      chmod a+x $@

$(obj)/rustc-wrapper: FORCE
	$(call if_changed,rustc-wrapper)

always-$(CONFIG_RUST) += rustc-wrapper
ifneq ($(CONFIG_RUST),)
targets += rustc-wrapper
endif
