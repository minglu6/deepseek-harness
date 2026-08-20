# Agent Note: 未知适配器家族在模型页仍提供 API 密钥

Status: implemented

[English](2026-08-20-unknown-adapter-models-key.md) | 中文

## 问题

模型编辑器按名字只为 `llm-deepseek` 与 `llm-pi-ai` 提供精选卡片。第三方适配器通过 `registerConfigurableProviders` 注册后会出现在「添加提供方」中——按目录顺序往往排在第一——但选中后只显示 `settings.yaml` 提示，Apply 禁用，没有密钥框。插件无法注入该卡片；模型页是和其他提供方风格一致的唯一配置界面。

## 决策

未知 settings namespace 仍渲染共用的 **API 密钥** 字段。Apply 通过 `credentials.set` 按 profile 引用（profile 没有引用时则为 `<ROUTE>_API_KEY`）写入所填密钥；schema 没有 `apiKeyEnv` 可记录时，物化一个空 profile。其余连接事实仍留在 `settings.yaml`。添加提供方列表按 provider id 排序，避免单独注册的插件被钉在顶部。

`llm-qoder` 是第三套精选家族：PAT、可选 VPC 实例，以及与 DeepSeek 相同的模型目录编辑器。不展示网关或 OpenAPI 地址。其他未知适配器仍只有密钥字段。

## 考虑过的方案

**在编辑器里为每个第三方 namespace 特判。** 否决：每个新插件都要改 harness UI。

**把密钥放在「设置 → 插件」。** 否决：那是插件调参面，不是提供方凭据面，和模型页风格不一致。

**未知家族继续只显示提示。** 否决：从 git 安装的插件将无法在产品 UI 里配置。

**已配置行也排序。** 推迟：已安装行保持目录顺序；只有添加列表会把新插件顶到最前。

## 后果

凡声明了 settings path 的适配器都可以在模型页填密钥。额外字段（VPC、自定义端点）仍走 yaml。没有 `apiKeyEnv` 的 schema 仍会物化用户 profile，使该行变为已配置且可删除。

## 测试

`packages/client/ui-settings-models/tests/components.client.spec.tsx` 覆盖添加列表顺序、未知家族的密钥字段，以及写入 `PLAIN_API_KEY` 并物化 profile。
