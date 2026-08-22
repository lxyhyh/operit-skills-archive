# Operit Skills Archive

这是从本地 Operit 环境归档的 Skill 文档仓库。仓库设置为 **Private**，用于个人备份与版本追踪。

## Archive Scope

- 归档范围：`/sdcard/Download/Operit/skills/` 下的 57 个顶层 Skill 目录中的 `SKILL.md`。
- 存放位置：`skills/<Skill 名称>.md`；文件名保留本地 Skill 原名称。
- 排除内容：`skill-creator/examples/` 等示例子目录，以及 MCP 配置、沙盒包、角色卡、模型配置、Token 和其他平台内部资产。
- 合规检查：已检查 frontmatter、重复名称和常见凭据模式；未发现疑似真实 API Key、Token、Cookie 或私钥。文档中的公开参考链接和示例路径按原文保留。
- 生成时间：2026-08-22。

## Sources & Attribution（来源与归属）

归档中的技能分为两类来源：**改编自第三方** 与 **本地自创**。归属按文档内容特征归类；如与实际情况有出入，以修正后的记录为准。

### 改编自第三方（主要来自 mattpocock/skills）

以下技能改编自知名 Skills 作者 [mattpocock](https://github.com/mattpocock) 的 [skills](https://github.com/mattpocock/skills) 仓库，经中文翻译与 Operit 平台本地化适配，核心方法论与结构保留原作者风格。**版权归原作者所有。**

共 19 个：

- bug-fixer
- TDD开发
- 功能实现
- 代码审查
- 调试诊断
- 合并冲突
- 对话交接
- 对话转PRD
- 计划转Issue
- 领域建模
- 快速原型
- 教学
- 技能初始化
- 技能编写
- 技能路由
- 需求追问文档
- 知识洁癖
- 架构改进
- Shoehorn迁移

### 本地自创

以下技能为 Operit 本地环境自研（原创），内容与结构由本人设计，不涉及第三方版权。

共 38 个：

- API设计与契约治理
- APK编译问题诊断
- Android发布与应用商店交付
- Android开发环境搭建
- Android角色协作开发
- Bug修复
- CI-CD与发布管理
- Git防护
- Issue分类
- LSPosed-Mod-Dev
- Miuix风格设计规范
- Operit Android工具链安装
- SandboxPackage_DEV
- operit-vibe-coding
- release-readiness-check
- skill-creator
- 产品分析与实验治理
- 代码设计
- 依赖升级与技术债治理
- 备份恢复与灾难恢复
- 复用优先
- 安全审查与威胁建模
- 性能分析与优化
- 技术文档与知识治理
- 提交钩子
- 数据库设计与数据迁移
- 文章编辑
- 无障碍与国际化
- 模块化分析
- 测试策略与质量保障
- 知识库管理
- 线上可观测性与事故响应
- 练习搭建
- 自动编译
- 资源占用优化
- 身份认证与权限设计
- 追问引擎
- 隐私合规与数据治理

## Caution

该仓库包含 Operit 内部工作规范和本地路径信息，默认不得改为公开仓库。若需要公开发布，应先重新进行版权、隐私、路径和安全审查；其中改编自第三方的技能还需确认其原始许可证条款。

## Contents

每个 Markdown 文件对应一个顶层 Skill；`skill-creator/examples/weekly-report-generator/SKILL.md` 是示例文档，不作为独立 Skill 归档。
