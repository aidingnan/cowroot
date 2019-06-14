# Path

## btrfs volume layout

```
/
    boot/
        system.env
        system.env.tmp
    cowroot/
        ro/
            fe305d84-719a-446a-a2dd-f7aaa16c84aa    #子卷
            ...
        rw/
            ephemeral/  # 子卷 
        tmp/
        tmpvol/ # 子卷
```

说明：

- tmp目录和tmpvol在启动时由hijacker清空重建;
- 使用cowroot-refresh-tmpvol清空和重建tmpvol，该命令返回路径；

## /run

```
/run/
    cowroot/
        root/           # btrfs根卷挂载点
        boot/
            boot.env 
```

