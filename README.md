monitoring cs

`monitoring cs` 是一款轻量、原生的 Windows 录屏工具，用于录制指定显示器画面与系统声音，并输出标准 MP4 视频。应用采用蓝色卡片式界面，支持多显示器选择、保存目录修改、30/60/120 FPS、NVENC H.264 硬件编码与软件编码回退，同时提供清晰的录制状态和计时显示。

## 主要功能

- 每次启动自动枚举当前连接的显示器
- 录制显示器原始分辨率画面
- 录制 Windows 系统声音
- 支持 30、60 和 120 FPS
- 优先使用 NVIDIA NVENC H.264
- NVENC 不可用时自动回退至软件 H.264
- 使用 Auto CQP 自动质量模式
- 输出包含 H.264 视频与 AAC 音频的 MP4 文件
- 支持高 DPI 和多种 Windows 显示缩放比例
- 不生成额外的 `config.json` 或日志文件

## 系统要求

- Windows 10 22H2 或更高版本
- 支持 Direct3D 11 的显卡
- 可用的系统音频输出设备
- 保存磁盘至少保留 2 GB 可用空间
- 如需使用 NVENC，需安装支持硬件编码的 NVIDIA 显卡驱动

## 使用方法

1. 运行 `monitoring cs.exe`。
2. 在显示器卡片中选择需要录制的屏幕。
3. 检查保存位置；如需调整，点击“更改”。
4. 点击“开始录制”，然后选择 30、60 或 120 FPS。
5. 录制开始后，状态栏会显示 `Recording` 和录制时间。
6. 点击“停止录制”或按下 F9，等待状态从 `Saving` 切换为 `Finished`。

录制文件默认保存到 Windows“视频”目录，文件名格式如下：

```text
yyyyMMdd_HHmmss.mp4
```

如发生自动分段，后续文件会增加编号：

```text
yyyyMMdd_HHmmss_001.mp4
```

## 快捷键

| 快捷键 | 操作 |
| --- | --- |
| F8 | 开始录制 |
| F9 | 停止录制 |

如果快捷键已被其他程序占用，仍可使用界面中的主按钮完成录制。

## 状态说明

| 状态 | 含义 |
| --- | --- |
| Ready | 已准备，可以开始录制 |
| Preparing | 正在初始化采集与编码组件 |
| Recording | 正在录制 |
| Recovering | 正在恢复异常的采集源 |
| Saving | 正在停止录制并保存文件 |
| Finished | 视频已保存完成 |
| Error | 录制失败，请查看错误提示 |

## 构建

开发环境需要 Visual Studio 2022 Build Tools、MSVC v143、Windows 10/11 SDK 和 CMake 3.25 或更高版本。

```powershell
cmake -S . -B build -A x64 -DBUILD_TESTING=ON
cmake --build build --config Release
ctest --test-dir build -C Release --output-on-failure
cmake --install build --config Release --prefix dist
```

仓库也提供一键发布构建脚本：

```powershell
cmd /c tools\build-release.cmd
```

构建完成后，应用位于：

```text
dist\monitoring cs.exe
```

## 隐私与文件

`monitoring cs` 不需要账号，不连接网络，不创建后台服务，也不会向视频中添加水印。录制结果仅写入用户选择的本地目录。
