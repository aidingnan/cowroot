# cowroot

`cowroot` is a boot/rootfs management solution for embedded Linux systems.

# How it works

`cowroot` differs from most existing solutions in that:

1. A full-fledged Linux distro, such as Debian, could be directly used on an embedded system. The only required modification is the adaption of the kernel and initramfs images to a format required by u-boot.

2. `cowroot` **ALWAYS** boots the system with a ready-only rootfs volume (snapshot). Then an ephemeral rw rootfs, called the workspace, is created from the ro one right after kernel starts, and the system is pivot_root-ed to it. During reboot, the previous workspace is discarded and a new one is created for the new session. This works the same way that a Docker container starts with an immutable image. 

3. `cowroot` builds the **mutability** of rootfs on top of the **immutability**. Mutable rootfs may be convenient and useful for developers, hobbyists, as well as power users. The mutability could be achieved by snapshoting a ro copy from the workspace each time the system shuts down. During next boot, the new snapshot is used as the ro rootfs. There are two issues to be taken into account in an implementation: how to deal with unexpected power loss and how to allow users to switch bewteen immutable mode and mutable mode. But they are details.

4. `cowroot` provides a simple protocol between the boot loader (u-boot) and the Linux system to try and confirm a ro rootfs is bootable. Switching to a new rootfs **ALWAYS** takes the steps required by the protocol. 

# Dependencies and Requirements

`cowroot` depends on btrfs and u-boot supports btrfs (read).

A block-device is required for storing all rootfs snapshots (as the btrfs sub-volume). eMMC is preferred for deployed devices. HDD or SSD may also used if they are available for u-boot.

# Note

Noting that it is the system's responsibility to confirm the ro rootfs is bootable. But this is not a trivial job. Since there may be many possibilities that a rootfs does not function properly. There may be kernel issues, system-level software issues, bad service configurations, and buggy applications. Failing to check those issues and wrongly confirming a bad rootfs to be bootable may cause the user stuck in a system and has no way to switch to another. So making sure the bootability test built into each released rootfs package won't lie.

Of course there could be a fail-safe ro rootfs to do the recovery or safe-mode job. But this is a policy and not implemented now. It is fairly easy to modify the u-boot script to forcfully boot the fail-safe rootfs when, say, some gpio is hold to required level during boot.






