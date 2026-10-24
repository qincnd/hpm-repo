# hpm-repo

HPM 官方包仓库。存放芯片驱动、设备驱动和工具链描述文件。
# !!!当前处于测试状态
## 仓库结构

```
hpm-repo/
├── index.json              仓库索引（根目录）
├── index.json.sig          索引签名（官方私钥签）
├── official.pub            官方公钥
├── <developer>/            每个开发者一个文件夹
│   ├── <developer>.pub         开发者公钥
│   ├── <developer>.pub.sig     公钥签名（官方私钥签）
│   ├── <package>-<version>.tar.gz
│   └── <package>-<version>.tar.gz.sig
└── toolchains/
    └── <toolchain>-<version>.json   工具链描述
```

## 信任链

```
official.pub（根信任）
    ↓ 验证
index.json.sig → index.json
    ↓ 验证
<developer>.pub.sig → <developer>.pub
    ↓ 验证
<package>.tar.gz.sig → <package>.tar.gz
```

## 镜像

任何人都可以自建镜像。镜像需要同步：

- `index.json` + `index.json.sig`
- 所有 `<developer>/` 目录

## 相关链接

- [HPM 主仓库](https://github.com/xzc842/hpm)
- [驱动包格式规范](https://github.com/xzc842/hpm/blob/main/docs/driver_package.md)
- [官方公钥指纹](#官方公钥指纹)

## 官方公钥指纹
（无）

## 测试公钥指纹
```
SHA256: ec:93:6d:00:bf:7d:3e:f4:f0:20:1b:c9:b5:a6:3a:4a:18:bc:fe:ef:af:8f:87:25:30:5d:44:0c:37:cf:22:a3
```

**首次使用 HPM 时，客户端会提示核对这个指纹。请确认。**
