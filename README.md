# hpm-repo

HPM 官方包仓库。存放芯片驱动、设备驱动和工具链描述文件。
# !!!当前处于测试状态，测试私钥公开
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

## 索引格式

`index.json`：

```json
{
  "schema": 1,
  "official_pub": "https://raw.githubusercontent.com/xzc842/hpm-repo/main/official.pub",
  "developers": {
    "alice": {
      "pub": "https://raw.githubusercontent.com/xzc842/hpm-repo/main/alice/alice.pub",
      "pub_sig": "https://raw.githubusercontent.com/xzc842/hpm-repo/main/alice/alice.pub.sig"
    }
  },
  "packages": {
    "ch552g": {
      "versions": {
        "1.0.0": {
          "kind": "chip",
          "developer": "alice",
          "url": "https://raw.githubusercontent.com/xzc842/hpm-repo/main/alice/ch552g-1.0.0.tar.gz",
          "sig_url": "https://raw.githubusercontent.com/xzc842/hpm-repo/main/alice/ch552g-1.0.0.tar.gz.sig",
          "sha256": "b28131c1de13cd97...",
          "size": 14011,
          "license": "MIT",
          "requires": {}
        }
      }
    },
    "ch552_usb_cdc": {
      "versions": {
        "1.0.0": {
          "kind": "driver",
          "developer": "alice",
          "url": "https://raw.githubusercontent.com/xzc842/hpm-repo/main/alice/ch552_usb_cdc-1.0.0.tar.gz",
          "sig_url": "https://raw.githubusercontent.com/xzc842/hpm-repo/main/alice/ch552_usb_cdc-1.0.0.tar.gz.sig",
          "sha256": "58929a9819c47591...",
          "size": 20458,
          "license": "MIT",
          "requires": {}
        }
      }
    }
  }
}
```

### 字段说明

| 字段 | 说明 |
|------|------|
| `schema` | 索引格式版本 |
| `official_pub` | 官方公钥 URL（仅参考，客户端用硬编码地址） |
| `developers` | 开发者公钥 URL |
| `packages` | 包列表 |
| `kind` | `chip` / `driver` |
| `developer` | 开发者名（对应 `<developer>/` 文件夹） |
| `url` | tarball 下载地址 |
| `sig_url` | tarball 签名地址 |
| `sha256` | tarball 的 SHA256 |
| `size` | tarball 大小（字节） |
| `license` | SPDX 许可证标识 |
| `requires` | 依赖简版（key → type） |

## 包格式

每个包是 `.tar.gz`，解压后是 `<name>-<version>/` 目录，符合 [HPM 驱动包格式规范](https://github.com/xzc842/hpm/blob/main/docs/driver_package.md)。

```
ch552g-1.0.0.tar.gz
└── ch552g-1.0.0/
    ├── chip.json
    ├── LICENSE
    ├── src/
    │   ├── ch552g.c
    │   └── hpm_pin.c
    └── include/
        ├── CH552.H
        ├── ch552g.h
        └── hpm_pin.h
```

## 工具链描述

`toolchains/<toolchain>-<version>.json`：

```json
{
  "name": "arm-none-eabi-gcc",
  "version": "12.2.0",
  "urls": {
    "linux-x64": "https://developer.arm.com/.../gcc-arm-none-eabi-12.2-linux.tar.gz",
    "darwin-x64": "https://developer.arm.com/.../gcc-arm-none-eabi-12.2-macos.tar.gz",
    "win32-x64": "https://developer.arm.com/.../gcc-arm-none-eabi-12.2-win32.zip"
  },
  "sha256": {
    "linux-x64": "abc123...",
    "darwin-x64": "def456...",
    "win32-x64": "ghi789..."
  },
  "extract": "tar.gz"
}
```

## 发布流程

### 1. 开发者准备

1. 生成 RSA-4096 密钥对：
   ```bash
   openssl genrsa -out alice.key 4096
   openssl rsa -in alice.key -pubout -out alice.pub
   ```

2. 把 `alice.pub` 发给官方，请求签名。

3. 官方用 `official.key` 签 `alice.pub`：
   ```bash
   openssl dgst -sha256 -sign official.key -out alice.pub.sig alice.pub
   ```

### 2. 打包

1. 把包放到 `~/.hpm/packages/<name>-<version>/`。

2. 打包：
   ```bash
   tar -czf alice/<name>-<version>.tar.gz -C ~/.hpm/packages <name>-<version>
   ```

3. 签名：
   ```bash
   openssl dgst -sha256 -sign alice.key -out alice/<name>-<version>.tar.gz.sig alice/<name>-<version>.tar.gz
   ```

4. 计算 SHA256：
   ```bash
   sha256sum alice/<name>-<version>.tar.gz
   ```

### 3. 更新索引

1. 在 `index.json` 里加包的 `url`、`sig_url`、`sha256`、`size`、`license`、`requires`。

2. 签索引：
   ```bash
   openssl dgst -sha256 -sign official.key -out index.json.sig index.json
   ```

3. 上传到 GitHub。

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
SHA256: e9:8a:54:b0:9e:39:33:a2:38:e0:00:41:8d:68:5c:5f:87:8f:7d:d4:48:ff:7e:8f:59:79:fa:02:79:40:6f:ee
```

**首次使用 HPM 时，客户端会提示核对这个指纹。请通过官方渠道确认。**
