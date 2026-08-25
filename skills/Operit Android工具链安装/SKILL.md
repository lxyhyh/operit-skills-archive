---
name: Operit Android工具链安装
description: 在 Operit 的 Ubuntu（ARM64/aarch64）环境中从零安装或重建完整 Android 开发工具链，包括换源、JDK、Gradle、Android SDK、NDK、ARM64 原生化适配、构建缓存与固化。适用于用户说"安装Android开发环境/JDK/Gradle/SDK/NDK/装工具链/环境重建/ARM64构建报错/exec format error"等场景。
---

# Android工具链安装

在 Operit 平台的 Ubuntu（ARM64）沙箱中，从零安装完整可用的 Android 开发工具链，并完成 ARM64 原生化适配与配置固化。本技能沉淀了已验证的完整流程，可在新环境或环境重置后一键复现。

## 适用场景

- Operit Ubuntu 沙箱为空环境，需要搭建 Android 开发工具链（JDK + Gradle + SDK + NDK）
- 环境被重置/损坏后需要重建
- ARM64 环境下构建报 `exec format error` / `cannot execute: required file not found`（x86-64 二进制问题）
- 需要配置构建加速（Build Cache / ccache / 镜像）

## 执行步骤

### 阶段 1：环境评估与软件源

1. 确认系统：`cat /etc/os-release`、`uname -m`（必须 aarch64）、`df -h /`
2. 实测国内镜像速度（下载 `ubuntu-ports/dists/noble/main/binary-arm64/Packages.xz` 计时）：中科大/清华/阿里云，选最快
3. 写入 `/etc/apt/sources.list`（**ARM64 必须用 `ubuntu-ports`**；若系统启用了 amd64 多架构，ports 行加 `[arch=arm64]`，并补 `[arch=amd64]` 的 ubuntu 主源，否则 apt update 404；**平台模板常缺 noble-security 行，务必补上，否则安全更新不可见**）：
   ```
   deb [arch=arm64] https://mirrors.ustc.edu.cn/ubuntu-ports/ noble main restricted universe multiverse
   deb [arch=arm64] https://mirrors.ustc.edu.cn/ubuntu-ports/ noble-updates main restricted universe multiverse
   deb [arch=arm64] https://mirrors.ustc.edu.cn/ubuntu-ports/ noble-backports main restricted universe multiverse
   deb [arch=arm64] https://mirrors.ustc.edu.cn/ubuntu-ports/ noble-security main restricted universe multiverse
   ```
4. `apt update` 后 `apt upgrade -y`（含安全补丁）

### 阶段 2：安装 JDK 与 Gradle

1. JDK：`apt install -y openjdk-25-jdk`（最新 LTS），`update-alternatives --set java/javac` 指向 25
2. Gradle：从 `https://services.gradle.org/versions/current` 查最新版，下载 `gradle-<ver>-bin.zip` 解压到 `/opt/gradle-<ver>`；**卸载 apt 旧版 gradle**（4.x），避免 PATH 抢占

### 阶段 3：安装 Android SDK

1. 下载 cmdline-tools：`https://dl.google.com/android/repository/commandlinetools-linux-<ver>_latest.zip`，解压到 `/opt/android-sdk/cmdline-tools/latest`
2. `sdkmanager --sdk_root=/opt/android-sdk "platform-tools" "platforms;android-36" "build-tools;37.0.0" "ndk;28.2.13676358"`（版本按需）
3. `adb` 用 apt 装 ARM64 原生版：`apt install -y adb fastboot`，软链到 SDK 的 platform-tools

### 阶段 4：ARM64 原生化（关键，勿跳过）

官方 SDK/NDK 只提供 x86-64 二进制，ARM64 上必须替换：

1. **aapt2/aidl/zipalign/split-select**：从 `https://github.com/Commit451/android-arm-build-tools` Releases 下载与 build-tools 版本对应的 ARM64 版（如 platform-tools-37.0.0），替换到 `build-tools/<ver>/`（原版备份到 `x86_64-backup/`）
2. **AGP 9.x 的 aapt2 override**：官方 Maven 只有 x86-64 构件（**且必须 ≥2.20 版本**：2.19 的社区静态版无法加载 android-36 的 android.jar）。把 ARM64 版 aapt2 放到 `/opt/aapt2/bin/aapt2`，在**全局** `~/.gradle/gradle.properties` 写 `android.aapt2FromMavenOverride=/opt/aapt2/bin/aapt2`（项目级配置只对单项目生效）
3. **NDK 原生工具链**：官方 NDK 只有 linux-x86_64 host。方案：系统 clang-18 + lld-18（apt 装 `lld-18`）+ NDK 自带 sysroot 组装，wrapper 脚本形如：
   ```bash
   exec /usr/bin/clang-18 --target=aarch64-linux-android<api> --sysroot=$NDK/.../sysroot -rtlib=compiler-rt --gcc-toolchain=$SYSROOT/usr -stdlib=libc++ ...
   ```
   注意：a) 需要把 NDK 的 `libclang_rt.builtins-aarch64-android.a` 复制到系统 clang 的 `lib/linux/`，否则链接报 `-lgcc` 找不到；b) 工具链必须部署到 `prebuilt/linux-x86_64/bin`（构建系统硬编码此路径），原 x86_64 备份为 `bin.x86_64-orig`；c) 为多 ABI 生成 armv7a/x86_64/i686/riscv64 wrapper；d) **勿遗漏 strip/调试工具**（llvm-strip、llvm-objcopy、llvm-symbolizer、llvm-objdump、llvm-readelf、llvm-dwarfdump 也要建 ARM64 wrapper：exec /usr/bin/llvm-<名>-18 "$@"；linux-x86_64/bin 与 linux-aarch64/bin 双份都要放，AGP hostTag 固定拼 linux-x86_64，部分探测按 JVM os.arch；漏掉 llvm-strip 的后果见踩坑 #9）

### 阶段 5：Gradle 全局加速与镜像

写入 `~/.gradle/gradle.properties`（对所有项目生效）：
```
android.aapt2FromMavenOverride=/opt/aapt2/bin/aapt2
org.gradle.caching=true
org.gradle.configuration-cache=true
org.gradle.parallel=true
org.gradle.workers.max=6
org.gradle.daemon=true
org.gradle.daemon.idletimeout=7200000
```
写入 `~/.gradle/init.gradle`（阿里云镜像 + **ARM64 strip 任务修复**，Gradle 9.x 写法：settingsEvaluated 内注入 pluginManagement/dependencyResolutionManagement 仓库；不要用 allprojects 方式，会与 settings 仓库模式冲突）：
   ```groovy
   // 镜像：settingsEvaluated 内 maven 仓库（见踩坑 #5），写法用 name = 'x'; url = 'y' 赋值语法（见踩坑 #10）
   // 必加：ARM64 下 AGP 会禁用 strip 任务导致 .so 无法打包进 APK（见踩坑 #9）
   gradle.projectsEvaluated { g ->
       g.rootProject.allprojects { p ->
           p.tasks.configureEach { t ->
               if (t.name.contains('strip')) { t.enabled = true }
           }
       }
   }
   ```

### 阶段 6：ccache（NDK C/C++ 加速）

1. `apt install -y ccache`，配置 `~/.ccache/ccache.conf`（max_size=10G, compression=true）
2. 全局环境变量 `export NDK_CCACHE=/usr/bin/ccache`（写入 /etc/profile.d/）

### 阶段 7：环境固化

1. `/etc/profile.d/android-env.sh`：JAVA_HOME/ANDROID_HOME/ANDROID_SDK_ROOT/GRADLE_HOME/PATH/NDK_CCACHE
2. 运维脚本放 `/opt/scripts/`，软链到 `/usr/local/bin/`（PATH 直接调用）：
   - `verify-env.sh`：10+ 项环境体检
   - `restore-arm64-tools.sh`：从备份恢复 build-tools 35/36/37 的 ARM64 工具
   - `switch-apt-mirror.sh ustc|tuna|aliyun`：一键切源（应对平台守护重置）
   - `backup-config.sh`：全量配置备份到 /sdcard
3. `.bashrc` 加会话自检（自动跑 verify-env）

### 阶段 8：Node 生态与 MCP 依赖核验

`node`/`npm`/`pnpm`（MCP npx 类依赖 pnpm dlx 链路）/`python3`/`uv`/`uvx`；确认 `~/mcp_plugins/` 下已部署插件的运行时依赖可用

### 阶段 9：验证

1. 跑 `verify-env.sh` 全绿
2. 干净项目试构建：AGP 9.3.1 + compileSdk 36，`gradle :app:assembleDebug` 应 BUILD SUCCESSFUL
3. NDK 试编译：`aarch64-linux-android35-clang -shared -o lib.so test.c` 产物应为 ARM aarch64 ELF
4. （项目含 native 时）验证 .so 打包：`unzip -l <apk> | grep lib/` 应出现 `lib/<abi>/xxx.so`，`aapt2 dump badging <apk>` 应含 `native-code:`；没有则查踩坑 #9
5. 缓存验证：clean 后二次构建应显著加速（FROM-CACHE）

## 约束边界

- 本技能面向 **ARM64/aarch64 Ubuntu**；amd64 环境不需要阶段 4
- 版本需匹配：aapt2 社区版要对应 build-tools 版本；AGP 9.x 必须配 aapt2 2.20+
- `sources.list` 可能被 Operit 平台守护重置为默认源，重建后用 `switch-apt-mirror.sh` 切回
- 不覆盖/不冲突已有「Android开发环境搭建」技能（本技能为从零全量安装，彼为检查补齐）
- 系统包安装操作前需用户确认；重命名/删除已有文件前先备份

## 期望输出

- 完整工具链清单（JDK/Gradle/AGP/SDK platforms/build-tools/NDK/aapt2 版本）
- `verify-env.sh` 全绿报告
- 干净项目构建成功 + APK 产物
- 固化脚本与配置落盘清单（路径、作用）
- 若中途失败：保留证据（日志、file 架构输出、错误信息）并给出修复建议

## 踩坑记录与经验（本会话实测，执行时避坑）

1. **AGP 对自定义 aapt2 有硬性要求**：`android.aapt2FromMavenOverride` 指向的文件必须是 **ELF 可执行文件**（shell 脚本会被拒），且**文件名必须以 `aapt2` 结尾**（AGP 源码 Aapt2FromMaven.kt 用 `endsWith("aapt2")` 校验，叫 `aapt2-launcher` 之类的名字会被拒绝）。自定义路径建议直接放 `/opt/aapt2/bin/aapt2`。
2. **aapt2 社区版版本必须 ≥2.20**：实测 2.19 版（如 Sou6900 静态版）链接 android-36 时报 `failed to load include path .../android-36/android.jar`，无法处理新资源格式；Commit451 的 2.20 版正常。
3. **GitHub 访问坑**：直连大文件（如 NDK 1.4GB tar）会超时/被墙（HTTP 000）；GitHub API 限流 60 次/小时，拿不到 releases 列表。对策：大文件用 `ghproxy.net`/`ghfast.top` 等代理且先分片测速；release 资产列表用网页 `releases/expanded_assets/<tag>` 抓取（不受 API 限流影响）。
4. **NDK 不要尝试用 qemu 包装 x86-64 工具链**：clang driver 通过 `/proc/self/exe` 定位自身并直接 exec 子进程（cc1/链接器），wrapper 三层链会断裂，且 lld/cc1 还会递归 exec 更多工具。正确方案：系统 clang/lld + NDK sysroot 组装原生工具链（见阶段 4）。
5. **init.gradle 写法坑（Gradle 9.x）**：a) 不要用 `def repo = { maven { ... } }` 闭包变量复用，会报 `Could not find method maven()`（委托上下文问题），应直接内联写仓库；b) 不要用 `allprojects { repositories { ... } }`，会与 settings 的 `repositoriesMode` 冲突（报 "prefer settings repositories"），正确做法是 `settingsEvaluated` 内注入。
6. **全局 vs 项目级配置**：`android.aapt2FromMavenOverride` 写在项目 `gradle.properties` 只对该项目生效；必须写 `~/.gradle/gradle.properties` 才对所有项目/新窗口生效。
7. **build-tools 替换后验证**：替换 ARM64 工具后，用 `file <工具> | grep aarch64` + `<工具> version` + `aapt2 dump badging <apk>` 三重验证，避免"替换了但没用上"。
8. **多架构 404 排查**：apt update 报 `binary-amd64/Packages 404` = 当前源不含该架构，检查 `[arch=...]` 限定与多架构源是否齐全（见阶段 1）。
9. **AGP（8/9 通用）在 ARM64 环境禁用 strip 任务 → native .so 无法打包进 APK**（本环境实测，社区未见先例）：现象：构建成功但 APK 无 `lib/`（jniLibs 或 AAR 依赖的 .so 全部丢失）；日志 `> Task :app:stripDebugDebugSymbols SKIPPED` + `task onlyIf 'Task is enabled' is false`。定位：任务在配置阶段末期被 AGP 置 `enabled=false`（`afterEvaluate` 时仍为 true，任务图就绪时已 false）；`packageDebug` 的 native 输入是 `stripped_native_libs/<variant>/stripDebugDebugSymbols/out`，strip 不执行则该目录为空 → APK 无 .so。已验证与以下**无关**：构建/配置缓存、daemon、llvm-strip 文件是否存在、ndkVersion 是否声明、Manifest 的 extractNativeLibs、AGP 版本（8.13 与 9.3.1 均复现）。**修复**（已全局固化在 `~/.gradle/init.gradle`）：
   ```groovy
   gradle.projectsEvaluated { g ->
       g.rootProject.allprojects { p ->
           p.tasks.configureEach { t ->
               if (t.name.contains('strip')) { t.enabled = true }
           }
       }
   }
   ```
   前提：NDK bin 里 `llvm-strip` 等必须是可执行的 ARM64 wrapper（见阶段 4d），否则强制启用后运行时报缺工具。
10. **init.gradle / Groovy DSL 空格赋值语法弃用**：`maven { name 'x'; url 'y' }` 在 Gradle 8/9 报 deprecation（Gradle 10 移除构建失败），应写 `maven { name = 'x'; url = 'y' }`。自查：`gradle <任务> --warning-mode all | grep -i deprecat` 应为 0。


## 参考

- 本环境已验证版本快照：`references/env-snapshot.md`
- 环境体检脚本：`scripts/verify-android-env.sh`（与 verify-env.sh 同逻辑）
