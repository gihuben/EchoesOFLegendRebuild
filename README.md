# EchoesOFLegendRebuild  
由于作者不会写作文+英语不好，以下皆由DEEPSEEK负责介绍：
# 回响物语重构 / Echoes of Legend

> Minecraft 1.20.1 · Forge 47.x · Java 17 · Windows x64

一个把「Java 层字节码守护」与「原生层运行时加固」合并进同一个模组的工程实践。
所有防护都建立在**通用判据**上（类加载器来源、接口实现、字节码行为特征），
不针对任何具体模组、包名或作者。

A modest attempt at combining a **Java bytecode guardian** with **native runtime
hardening** inside a single Forge mod. Every check is based on generic criteria
(loader provenance, interface implementation, bytecode behaviour) — never on
specific mod names, packages or authors.

---

## 功能 / Features

### 游戏内容 / Gameplay

| 名称 | 说明 |
|---|---|
| **ARTEMIS（阿尔忒弥斯）** | 神器。右键/左键攻击、禁生成、结构化移除、改血、循环清洗 |
| **摧毁渲染** | 右键把 `gdi32!SwapBuffers` 切到全吞模式 ⇒ 整屏白屏而逻辑照常，再次右键恢复 |
| **配置物品** | 右键打开原生 GDI+ 配置界面（`ECHO_CONFIG_UI.DLL`） |

### 防御 / Defense

| 项 | 说明 |
|---|---|
| `基础防御` | 伤害不落地 / 死亡与移除被拦截 / 死亡界面不渲染 / 受击渲染压制 |
| `护盾+防火` | 吸收护盾锁 100、火焰免疫 |
| `永不饥饿+清理负面效果` | 按 `MobEffectCategory.HARMFUL` 分类清理 + 饥饿饱和度锁满 |
| `飞行` | 强制飞行资格并保持 |
| `防缴械` | 背包被完全掏空时把主手补为神器 |
| `防御模式` | 潜行+左键开 / 潜行+右键关，接管渲染 |

> `基础防御` 是「伤害与死亡」这组的总闸；其余四项为**常驻增益**，
> 判据是「玩家背包（含副手）此刻持有神器」，不受总闸影响。

### 底层 / Low-level

| 模块 | 说明 |
|---|---|
| `echo_jvmti_lockdown.dll` | JVMTI 函数表 47 槽位 typed proxy 锁定（锁表 / 假成功 / 剔帧 / 能力查询清零） |
| `echo_native_guard.dll` | 库加载拦截、渲染钩子、句柄兜底、隐藏高优先级守护线程 |
| `echo_config_ui.cpp` | 纯 Win32 + GDI+ 原生配置界面 |
| Java Agent | 运行时 bytecode transformer：空壳 / 全返回 / 危险调用中和 / 注入抹除 等 |
| `MinHook` | 内联钩子引擎（BSD，`native/minhook`） |

---

## 环境要求 / Requirements

* **JDK 17**（推荐 17.0.8）
* **Windows x64**
* **Visual Studio 2022 / Build Tools**（含 C++ 工具链）—— 仅原生模块需要
* 网络可访问 `maven.minecraftforge.net`

> 未安装 MSVC 时，`build.gradle` 会跳过三个原生 DLL 的编译并打印警告，
> Java 侧装配失败即 fail-open 降级，**不会**导致启动失败。

---

## 构建 / Building

```bash
git clone <your-repo-url>
cd echoes-of-legend

# 仅 Java 部分
./gradlew build

# 含原生 DLL（需要 MSVC）
./gradlew jar
```

产物：`build/libs/echoes_of_legend-1.0.0.jar`

原生 DLL 会被编译到 `build/native/` 并自动打进 jar 的 `assets/` 目录。
`src/main/resources/assets/*.dll` 不入版本库（见 `.gitignore`），它们每次都由构建生成。

### 在 IDE 中运行 / Run in IDE

```bash
./gradlew genIntellijRuns     # IntelliJ
./gradlew genEclipseRuns      # Eclipse
```

---

## 配置 / Configuration

配置文件：`<游戏目录>/config/echoes.txt`（UTF-8，首次启动自动生成）

* 修改后**重启游戏**生效（配置不热重载）
* 一切开关都在这个文件里，一行一项
* 文件头部保留了项目自身的说明文字

---

## 项目结构 / Layout

```
src/main/java/com/echoxy/
├── agent/      Java Agent：EchoGuardTransformer 等运行时字节码改写
├── attack/     攻击逻辑（禁生成 / 结构化移除 / 改血 / 循环清洗）
├── boot/       启动装配、配置、日志、服务表处理
├── client/     客户端渲染（着色器 / 模型）
├── defense/    防御处理器与客户端判定
├── framecoup/  整帧渲染接管
├── init/       注册（物品 / 创造栏）
├── item/       物品实现
├── mixin/      Mixin 注入点
├── probe/      隐藏类探针
└── service/    服务表相关

native/
├── cpp/        模组自研原生模块
└── minhook/    MinHook（BSD，第三方）
```

---

## 已知限制 / Known limitations

* 原生模块仅支持 **Windows x64**。
* 模组会进行进程内运行时的函数表锁定与内联钩子，**部分杀毒软件可能误报**，
  这是此类工具的正常现象，与恶意行为无关。
* 仅面向单机/自建环境中的实验用途，不建议用于公开服务器。
* `build.gradle` 中的 MSVC 路径探测为通用位置扫描，非本机专属配置。

---

## 许可证 / License

* 本仓库自有代码：**MIT**（见 `LICENSE`）
* `native/minhook`：**BSD 2-Clause**（见 `native/minhook/LICENSE.txt`）
* `LICENSE.txt`：Minecraft Forge MDK 原始许可（LGPL 2.1）

---

## 免责声明 / Disclaimer

本项目用于学习 Minecraft 模组开发中的**类加载机制、JVMTI 接口与原生模块装配**，
以及研究"模组在敌对环境下如何保持自身功能完整"这一工程问题。

请仅在你有权操作的环境中运行。作者不对因使用本模组造成的任何直接或间接损失负责。
