<p align="center">
  <img src="./logo.jpg" width="120" alt="QAMule Logo">
</p>

<h1 align="center">QAMule</h1>

<p align="center">
	Agent-native Android QA solution where the agent is the primary executor.
</p>

<p align="center">
	探索真实应用，保留失败现场，将稳定流程沉淀为 pytest，并持续积累可复用的产品知识。
</p>

---

## 认识 QAMule

QAMule 是一套 **agent-first Android QA 解决方案**，面向希望让 AI 在真实设备上执行测试，而不只是生成脚本的团队。

它从实时探索与验证开始：agent 会检查应用、操作设备、保留有价值的失败现场，并将可复用的知识写回 `knowledge-base/`。

| | 传统测试自动化 | QAMule |
|---|---|---|
| **主要执行者** | 脚本 | AI agent |
| **AI 角色** | 生成或维护脚本 | 直接执行、验证和诊断 |
| **脚本** | 需要预先编写 | 可选，可将稳定流程沉淀为脚本以便更快回放 |
| **失败处理** | 通常只有快照，环境状态会丢失 | 保留失败现场，便于检查和恢复 |
| **新场景** | 先写脚本，再验证 | 直接在真实设备上探索和测试 |
| **知识沉淀** | 分散在脚本和人工记忆中 | 持续写入知识库，便于复用 |

## 快速开始

只需几条命令，你就可以探索页面、测试功能、采集真实设备轨迹，并保留生成的知识以便后续复用。

### 前置条件

- 一台通过 USB 连接的 Android 设备，或一个 Android 模拟器
- [UV](https://docs.astral.sh/uv/getting-started/installation/)：用于管理 Python 环境和依赖
- [ADB](https://developer.android.com/tools/releases/platform-tools)：Android Debug Bridge

### 安装

1. 将 QAMule 作为 agent plugin 安装到你的项目中：

```bash
# GitHub Copilot
copilot plugin marketplace add qamule/qamule
copilot plugin install qamule@qamule

# Claude Code
/plugin marketplace add qamule/qamule
/plugin install qamule@qamule

# VS Code
# command + shift + p -> "Chat: Install plugin from Source" -> "qamule/qamule"
```

2. 初始化项目结构：

```text
/qamule setup <your_project_name>
```

这会为 QAMule 创建基础 UV 项目，并安装所需依赖。

### 使用方式

`@QA` 表示选择 QA agent 来执行命令，`@Distiller` 表示选择 Distiller agent 来执行命令。

`/knowledge-base` 表示使用 `knowledge-base` skill 来读取或记录可复用的测试知识。

#### 旧版 QAMule 项目迁移

如果你已有旧版 QAMule 项目，可以运行下面的命令迁移到新的结构：

```text
/qamule qamule-project-migration
```

#### 写入知识

```text
/knowledge-base Record that this app cannot be launched reliably through adb and must be entered through the in-app UI flow
```

#### 探索页面

```text
@QA Explore the Settings app home page
```

#### 测试功能

```text
@QA Validate that the Bluetooth toggle in Settings can be enabled and disabled correctly
```

#### 以暂停模式运行已有测试并实时报告

```text
@QA Run existing tests on pause mode with real-time report
```

#### 采集训练轨迹

```text
@Distiller Collect one real-device trajectory for opening the Bluetooth settings page in com.android.settings
```

#### 在网页中查看轨迹

```text
@Distiller Open the trajectory in the web page
```

## 许可证

[MIT](LICENSE) (c) 2026 QAMule
