# scrcpy4

[English](README.md) · 简体中文

scrcpy4 是面向 Android 4.4（API 19）及以上设备的 scrcpy 兼容分支，可通过 USB 或 TCP/IP 在电脑上显示并操作 Android 设备，无需 root。

本仓库以 [zxystd/scrcpy 的 `api19` 分支](https://github.com/zxystd/scrcpy/tree/api19) 为基础，补充了 Android 4.4 连接和输入兼容处理，以及 macOS 触控板滚动修复。它不是 Genymobile 官方发布的独立版本。

## 功能与兼容处理

- 支持 Android 4.4 / API 19 及以上版本。
- Android 4.4 上直接使用 `adb forward` 建立连接，并在启动服务端时设置 `ANDROID_DATA` 和 Dalvik 缓存目录。
- 鼠标点击以触摸事件注入，避免 Android 4.4 对鼠标按钮状态的兼容问题。
- 保留 SDL 触控板滚轮的浮点增量，降低小幅滚动被整数截断的影响。
- 滚轮事件固定注入到设备画面中央，减少页面滚动时命中目标变化造成的中断；当前滚动比例为 `0.35`，可在 `server/src/main/java/com/genymobile/scrcpy/EventController.java` 的 `SCROLL_SPEED` 调整。
- 处理部分 KitKat 视频解码器输出尺寸与屏幕尺寸有轻微裁切差异的情况。
- 兼容较新的 FFmpeg API，可在当前 macOS Homebrew 工具链下构建。

触控板滚动比例是滚轮输入增量的缩放值，与视频帧率或投屏画面尺寸无关。

## 环境要求

- Android 4.4/API 19 或更新版本，并在设备上启用 USB 调试。
- Android SDK Platform Tools（提供 `adb`）。
- macOS 构建需要 SDL2、FFmpeg、Meson、Ninja、pkg-config、JDK 和 Android SDK。

其他操作系统的依赖说明请参阅英文版 [README.md](README.md)。

## macOS 构建

安装客户端依赖：

```bash
brew install sdl2 ffmpeg pkg-config meson ninja
```

安装 JDK 17 和 Android SDK，然后设置环境变量。以下 SDK 路径是 Android Studio 的默认路径：

```bash
export JAVA_HOME="$(/usr/libexec/java_home -v 17)"
export ANDROID_HOME="$HOME/Library/Android/sdk"
export PATH="$JAVA_HOME/bin:$ANDROID_HOME/platform-tools:$PATH"
```

克隆并构建：

```bash
git clone https://github.com/ghfork/scrcpy4.git
cd scrcpy4
meson setup build --buildtype release
meson compile -C build
```

构建完成后，直接指定设备序列号启动：

```bash
adb devices
./build/app/scrcpy -s <设备序列号>
```

如果连接了多台设备，必须使用 `-s` 选择目标设备。

## macOS 本地安装

可将程序安装到独立目录，并创建 `scrcpy4` 命令：

```bash
meson setup build --buildtype release --prefix="$HOME/.local/opt/scrcpy4"
meson compile -C build
meson install -C build
mkdir -p "$HOME/.local/bin"
ln -sf "$HOME/.local/opt/scrcpy4/bin/scrcpy" "$HOME/.local/bin/scrcpy4"
```

确保 `$HOME/.local/bin` 在 `PATH` 中，然后运行：

```bash
scrcpy4 -s <设备序列号>
```

## 常用操作

- `Control+B`：返回。
- `Control+H`：返回主屏幕。
- `Control+S`：打开最近任务。
- `Control+F`：切换全屏。
- 触控板滚动：在屏幕中央注入 Android 滚轮事件。

## 已验证环境

本次开发在 macOS arm64 和 OPPO R7s（Android 4.4.4/API 19）上验证了投屏、键盘操作、点击和触控板滚动。其他设备型号、Android 版本及应用界面仍可能存在差异。
