# HPM Repo 维护文档

## 一、仓库概述

`hpm-repo` 是 HPM 的包仓库，存放：

- **索引**（`index.json` + `index.json.sig`）
- **官方公钥**（`official.pub`）
- **开发者目录**（每个开发者一个文件夹）
- **工具链描述**（`toolchains/`）

**仓库地址**：https://github.com/xzc842/hpm-repo

**客户端配置**：

```bash
export HPM_REGISTRY="https://raw.githubusercontent.com/xzc842/hpm-repo/main/index.json"
export HPM_OFFICIAL_PUB="https://raw.githubusercontent.com/xzc842/hpm-repo/main/official.pub"
```

## 二、目录结构

```
hpm-repo/
├── README.md                    仓库说明
├── .gitignore                   忽略 *.key 等私钥
├── index.json                   索引
├── index.json.sig               索引签名（官方私钥签）
├── official.pub                 官方公钥
├── official.key                 官方私钥（不要提交）
├── alice/                       开发者 alice
│   ├── alice.pub                开发者公钥
│   ├── alice.pub.sig            公钥签名（官方私钥签）
│   ├── alice.key                开发者私钥（不要提交）
│   ├── ch552g-1.0.0.tar.gz
│   ├── ch552g-1.0.0.tar.gz.sig
│   ├── ch552_usb_cdc-1.0.0.tar.gz
│   └── ch552_usb_cdc-1.0.0.tar.gz.sig
├── bob/                         开发者 bob
│   └── ...
└── toolchains/                  工具链描述
    └── arm-none-eabi-gcc-12.2.0.json
```

## 三、信任链

```
official.pub（根信任）
    ↓ 验证
index.json.sig → index.json
    ↓ 验证
<developer>.pub.sig → <developer>.pub
    ↓ 验证
<package>.tar.gz.sig → <package>.tar.gz
```

## 四、索引格式

`index.json`：

```json
{
  "schema": 1,
  "official_pub": "{baseurl}/official.pub",
  "developers": {
    "alice": {
      "pub": "{baseurl}/alice/alice.pub",
      "pub_sig": "{baseurl}/alice/alice.pub.sig"
    }
  },
  "packages": {
    "ch552g": {
      "versions": {
        "1.0.0": {
          "kind": "chip",
          "developer": "alice",
          "url": "{baseurl}/alice/ch552g-1.0.0.tar.gz",
          "sig_url": "{baseurl}/alice/ch552g-1.0.0.tar.gz.sig",
          "sha256": "57617365374952deb812908de6956d1c36726deaa40f260d85e4d797968c9460",
          "size": 13912,
          "license": "MIT",
          "requires": {}
        }
      }
    }
  }
}
```

### `{baseurl}` 占位符

所有 URL 用 `{baseurl}` 开头，客户端根据索引 URL 自动替换：

- 索引 URL：`https://raw.githubusercontent.com/xzc842/hpm-repo/main/index.json`
- `{baseurl}` → `https://raw.githubusercontent.com/xzc842/hpm-repo/main`

**好处**：镜像改域名时不用改 `index.json`。

### 字段说明

| 字段 | 说明 |
|------|------|
| `schema` | 索引格式版本 |
| `official_pub` | 官方公钥 URL（客户端用硬编码地址，此字段仅参考） |
| `developers` | 开发者公钥 |
| `packages` | 包列表 |
| `kind` | `chip` / `driver` |
| `developer` | 开发者名（对应目录） |
| `url` | tarball 下载地址 |
| `sig_url` | tarball 签名地址 |
| `sha256` | tarball SHA256 |
| `size` | tarball 字节数 |
| `license` | SPDX 标识 |
| `requires` | 依赖简版（key → type） |

## 五、发布新包

### 1. 开发者准备包

驱动包符合 [HPM 驱动包格式规范](https://github.com/xzc842/hpm/blob/main/docs/driver_package.md)：

```
ch552g-1.0.0/
├── chip.json 或 manifest.json
├── LICENSE
├── src/
└── include/
```

**目录名必须是 `<name>-<version>`。**

### 2. 打包 + 签名

```bash
hpm-dev publish <包目录> --key alice/alice.key
```

输出：

```
ok 包: ch552g@1.0.0
ok 打包: publish/ch552g-1.0.0.tar.gz
ok 签名: publish/ch552g-1.0.0.tar.gz.sig

包信息（用于 index.json）：
  name:    ch552g
  version: 1.0.0
  sha256:  57617365374952deb812908de6956d1c36726deaa40f260d85e4d797968c9460
  size:    13912
```


### 3. 更新到仓库


## 六、发布新开发者

### 1. 开发者生成密钥对

```bash
mkdir -p bob
openssl genrsa -out bob/bob.key 4096
openssl rsa -in bob/bob.key -pubout -out bob/bob.pub
```

### 2. 官方签开发者公钥

### 3. 更新到仓库



## 七、撤销包（安全事件）

### 1. 从 `index.json` 删掉

```bash
# 删掉包的版本
```

### 2. 重新签索引

```bash
openssl dgst -sha256 -sign official.key -out index.json.sig index.json
```

### 3. 保留 tarball

**不要删 tarball**，历史用户可能已经下载。**新用户拿不到，老用户还能用。**

### 4. 通知用户

**在 README 或 GitHub Issue 里发公告。**

### 5. 撤销开发者（严重）

**如果开发者私钥泄露**：

1. 从 `index.json` 的 `developers` 里删掉该开发者
2. 删掉该开发者所有包的 `versions`
3. 重新签索引
4. 公告

**开发者重新生成密钥对，官方重新签。**

## 十一、镜像

### 1. 自建镜像

任何人可以自建镜像：

```bash
git clone https://github.com/xzc842/hpm-repo.git mirror
cd mirror
# 启动 HTTP 服务器
python3 -m http.server 8000
```

### 2. 镜像配置

**客户端**：

```bash
export HPM_REGISTRY="https://mirror.example.com/index.json"
export HPM_OFFICIAL_PUB="https://raw.githubusercontent.com/xzc842/hpm-repo/main/official.pub"
```

**`HPM_OFFICIAL_PUB` 仍指向官方**（官方公钥只从官方地址下载，不从镜像）。

**`HPM_REGISTRY` 指向镜像**，`{baseurl}` 自动替换成镜像的 base。

### 3. 镜像同步

```bash
cd mirror
git pull
```

**镜像不需要改 `index.json`**，`{baseurl}` 让客户端自动替换。

## 十二、多镜像 fallback

客户端支持多个镜像：

```bash
export HPM_REGISTRY="https://mirror1.example.com/index.json,https://mirror2.example.com/index.json,https://raw.githubusercontent.com/xzc842/hpm-repo/main/index.json"
```

**第一个失败，自动试第二个。**

**每个镜像的 `index.json` 和签名独立验证**。`HPM_OFFICIAL_PUB` 仍指向官方。

## 十三、安全

### 1. 私钥管理

- **`alice.key`**：开发者自己保存
- **绝不提交到 git**
- **建议离线机器签名**，签完再上传

### 2. 签名流程

```
官方私钥签 index.json
官方私钥签开发者公钥
开发者私钥签自己的包
```

### 3. 指纹验证

**客户端首次使用时提示官方公钥指纹**，用户手动确认。

**指纹变化时再次提示**，警告可能是中间人攻击。

**当前官方公钥指纹**：

```
SHA256: e9:8a:54:b0:9e:39:33:a2:38:e0:00:41:8d:68:5c:5f:87:8f:7d:d4:48:ff:7e:8f:59:79:fa:02:79:40:6f:ee
```

### 4. 撤销

**私钥泄露**：

1. 立即撤销包/开发者
2. 重新生成密钥对
3. 公告


## 十四、版本策略

### semver

- `1.0.0` → `1.0.1`：bug 修复
- `1.0.0` → `1.1.0`：新功能，向后兼容
- `1.0.0` → `2.0.0`：不兼容变更

### 客户端解析

`project.json` 里可以写：

```json
"driver": "rc522@^1.2.0"
```

**`^1.2.0` 匹配 `>=1.2.0 <2.0.0`。**

**`driver.json` 锁定实际版本。**

## 十八、故障排查

### 客户端「索引签名无效」

**原因**：`index.json` 改了但没重新签。

**修复**：

```bash
openssl dgst -sha256 -sign official.key -out index.json.sig index.json
```

### 客户端「SHA256 不匹配」

**原因**：`index.json` 里的 `sha256` 跟实际 tarball 不符。

**修复**：

```bash
sha256sum alice/ch552g-1.0.0.tar.gz
# 更新 index.json 里的 sha256
```

### 客户端「开发者公钥签名无效」

**原因**：开发者公钥被改，或签名不对。

**修复**：

```bash
openssl dgst -sha256 -sign official.key -out alice/alice.pub.sig alice/alice.pub
```

### 客户端「包签名无效」

**原因**：tarball 被改，或签名不对。

**修复**：

```bash
hpm-dev publish <包目录> --key alice/alice.key
```

### 镜像不同步

**原因**：镜像的 `index.json` 过期。

**修复**：

```bash
cd mirror
git pull
```

## 十九、维护清单

### 每次发布包

- [ ] `hpm-dev publish` 打包签名
- [ ] 复制 `.tar.gz` 和 `.sig` 到开发者目录
- [ ] 更新 `index.json`（`url`、`sig_url`、`sha256`、`size`、`requires`）
- [ ] 重新签 `index.json.sig`
- [ ] 验证签名
- [ ] `git commit` + `git push`

### 每次加开发者

- [ ] 开发者生成密钥对
- [ ] 官方签开发者公钥
- [ ] 复制 `bob.pub` 和 `bob.pub.sig` 到 `bob/`
- [ ] 更新 `index.json` 的 `developers`
- [ ] 重新签 `index.json.sig`
- [ ] `git commit` + `git push`

### 每次安全事件

- [ ] 撤销包/开发者
- [ ] 重新签 `index.json.sig`
- [ ] 公告
- [ ] 重新生成密钥对

---

**文档版本**：v1.0
**最后更新**：随仓库维护同步