TOP=../..
include $(TOP)/Make.config

export DEVELOPER_DIR=$(XCODE_DEVELOPER_ROOT)

#
# mmp
#

MMP_CONF=Release
MMP_DIR=bin/$(MMP_CONF)
LOCAL_MMP=$(MMP_DIR)/mmp.exe
LOCAL_MMP_COMMAND=$(SYSTEM_MONO) --debug $(LOCAL_MMP)

# mmp.csproj.inc contains the mmp_dependencies variable used to determine if mmp needs to be rebuilt or not.
-include mmp.csproj.inc

$(MMP_DIR)/mmp.exe: $(mmp_dependencies)
	$(Q_GEN) $(SYSTEM_MSBUILD) $(TOP)/Xamarin.Mac.sln "/t:mmp" $(XBUILD_VERBOSITY) /p:Configuration=$(MMP_CONF)

MMP_TARGETS = \
	$(MAC_DESTDIR)$(MAC_FRAMEWORK_CURRENT_DIR)/bin/mmp \
	$(MAC_DESTDIR)$(MAC_FRAMEWORK_CURRENT_DIR)/lib/mmp/mmp.exe \
	$(MAC_DESTDIR)$(MAC_FRAMEWORK_CURRENT_DIR)/lib/mmp/Xamarin.Mac.registrar.mobile.a \
	$(MAC_DESTDIR)$(MAC_FRAMEWORK_CURRENT_DIR)/lib/mmp/Xamarin.Mac.registrar.full.a \

MMP_DIRECTORIES = \
	$(MAC_DESTDIR)$(MAC_FRAMEWORK_CURRENT_DIR)/bin \
	$(MAC_DESTDIR)$(MAC_FRAMEWORK_CURRENT_DIR)/lib/mmp \

$(MAC_DESTDIR)$(MAC_FRAMEWORK_CURRENT_DIR)/bin/mmp: mmp | $(MAC_DESTDIR)$(MAC_FRAMEWORK_CURRENT_DIR)/bin
	$(Q) $(CP) $< $@

$(MAC_DESTDIR)$(MAC_FRAMEWORK_CURRENT_DIR)/lib/mmp/mmp.exe: $(MMP_DIR)/mmp.exe | $(MAC_DESTDIR)$(MAC_FRAMEWORK_CURRENT_DIR)/lib/mmp
	$(Q) $(CP) $(dir $<)/*.exe $(dir $@)
	$(Q) $(CP) $(dir $<)/*.dll $(dir $@)
	$(Q) $(CP) $(dir $<)/*.pdb $(dir $@)

$(MAC_DESTDIR)$(MAC_FRAMEWORK_CURRENT_DIR)/lib/mmp/Xamarin.Mac.registrar.%.a: Xamarin.Mac.registrar.%.a | $(MAC_DESTDIR)$(MAC_FRAMEWORK_CURRENT_DIR)/lib/mmp
	$(Q) $(CP) $<* $(dir $@)

$(MMP_DIRECTORIES):
	$(Q) mkdir -p $@

#
# Partial static registrar libraries
#

GENERATE_PART_REGISTRAR = $(Q_GEN) $(LOCAL_MMP_COMMAND) --xamarin-framework-directory=$(MAC_DESTDIR)$(MAC_FRAMEWORK_CURRENT_DIR) -q --runregistrar:$(abspath $@) --sdkroot $(XCODE_DEVELOPER_ROOT) --sdk $(OSX_SDK_VERSION) $(abspath $<) --registrar:static --arch=x86_64 --nolink

Xamarin.Mac.registrar.mobile.x86_64.m: $(TOP)/src/build/mac/mobile-64/Xamarin.Mac.dll $(LOCAL_MMP)
	$(GENERATE_PART_REGISTRAR) --target-framework Xamarin.Mac,Version=v2.0,Profile=Mobile -a:$(MOBILE_BCL_DIR)/mscorlib.dll
	$(Q) touch Xamarin.Mac.registrar.mobile.x86_64.m Xamarin.Mac.registrar.mobile.x86_64.h

Xamarin.Mac.registrar.full.x86_64.m:   $(TOP)/src/build/mac/full-64/Xamarin.Mac.dll $(LOCAL_MMP)
	$(GENERATE_PART_REGISTRAR) --target-framework Xamarin.Mac,Version=v4.5,Profile=Full -a:$(FULL_BCL_DIR)/mscorlib.dll
	$(Q) touch Xamarin.Mac.registrar.full.x86_64.m Xamarin.Mac.registrar.full.x86_64.h

Xamarin.Mac.registrar.%.x86_64.a: Xamarin.Mac.registrar.%.x86_64.m
	$(Q_CC) $(MAC_CC) -DDEBUG -g -gdwarf-2 -x objective-c++ -std=c++14 -o $@ -c -arch x86_64 $< -Wall -Wno-unguarded-availability-new -I$(MAC_DESTDIR)$(XAMARIN_MACOS_SDK)/include -mmacosx-version-min=$(MIN_OSX_SDK_VERSION) -fobjc-runtime=macosx

Xamarin.Mac.registrar.%.a: Xamarin.Mac.registrar.%.x86_64.a
	$(Q) $(CP) $< $@

ifdef INCLUDE_MAC
install-local:: $(MMP_TARGETS)
all-local:: $(MMP_TARGETS)
endif

clean-local::
	rm -f mmp.stub.c mmp.helper.o
	rm -rf bin obj
	rm -f Xamarin.Mac.registrar.*
	$(SYSTEM_MSBUILD) "/t:Clean" /p:Configuration=$(MMP_CONF) *.csproj

include $(TOP)/mk/rules.mk
include ../common/Make.common

# make will automatically consider files created in chained implicit rules as temporary files, and delete them afterwards
# marking those files as .SECONDARY will prevent that deletion.
.SECONDARY: $(foreach ext,h m a,Xamarin.Mac.registrar.mobile.x86_64.$(ext) Xamarin.Mac.registrar.full.x86_64.$(ext))
.SECONDARY: Xamarin.Mac.registrar.mobile.a Xamarin.Mac.registrar.full.a
