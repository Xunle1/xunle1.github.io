---
title: Chromium 定制开发
author: 荀乐
date: 2024-04-01 18:29:06
categories: 技术
tags: Chromium
---

本文用于记录在工作中对 Chromium 的定制化开发。基于 84 版本 Chromium 浏览器。

# 功能

## 图标和浏览器安装名称

浏览器图标：

- `chrome/app/theme/chromium/product_logo_*.png`
- `chrome/app/theme/chromium/win/chormium.ico`, `chrome/app/theme/chromium/win/tiles/Logo.png`, `chrome/app/theme/chromium/win/tiles/SmallLogo.png`
- `chrome/app/theme/default_100_percent/chromium/product_logo*.png`, `chrome/app/theme/default_200_percent/chromium/product_logo*.png`

浏览器安装名称：

- `chrome/app/chromium_strings.grd`

## 安装位置

修改浏览器安装位置：

- `chrome/installer/setup/setup_main.cc --> wWinMain --> installer_state.Initialize() --> chrome/installer/setup/installer_state.cc --> InstallerState::Initialize` 修改 `target_path_` 路径

比如：

```cpp
...
// target_path_ = GetChromeInstallPath(system_install());
target_path_ = base::FilePath(L"C:\\");
target_path_ = taget_path_.Append(install_static::GetChromeInstallSubDirectory()); // Append "kCompanyPathName\\"
target_path_ = target_path_.Append(kInstallBinaryDir); // Append "Appliation\\"
...
```

上述代码将安装位置设置为：`C:\\${ComppanyPathName}\\Application\\`

# 附录

参考：
- [图标和浏览器名称](https://stackoverflow.com/questions/64166411/how-to-change-chromium-browser-logo-and-name)