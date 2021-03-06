MODNAME:=netmap
SUBSYS:=generic monitor pipe vale
SRCDIR:=/home/wangbojing/share/netmap/LINUX

# list of objects for this module
#
# objects whose source file is in ../sys/dev/netmap
# the source is not here so we need to specify a dependency
$(foreach s,$(SUBSYS),$(eval CONFIG_NETMAP_$(shell echo $s|tr a-z- A-Z_)=y))

remoteobjs-y := netmap_mem2.o netmap_mbq.o

remoteobjs-$(CONFIG_NETMAP_VALE)    += netmap_vale.o netmap_offloadings.o
remoteobjs-$(CONFIG_NETMAP_PIPE)    += netmap_pipe.o
remoteobjs-$(CONFIG_NETMAP_MONITOR) += netmap_monitor.o
remoteobjs-$(CONFIG_NETMAP_GENERIC) += netmap_generic.o
remoteobjs-ptnetmap-$(CONFIG_NETMAP_PTNETMAP_GUEST) = netmap_pt.o
remoteobjs-ptnetmap-$(CONFIG_NETMAP_PTNETMAP_HOST)  = netmap_pt.o
remoteobjs-y += $(remoteobjs-ptnetmap-y)

define remote_template
$$(obj)/$(1): %.o: $$(SRCDIR)/../sys/dev/netmap/$(2) FORCE
	$$(call if_changed_rule,cc_o_c)
endef
$(foreach o,$(remoteobjs-y),$(eval $(call remote_template,$(o),$(o:.o=.c))))
# we compile netmap.c into netmap_common.o to allow
# for MODNAME=netmap
$(eval $(call remote_template,netmap_common.o,netmap.c))

$(obj)/netmap_linux.o: %.o: $(SRCDIR)/netmap_linux.c FORCE
	$(call if_changed_rule,cc_o_c)

# all objects
$(MODNAME)-objs := $(remoteobjs-y) netmap_common.o netmap_linux.o

ifdef CONFIG_NETMAP_PTNETMAP_GUEST
$(obj)/netmap_ptnet.o: %.o: $(SRCDIR)/netmap_ptnet.c FORCE
	$(call if_changed_rule,cc_o_c)

$(MODNAME)-objs += netmap_ptnet.o
endif

obj-$(CONFIG_NETMAP) = $(MODNAME).o

ifdef NETMAP_DRIVER_SUFFIX
  $(foreach v,$(filter %.o,$(O_DRIVERS)),$(eval $(v:.o=$(NETMAP_DRIVER_SUFFIX)-objs) := $v))
  $(foreach v,$(filter %.o,$(O_DRIVERS)),$(info $(v:.o=$(NETMAP_DRIVER_SUFFIX)-objs) := $v))
  obj-m += $(O_DRIVERS:%.o=%$(NETMAP_DRIVER_SUFFIX).o)
else
	obj-m += $(O_DRIVERS)
endif
