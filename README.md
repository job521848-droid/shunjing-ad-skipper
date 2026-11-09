# 瞬净 ShunJing

> 一款基于 Android 无障碍服务的本地广告跳过工具：识别开屏、插屏、悬浮、纯图片与叠加式下载广告的「跳过 / 关闭」，同时对 **扫码、支付、密码、验证码、授权登录** 页面保持硬锁定。

![version](https://img.shields.io/badge/version-1.6.8-blue)
![minSdk](https://img.shields.io/badge/minSdk-24-green)
![targetSdk](https://img.shields.io/badge/targetSdk-35-green)
![language](https://img.shields.io/badge/Java-17-orange)
![tests](https://img.shields.io/badge/unit%20tests-223%20passed-brightgreen)

---

## 一、界面预览

| 主界面（1.5.12） | 启动动效（1.6.1） |
|---|---|
| ![界面预览](docs/screenshots/界面预览.png) | ![启动动效](docs/screenshots/启动动效.png) |

首次安装会在开场动效结束后播八步聚光灯导览：其余压暗，逐个高亮首页控件；
洞会弹簧滑位，边框有流光绕行、四角挂 HUD 角标，换步炸开冲击波与光尘，开场扫一遍整屏。
![聚光灯导览](docs/screenshots/聚光灯导览.png)

## 二、它解决什么问题

绝大多数广告跳过工具靠「关键词 + 固定坐标」，遇到自绘弹窗、WebView 模板、纯图片关闭键就会失效，或者反过来误点支付按钮。

瞬净的判断链条是**证据优先**：先判断「这一页是不是广告、是第几层广告」，再决定「哪个控件是关闭键」，最后**点完必须验证页面是否真的变了**。任何证据不足的情况一律不动手。

## 三、核心能力

| 能力 | 说明 |
|---|---|
| 开屏 / 插屏跳过 | 事件驱动扫描，窗口变化即评估，不等固定轮询 |
| 倒计时跳过 | 识别「5s 跳过」类控件，倒计时结束前不抢点 |
| 叠层弹窗定位 | 居中卡片 + 右上角邻域小图标 + 无文字无 id，四条同时成立 |
| 纯图片关闭键 | 右 / 左上角、右下角几何定位，按屏宽比例限定尺寸 |
| WebView 模板广告 | 靠「反馈 ⟷ 关闭」同排对偶关系定位，整树 `clickable=false` 也能点 |
| 摇一摇广告 | 专用页面特征识别 + 8 秒跳转拦截 |
| 叠加式下载广告 | 监管合规条 + 视频素材 + 下载诱导文案 + Dialog 浮层，加权 ≥5 才成立 |
| 点击后校验 | 点击未生效则本地自适应补点，仍失败才升级 |
| 学习模式 | 用户手点一次，记录控件特征与坐标，下次自动命中 |
| 云端视觉复核（可选） | 本地证据不足时才调用，支持 OpenAI 兼容接口，Key 由用户自填 |
| 临时暂停 | 暂停到指定时间点，到点自动恢复，关总开关自动清除 |
| 商店跳转回退 | 广告跳进应用商店后自动退回原页面（**默认关闭**） |

## 四、三档模式

| 模式 | 适合谁 | 行为 |
|---|---|---|
| **安静** | 怕误点、只要最稳的 | 只处理明说「广告 / 跳过」的页面，运营弹窗不碰，激励视频一律保护 |
| **均衡**（默认） | 大多数人 | 广告与运营弹窗都关，激励视频播完再关，奖励不丢 |
| **极速** | 追求开屏绝对无感 | 见到关闭控件就点，激励视频直接放弃奖励 |

三档只改**行动门槛**，安全硬锁在任何模式下都不变。手动改动任一开关后，模式自动落回「自定义」。

## 五、安全边界（不可关闭）

- **支付 / 扫码保护**：二维码、付款码、收付款、密码、验证码、指纹人脸、授权登录页面，任何规则与 AI 证据都不触发点击，也不会把页面内容上传；
- **系统危险页**：安装器、权限授予、卸载确认、系统设置关键页；
- **受保护包**：桌面、系统 UI、瞬净自身不参与判断；
- **隐私**：`allowBackup=false`，不申请存储 / 悬浮窗 / 无障碍以外的权限，云端视觉仅在用户显式开启并配置后才可能联网。

当前权限清单仅 3 项：`POST_NOTIFICATIONS`（前台状态通知）、`VIBRATE`（换挡震动）、`INTERNET`（仅在云端视觉开启时使用）。

## 六、目录结构

```
ShunJing/
├── app/
│   ├── build.gradle                 # 应用模块：minSdk 24 / targetSdk 35 / versionCode 30
│   ├── proguard-rules.pro
│   └── src/
│       ├── main/
│       │   ├── AndroidManifest.xml
│       │   ├── java/app/shunjing/skipper/
│       │   │   ├── ShunJingApp.java
│       │   │   ├── CrashLog.java
│       │   │   ├── data/            # 偏好、暂停、预设、日志、留存策略
│       │   │   ├── engine/          # 决策引擎、证据模型、各类关闭键定位器
│       │   │   ├── service/         # 无障碍服务、通知、抓取入口
│       │   │   └── ui/              # 首页、设置、日志、诊断、启动动效
│       │   └── res/                 # 布局用代码构建，此处为图标 / 动画 / 样式
│       └── test/java/...            # 215 项 JVM 单元测试
├── gradle/wrapper/                  # Gradle Wrapper（版本锁定）
├── docs/                            # 使用说明、更新记录、动效预览
├── build.gradle                     # AGP 9.0.0
├── settings.gradle
├── gradle.properties
├── gradlew / gradlew.bat
├── .gitignore
└── .gitattributes
```

## 七、构建

环境要求：JDK 17、Android SDK（compileSdk 35）、Gradle 由 Wrapper 自动拉取。

```bash
# 调试包
./gradlew :app:assembleDebug

# 单元测试（215 项，纯 JVM，无需设备）
./gradlew :app:testDebugUnitTest

# 发布包（默认输出未签名 APK）
./gradlew :app:assembleRelease
```

仓库**不含签名文件**。发布包需要在本地配置签名后自行签出，例如：

```bash
apksigner sign --ks your-release.jks --ks-key-alias youralias \
  --out ShunJing-release.apk app/build/outputs/apk/release/app-release-unsigned.apk
```

## 八、测试

`app/src/test` 下为纯 JVM 测试，覆盖决策引擎、敏感页硬锁、商店回退、返回键识别、下载广告定位等。代表性用例直接用真机抓取的控件树复刻，例如 `DownloadAdCloseTest` 还原了一份 32 节点的叠加下载广告页面，验证「点的是右下角 68×68 图标，而不是底部写着广告的角标」。

```bash
./gradlew :app:testDebugUnitTest --tests "app.shunjing.skipper.engine.*"
```

## 九、版本记录

| 版本 | 要点 |
|---|---|
| 1.6.8 | 导览动效重做：弹簧滑位、边框流光、HUD 角标、冲击波与光尘、开场扫描、翻页音效与触感 |
| 1.6.7 | 新手引导改为聚光灯导览：真实首页上逐个高亮控件、自动滚动跟随、隐藏步骤自动跳过 |
| 1.6.6 | 内置六页新手引导、设置页可重看、正文自适应滚动 |
| 1.6.5 | 叠加式下载广告识别、右下角关闭键定位、新手指引误判修复 |
| 1.6.4 | 应用内诊断报告可读性、四角候选判定 |
| 1.6.3 | 返回键误判为关闭键的修复 |
| 1.6.1 | 启动动效重做：2.34s 时间轴、分层退场、首页推近衔接 |
| 1.5.x | 临时暂停、商店跳转回退、WebView 模板、摇一摇、三档配色 |
| 1.4.x | 三档场景模式、叠层弹窗、云端视觉服务商预设 |
| 1.3.x | 倒计时跳过、点击后校验、云端视觉复核 |

详细更新说明见 [`docs/`](docs/)；新手先读 [`docs/简易使用指南.md`](docs/简易使用指南.md)。

## 十、免责声明

本项目仅供学习 Android 无障碍服务、控件树分析与启发式决策使用。请遵守目标应用的服务条款与当地法律法规，不要用于绕过付费内容或破坏他人服务。使用本工具产生的任何后果由使用者自行承担。

## 十一、License

未指定开源许可证。如需公开分发，请先补充 `LICENSE`（如 MIT / Apache-2.0）。
