# 软件包管理

## RPM

> 安装

```bash
rpm -ivh 包全名
	-i(install) 安装
	-v(verbose) 显示详细信息
	-h(hash) 显示进度
	--nodeps 不检测依赖性
```

> 升级

```bash
rpm -Uvh 包全名
	-U(upgrade) 升级
```

> 卸载

```bash
rpm -e 包名
	-e(erase) 卸载
	--nodeps 不检查依赖性
```



## yum管理

> 查询可用软件包列表

```bash
yum list长度
```

> 安装

```bash
yum -y install 包名
	-y(yes) 自动回答yes
```

