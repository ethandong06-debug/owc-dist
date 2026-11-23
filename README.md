# owc-dist

OWC 控制台的公开**发行通道**。源码仓库保持私有；这里只放已经签名、可直接校验的 App 升级产物。

## App 从哪里取更新

固定主源（始终指向最新正式 Release）：

```
https://github.com/ethandong06-debug/owc-dist/releases/latest/download/app-manifest.json
```

兜底清单（main 分支）：

```
https://raw.githubusercontent.com/ethandong06-debug/owc-dist/main/app-manifest.json
```

App 先校验清单的 Ed25519 签名，再下载 APK 并校验 SHA256 与大小；最后交给 Android 系统安装器，由用户确认安装。

## 每个 Release 的资产

- `OpenWrt-Console-<版本>-arm64.apk` — 已签名的 arm64 安装包
- `app-manifest.json` — 对应版本的签名清单，`.url` 指向本 Release 内的 APK

Release 是版本化、不可变的下载地址；`latest` 只指向最新正式版。旧版本仍可从各自 Release 回滚下载。

## 发布新版本

1. 在私有源码仓库构建并验证 APK。
2. 用仓库外的 Ed25519 私钥生成清单：

   ```bash
   cd router/owc-daemon
   go run ./cmd/owc-app-sign \
     -apk "../../OpenWrt 控制台-A3.5.0-arm64.apk" \
     -version A3.5.0 -version-code 30500 \
     -url "https://github.com/ethandong06-debug/owc-dist/releases/download/A3.5.0/OpenWrt-Console-A3.5.0-arm64.apk" \
     -key ~/.owc-release/ed25519.priv \
     -out /tmp/app-manifest.json
   ```

3. 把 APK 上传到一个**短期可下载地址**，然后提交 `release-request.json` 触发发布工作流：

   ```json
   {
     "version": "A3.5.0",
     "version_code": 30500,
     "tag": "A3.5.0",
     "apk_url": "https://…",
     "asset_name": "OpenWrt-Console-A3.5.0-arm64.apk",
     "sha256": "…",
     "size": 24707192,
     "signature": "…",
     "notes": "…"
   }
   ```

   工作流会校验 APK 的 SHA256/大小，创建 GitHub Release、上传 APK 和签名清单，并把 `app-manifest.json` 写回 main。

4. 删除 `release-request.json` 由工作流自动完成；下一次发布重复第 2–3 步。

## 不要做的事

- 不要把 `aka.doubaocdn.com/s/...` 这类临时分享链接写进长期清单。
- 不要把 APK 反复提交进 Git 历史；Release asset 才是二进制归档位置。
- 不要把 jsDelivr 作为主清单源；它的缓存可能长时间返回旧版本。
- 不要把签名私钥提交到任何仓库。
