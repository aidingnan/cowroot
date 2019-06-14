# Environment Variables

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





