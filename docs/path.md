# Path

## btrfs卷

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
- 应用程序可使用cowroot-empty-tmpvol清空和重建tmpvol，该命令返回路径；

## `/run`目录

```
/run/
    cowroot/
        root/           # btrfs根卷挂载点
        boot/
            boot.env 
```

Hijacker在启动的最初阶段就创建该文件夹，包括boot.env文件；

在Hijacker和system上下文下该路径均可用；

Hijacker在启动最初完成recover工作，recover操作之前需要boot.env，recover本身会向boot.env增加内容。

