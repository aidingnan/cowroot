# Environments and Commands

## Environments

u-boot负责维护和持久化两个env变量：

- `loader_l`: sub-volume uuid
- `loader_r`: sub-volume uuid

在启动时u-boot通过kernel bootargs传递这两个变量，同时提供`loader_op`参数供开发人员debug；

系统通过[root volume mount point]/boot/system.env文件持久化四个env变量：
- `system_l`: sub-volume uuid
- `system_l_opts`: ro或rw
- `system_r`: sub-volume uuid
- `system_r_opts`: ro或rw

u-boot仅会提取其中的`system_l`和`system_r`，另外两个变量为系统自己的策略，与boot协议无关；

系统启动后，Hijacker会创建如下目录：

```
/run/
    cowroot/
        root///
        boot/
            boot.env
```

其中root目录是root volume的挂载点，boot目录用于存放与cowroot逻辑有关的一次性使用文件。

hijacker会将下述env保存到boot.env文件中：

- `root_vol`：btrfs根卷的uuid
- `ro_subvol`：此次启动系统时u-boot提供的ro挂载的子卷的uuid
- `_system_l`：启动时的system_l
- `_system_l_opts`：启动时的system_l_opts
- `_system_r`：启动时的system_r
- `_system_r_opts`：启动时的sytem_r_opts
- `loader_l`
- `loader_r`
- `loader_op`

其中`'_'`前缀的变量是系统启动时system.env中的副本，system.env中的变量值可能在系统启动后被修改，修改的原因包括：

- Hijacker负责的recover，`aa|ab -> aa|aa`
- 系统comfirm一次切换成功，`ab|ab -> ab|bb`
- 系统发起了一次切换请求，`aa|aa -> aa|ab, ab|ab -> ab|bb -> ab|bc`

可以通过比对system_xxxx和_system_xxxx变量推断系统中已经做过的操作。

## Commands

### recover

指u-boot尝试启动新rootfs失败后的恢复。

u-boot看到`ab|ab`组合时执行恢复，`ab|ab -> aa|ab`；

hijacker看到`aa|ab`时判定u-boot做了恢复，hijacker负责在保存了boot.env之后，修改system.env内容至`aa|aa`；用户空间程序并不会看到`aa|ab`组合，只会看到`aa|aa`，需要检查_system_#内容确认是否执行了恢复。

/sbin目录中提供`cowroot-recover`命令执行该操作；该命令会在boot.env中增加recover和recover_opts两个环境变量记录被恢复卷的信息（b）。

### checkout

checkout指发出请求切换至另一个rootfs。

可以发生checkout的组合包括：

- `aa|aa` (包括恢复成的)
- `ab|bb`
- `aa|ab`
- `ab|bc`

在后面两个情况里，已经有一次checkout，如果覆盖需要提供额外的操作参数。

上述四种组合可以简化成两种：
- `_a|aa`
- `_a|ab`

在`ab|ab`组合下禁止checkout。

/sbin目录中提供`cowroot-checkout`命令执行该操作；命令参数格式如下：

checkout from `_a|aa`
```
cowroot-checkout -w UUID
```

- `-w`，读写
- `-r`参数忽略；


checkout from `_a|ab`
```
cowroot-checkout -r [-w] [UUID]
```

- `-w`，checkout的rootfs使用读写模式
- UUID如未提供，`-r`会取消请求，即：`_a|ab -> _a|aa`
- UUID如未提供，`-w`视为非法


### confirm

confirm指当前u-boot正在尝试启动一个新的rootfs，用户空间需更新system.env确认该rootfs可用；迁移过程为`ab|ab -> ab|bb`，该过程可继续checkout至新的切换rootfs请求，即`ab|bb -> ab|bc`，其中c可以是a。

/sbin目录中提供`cowroot-confirm`命令执行该操作；

### user checkout

user checkout的意思是，一个checkout/confirm的周期均从外部发起，例如http api service，此时要求cowroot提供统一的接口，即同样的命令调用两次，一次是在旧的rootfs中，另一次是在新的rootfs中，保证两次调用的代码路径一致是为了保证升级后的新系统是可以继续切换的，如果使用其他方式进行confirm，可能造成用户被trap在一个功能不正常的rootfs中。

/sbin目录中提供`cowroot-user-checkout`命令实现该操作；

该命令分别调用cowroot-checkout命令和cowroot-confirm命令；

### snapshot

`cowroot-snapshot`提供该命令，目前仅支持从tmpvol snapshot到ro目录下，返回uuid；

### (auto) commit

(auto) commit是保存workspace；在cowroot设计下，一个可以持续保存修改的workspace，是通过snapshot和checkout实现的；而不是直接重用workspace。

(auto) commit只是（应该）发生在关机时的checkout，如果系统已经出现了手工checkout，env状态呈现为：

- `aa|ab`
- `ab|bc`

应理解为用户希望切换到指定的rootfs，而不是当前workspace的snapshot，这时有两种设计策略：只snapshot，不checkout，或者既不snapshot，也不checkout；设计上采用了后者。

如果关机时看到了ab|ab组合，这意味着该rootfs未能confirm，此时不应该自动commit。

在关机时可以执行自动commit的条件是：

```
       rw
[ab]a|aa
```
如果未能正常关机，下一次系统启动后，hijacker将只会看到aa|aa组合（system左右同值等于重置loader的左右值，loader_op记为sync）；此时如果system右值选项为rw，可确认为异常关机，hijacker将执行自动commit动作后重启。

无论在关机时还是hijacker运行时，(auto) commit都可以使用以下路径找到workspace：

```
/run/cowroot/root/cowroot/rw/ephemeral
```





