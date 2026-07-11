monitoring cs

面向 Windows 10 与游戏录制的原生便携录屏工具。功能边界只有显示器画面和系统声音录制；不包含直播、场景、滤镜、截图、账号、联网、更新、日志或后台服务。

系统要求

- Windows 10 22H2（Build 19045）或更高版本
- 支持 Direct3D 11 的显示适配器
- NVIDIA GPU 可优先使用系统 Media Foundation 暴露的 NVENC H.264；初始化失败会重试一次，再回退软件 H.264
- 程序目录及视频保存目录必须可写

开发构建需要 Visual Studio 2022、Desktop development with C++、MSVC v143、Windows 10/11 SDK、CMake 3.25+。发布程序使用静态 MSVC Runtime，不要求用户安装开发环境。

使用方法

1. 将 `Recorder.exe` 和 `config.json` 放在任意可写目录。
2. 双击运行，选择本次要录制的显示器和视频保存目录。
3. 选择 30、60 或 120 FPS。
4. 点击“开始录制”或按 F8；点击“停止录制”或按 F9 完成保存。
5. 录制状态下可点击“新建分段”。没有暂停功能。

若 F8/F9 已被其他程序注册，界面会明确提示，可继续使用主按钮。程序不写注册表、不创建服务、不自动启动、不联网。

媒体实现

- Windows Graphics Capture 优先，失败时回退 Desktop Duplication
- D3D11 Video Processor 在 GPU 上将 BGRA 转成 NV12
- 12 槽固定 Texture Pool；Media Foundation sample 持有槽位令牌，编码器释放样本后纹理才可复用
- Scheduler 按用户选择的目标 FPS 输出；WGC 静态画面时重复最近有效 GPU 纹理
- WASAPI Loopback，48 kHz、Stereo、16-bit PCM；AAC 目标约 192 kbps
- QPC 单调时钟；音频按真实 WASAPI 帧数推进，超过 5 ms 相位误差时每包最多校正一个 PCM frame
- H.264 CQP/Quality、低延迟、实时、无 B 帧、单 Pass；GTX 1080 Ti 使用偏速度策略
- fragmented MP4：`ftyp/moov` 在前，随后持续写入 `moof/mdat`，约每两秒刷新 byte stream
- 30 GiB、2 小时或手动命令触发分段；文件名为 `yyyyMMdd_HHmmss[_001].mp4`
- 启动时只检查符合本程序命名规则的 MP4；安全时截断未完成的尾部 fragment，无法确认时保留原文件

本机真实验证结果

构建环境：Windows 10 Build 19045、MSVC 19.44、Windows SDK 10.0.26100、x64 Release。

- 2560×1440、60 FPS、H.264 + AAC 真实录制成功
- Windows 属性处理器识别：H.264、AAC 192 kbps、2560×1440、60.00 FPS
- 10 秒正常停止录制生成约 18–30 MB MP4，Finalize 后结构完整
- 手动分段产生两个独立的 fragmented MP4；两段均包含 `ftyp, moov, moof, mdat`，正常停止包含 `mfra`
- 强制终止 8 秒录制后得到约 19.6 MB 文件；检测到 52 个完整顶层 box，最后一个 `mdat` 边界完整
- 2 小时 +100 ppm 音频时钟偏差模拟与 30 GiB/2 小时阈值测试通过
- Release 构建和 CTest `media_invariants` 通过

当前构建的 SHA-256：

`18F2FA89CCEB8AC6441B848A4B1577FD2DC82AC9C956027B9F50656A71DC5C27`

构建

```powershell
cmake -S . -B build -A x64 -DBUILD_TESTING=ON
cmake --build build --config Release
ctest --test-dir build -C Release --output-on-failure
cmake --install build --config Release --prefix dist
=
