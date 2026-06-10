在原版基础上做了修改以适配ubuntu26.04
## 修改内容总结

### 问题分析

原 deb 包在 Ubuntu 26.04 上不兼容的原因有两类：

|类型|原包依赖|Ubuntu 26.04 对应包|原因|
|---|---|---|---|
|**t64 过渡**|`libasound2`|`libasound2t64`|64-bit time_t 转换|
|**t64 过渡**|`libatk-bridge2.0-0`|`libatk-bridge2.0-0t64`|同上|
|**t64 过渡**|`libatk1.0-0`|`libatk1.0-0t64`|同上|
|**t64 过渡**|`libatspi2.0-0`|`libatspi2.0-0t64`|同上|
|**t64 过渡**|`libcups2`|`libcups2t64`|同上|
|**t64 过渡**|`libglib2.0-0`|`libglib2.0-0t64`|同上|
|**t64 过渡**|`libgtk-3-0`|`libgtk-3-0t64`|同上|
|**t64 过渡**|`libpng16-16`|`libpng16-16t64`|同上|
|**重命名**|`libgdk-pixbuf2.0-0`|`libgdk-pixbuf-2.0-0`|包名格式调整|
|**API 升级**|`libwebkit2gtk-4.0-37`|`libwebkit2gtk-4.1-0`|WebKitGTK API 升级|
|**API 升级**|`libjavascriptcoregtk-4.0-18`|`libjavascriptcoregtk-4.1-0`|WebKitGTK API 升级|

### 具体修改

1. **`DEBIAN/control`** — 使用 `|` 替代依赖语法，优先使用 26.04 新包名，同时兼容旧名：
    
    ```
    libgtk-3-0t64 (>= 3.22.25) | libgtk-3-0 (>= 3.22.25)
    libwebkit2gtk-4.1-0 | libwebkit2gtk-4.0-37 (>= 2.15.1)
    ```
    
2. **`lib-compat/` 目录** — 创建 WebKit 4.0→4.1 的兼容符号链接：
    
    - `libwebkit2gtk-4.0.so.37` → `/usr/lib/x86_64-linux-gnu/libwebkit2gtk-4.1.so.0`
    - `libjavascriptcoregtk-4.0.so.18` → `/usr/lib/x86_64-linux-gnu/libjavascriptcoregtk-4.1.so.0`
3. **`cmclient.sh`** — 修改启动脚本，自动检测 WebKit 版本并加载兼容层：
    
    - 如果系统有 `libwebkit2gtk-4.0.so.37` → 直接使用（旧系统）
    - 如果没有 → 将 `lib-compat` 加入 `LD_LIBRARY_PATH`（Ubuntu 26.04）

### 安装方式

分卷压缩总7个包，解压缩之后运行以下命令：
```bash
sudo dpkg -i cn.coremail.cmclient-ubuntu_4.2.1-1427_amd64.ubuntu2604.deb
sudo apt-get install -f  # 自动补齐缺失依赖
```

> ⚠️ **注意**：WebKit 4.0 和 4.1 的 ABI 并非完全兼容（主要差异在网络层 soup2 vs soup3）。兼容符号链接覆盖了大部分渲染相关符号，基本邮件功能应该正常，但如果遇到 WebKit 相关崩溃，可能需要保留安装 `libwebkit2gtk-4.0-37`（从旧源）。