kernelsu-objs := ksu.o
kernelsu-objs += allowlist.o
kernelsu-objs += apk_sign.o
kernelsu-objs += sucompat.o
kernelsu-objs += throne_tracker.o
kernelsu-objs += core_hook.o
kernelsu-objs += ksud.o
kernelsu-objs += embed_ksud.o
kernelsu-objs += kernel_compat.o

kernelsu-objs += selinux/selinux.o
kernelsu-objs += selinux/sepolicy.o
kernelsu-objs += selinux/rules.o
ccflags-y += -I$(srctree)/security/selinux -I$(srctree)/security/selinux/include
ccflags-y += -I$(objtree)/security/selinux -include $(srctree)/include/uapi/asm-generic/errno.h

obj-$(CONFIG_KSU) += kernelsu.o

ccflags-y += -DKSU_VERSION=12104

ifeq ($(shell grep -q " current_sid(void)" $(srctree)/security/selinux/include/objsec.h; echo $$?),0)
ccflags-y += -DKSU_COMPAT_HAS_CURRENT_SID
endif

ifeq ($(shell grep -q "struct selinux_state " $(srctree)/security/selinux/include/security.h; echo $$?),0)
ccflags-y += -DKSU_COMPAT_HAS_SELINUX_STATE
endif

ifndef KSU_EXPECTED_SIZE
KSU_EXPECTED_SIZE := 0x033b
endif

ifndef KSU_EXPECTED_HASH
KSU_EXPECTED_HASH := c371061b19d8c7d7d6133c6a9bafe198fa944e50c1b31c9d8daa8d7f1fc2d2d6
endif

ifdef KSU_MANAGER_PACKAGE
ccflags-y += -DKSU_MANAGER_PACKAGE=\"$(KSU_MANAGER_PACKAGE)\"
$(info -- KernelSU Manager package name: $(KSU_MANAGER_PACKAGE))
endif

$(info -- KernelSU Manager signature size: $(KSU_EXPECTED_SIZE))
$(info -- KernelSU Manager signature hash: $(KSU_EXPECTED_HASH))

ccflags-y += -DEXPECTED_SIZE=$(KSU_EXPECTED_SIZE)
ccflags-y += -DEXPECTED_HASH=\"$(KSU_EXPECTED_HASH)\"

# UL, look for iterate_dir on ‎fs/readdir.c
ifeq ($(shell grep -q "^int iterate_dir" $(srctree)/fs/readdir.c 2>/dev/null; echo $$?),0)
$(info -- KernelSU/compat: iterate_dir found!)
ccflags-y += -DKSU_HAS_ITERATE_DIR
endif

# UL, look for read_iter on f_op struct
ifeq ($(shell grep -q "read_iter" $(srctree)/include/linux/fs.h 2>/dev/null; echo $$?),0)
$(info -- KernelSU/compat: f_op->read_iter found!)
ccflags-y += -DKSU_HAS_FOP_READ_ITER
endif

# UL, look for "ext4_unregister_sysfs" on fs/ext4
ifeq ($(shell grep -q "^extern void ext4_unregister_sysfs" $(srctree)/fs/ext4/ext4.h 2>/dev/null; echo $$?),0)
ccflags-y += -DKSU_HAS_MODERN_EXT4
endif

ifeq ($(shell grep -q "int path_umount" $(srctree)/fs/namespace.c; echo $$?),0)
$(info -- KernelSU/compat: path_umount found)
ccflags-y += -DKSU_HAS_PATH_UMOUNT
endif

ifeq ($(shell grep -q "^long copy_from_user_nofault" $(srctree)/include/linux/uaccess.h; echo $$?),0)
$(info -- KernelSU/compat: copy_from_user_nofault found)
ccflags-y += -DKSU_COPY_FROM_USER_NOFAULT
endif

ifeq ($(shell grep -q "^extern long probe_user_read" $(srctree)/include/linux/uaccess.h; echo $$?),0)
$(info -- KernelSU/compat: probe_user_read found)
ccflags-y += -DKSU_PROBE_USER_READ
endif

ifeq ($(shell grep -q "ssize_t kernel_read" $(srctree)/fs/read_write.c; echo $$?),0)
$(info -- KernelSU/compat: newer kernel_read found)
ccflags-y += -DKSU_NEW_KERNEL_READ
endif

ifeq ($(shell grep "ssize_t kernel_write" $(srctree)/fs/read_write.c | grep -q "const void" ; echo $$?),0)
$(info -- KernelSU/compat: newer kernel_write found)
ccflags-y += -DKSU_NEW_KERNEL_WRITE
endif

ccflags-y += -Wno-implicit-function-declaration -Wno-strict-prototypes -Wno-int-conversion -Wno-gcc-compat
ccflags-y += -Wno-declaration-after-statement -Wno-unused-function

## For susfs stuff ##
ifeq ($(shell test -e $(srctree)/fs/susfs.c; echo $$?),0)
$(eval SUSFS_VERSION=$(shell cat $(srctree)/include/linux/susfs.h | grep -E '^#define SUSFS_VERSION' | cut -d' ' -f3 | sed 's/"//g'))
$(info )
$(info -- SUSFS_VERSION: $(SUSFS_VERSION))
else
$(info -- You have not integrate susfs in your kernel.)
$(info -- Read: https://gitlab.com/simonpunk/susfs4ksu)
endif
# Keep a new line here!! Because someone may append config
