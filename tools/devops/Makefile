TOP=../..
include $(TOP)/Make.config

device-tests-provisioning.csx: device-tests-provisioning.csx.in Makefile $(TOP)/Make.config
	$(Q_GEN) sed \
		-e 's#@XCODE_XIP_NAME@#$(notdir $(XCODE_URL))#g' \
		-e 's#@XI_PACKAGE@#$(XI_PACKAGE)#g' \
		-e 's#@MONO_PACKAGE@#$(MIN_MONO_URL)#g' \
		-e 's#@VS_PACKAGE@#$(MIN_VISUAL_STUDIO_URL)#g' \
		$< > $@;
