---
tags: [ai, speech, asr, tts, spacemit]
domain: ai
aliases: [Speech-SDK, Speech SDK, sm-sdk, 语音SDK]
status: approved
created: 2026-11-16
updated: 2026-11-16
source: C:\Users\luojunlin\docs-ai\en\application_tools\speechsdk.md
superseded_by:
---

# Speech SDK

## 定义

**Speech Model SDK**（`sm-sdk`）是 SpacemiT 的综合语音 AI 服务平台，把多种语音处理技术整合在统一方案中。

> ⚠️ 官方说明：当前版本为临时版本，后续将并入 AI SDK。

## 包含功能

| 功能 | 缩写 | 说明 |
|---|---|---|
| 声活检测 | VAD | 检测音频中是否有人声 |
| 自动语音识别 | ASR | 语音转文字 |
| 机器翻译 | MT | 实时翻译 |
| 文字转语音 | TTS | 文字合成语音 |
| 关键词唤醒 | KWS | 检测特定唤醒词 |
| 说话人分离 | Diarization | 区分多个说话人 |

## 平台支持

| 平台 / 系统 | 支持 |
|---|---|
| K3 Bianbu LXQT/GNOME | ✅ |
| K1 Buildroot | ❌ |
| K1 OpenHarmony | ❌ |
| K1 Bianbu LXQT/GNOME | ❌ |
| K3 Buildroot | ❌ |
| K3 OpenHarmony | ❌ |

> 目前仅支持 K3 Bianbu 桌面版。

## 相关词条

- [[../ai/AI-Architecture|AI 架构]] — 整个 AI 软件栈的分层
- [[../ai/AI-SDK|AI SDK]] — 通用 AI 开发套件（Speech SDK 未来将并入）
- [[../hardware/products/K3]] — 目前支持的芯片

## 来源文档

- `docs-ai/en/application_tools/speechsdk.md`
