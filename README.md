# cowroot

`cowroot` is a boot/rootfs management solution for embedded Linux systems.

A full-fledged Linux distro, such as Debian, could be directly used on an embedded system.  the generation of the uncompressed kernel and init ramdisk for u-boot.

# How it works

1. `cowroot` **ALWAYS** boots the system from a ready-only rootfs (a btrfs snapshot). 

A read-write copy is created and mounted as the real rootfs right after the kernel starts. This copy is an ephemeral workspace. It is discarded during next boot. This works much like a Docker container and we may say that the rootfs is **immutable**.

2. Switching to a new rootfs **ALWAYS** takes a two-step procedure. The boot loader tries to boot new rootfs and checks the confirmation during next boot. If the Linux system confirms that new rootfs is **bootable**, the boot loader accepts the switching request, otherwise, the request is rejected and the boot loader will boot original rootfs.

Noting that it is the Linux system's responsibility to confirm new rootfs is bootable, and this is a non-trivial job. DON'T simply put a systemd service confirming it when system starts. Even if system starts successfully, there may be issues, such as network interface driver doesn't work properly, leaving user no chance to switch to another rootfs any more.

As long as the new Linux system does not lie, the boot loader guarantees a usable system.

3. Mutable rootfs may be convenient and useful for developers, hobbyists, as well as power users. `cowroot` builds the **mutability** on top of the **immutability**. The basic idea is to snapshot a ro copy from the rw workspace and then switch to it in next boot, though there are a few detailed issue in implemntation.

`cowroot` could be best understood as a mechanism, not a policy. Most of the real world business, such as the system upgrade/downgrade, factory reset, and recovery mode, are not defined in this layer.  


# Dependencies and Requirements

`cowroot` depends on btrfs and u-boot supports btrfs (read).

A block-device is required for storing all rootfs snapshots (as the btrfs sub-volume). eMMC is preferred for deployed devices. HDD or SSD may also used if they are available for u-boot.





Of course there could be a fail-safe ro rootfs to do the recovery or safe-mode job. But this is a policy and not implemented now. It is fairly easy to modify the u-boot script to forcfully boot the fail-safe rootfs when, say, some gpio is hold to required level during boot.






