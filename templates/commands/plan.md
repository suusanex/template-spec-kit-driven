---
description: Execute the implementation planning workflow using the plan template to generate design artifacts.
scripts:
  sh: scripts/bash/setup-plan.sh --json
  ps: scripts/powershell/setup-plan.ps1 -Json
agent_scripts:
  sh: scripts/bash/update-agent-context.sh __AGENT__
  ps: scripts/powershell/update-agent-context.ps1 -AgentType __AGENT__
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

1. **Setup**: Run `{SCRIPT}` from repo root and parse JSON for FEATURE_SPEC, IMPL_PLAN, SPECS_DIR, BRANCH. For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. **Load context**: Read FEATURE_SPEC and `/memory/constitution.md`. Load IMPL_PLAN template (already copied).

3. **Execute plan workflow**: Follow the structure in IMPL_PLAN template to:
   - Fill Technical Context (mark unknowns as "NEEDS CLARIFICATION")
   - Fill Constitution Check section from constitution
   - Evaluate gates (ERROR if violations unjustified)
   - Phase 0: Generate research.md (resolve all NEEDS CLARIFICATION)
   - Phase 1 generates functional-design.md, integration-test.md, quickstart.md
   - Phase 1: Update agent context by running the agent script
   - Re-evaluate Constitution Check post-design

4. **Stop and report**: Command ends after Phase 2 planning. Report branch, IMPL_PLAN path, and generated artifacts.

## Phases

### Phase 0: Outline & Research

1. **Extract unknowns from Technical Context** above:
   - For each NEEDS CLARIFICATION → research task
   - For each dependency → best practices task
   - For each integration → patterns task

2. **Generate and dispatch research agents**:
   ```
   For each unknown in Technical Context:
     Task: "Research {unknown} for {feature context}"
   For each technology choice:
     Task: "Find best practices for {tech} in {domain}"
   ```

3. **Consolidate findings** in `research.md` using format:
   - Decision: [what was chosen]
   - Rationale: [why chosen]
   - Alternatives considered: [what else evaluated]

**Output**: research.md with all NEEDS CLARIFICATION resolved

### Phase 1: Design

**Prerequisites:** `research.md` complete

### ドキュメントと章の構成

ドキュメントは次のファイルへ分割する。

1. functional-design.md
   - 外部仕様書
2. integration-test.md
   - 統合テスト計画
3. quickstart.md
   - ソフトウェアを使用するユーザー向けの、クイックスタートガイド

#### functional-design.md

次のコメントブロックへ記載する見出しを使用して作成すること。見出し内の本文に書かれているのは、その章の記載内容の説明である。

```markdown
# 外部仕様書

## 概要

## 機能

ソフトウェアが持つ機能の一覧と、それぞれの説明を記載する。

## ユーザーインターフェース

GUIもしくはCLIを含む場合、その定義を記載する。GUIの場合は画面定義・画面の動作など。CLIの場合はコマンドの定義など。

GUI/CLIどちらのケースでも、ユーザーへ表示するメッセージが有る場合は、メッセージの文字列とメッセージを表示する条件を一覧表で記載すること。

## ソフトウェアインターフェース

外部システムとのインターフェースを定義する。APIエンドポイントやデータフォーマットなどを記載する。APIエンドポイントは、WebAPIであればOpenAPI形式、.NETクラスライブラリについてはC#のXMLドキュメントコメント形式で、ネイティブC++についてはVisual C++のXMLドキュメントコメント形式で記載すること。

## 実現方式

ソフトウェアの構成図、アーキテクチャや使用する技術スタックを記載する。また、機能を実現する上で方式の指定や重要なポイントがあれば記載する（Phase 0で判明した内容など）。

## 保守機能

ソフトウェアを保守するための機能について記載する。ログの取得方法、設定変更方法、監視ポイントなど。

## 開発環境

ソフトウェアをビルド・デプロイするために必要な環境を記載する。

```

#### integration-test.md

統合テストの観点を記載する。テストコードで完結するUnitTestの記載は含めず、実際の運用環境で行なうテストの観点を記載する。

### その他の処理


3. **Agent context update**:
   - Run `{AGENT_SCRIPT}`
   - These scripts detect which AI agent is in use
   - Update the appropriate agent-specific context file
   - Add only new technology from current plan
   - Preserve manual additions between markers

**Output**: functional-design.md, integration-test.md, /contracts/*, quickstart.md, agent-specific file

## Key rules

- Use absolute paths
- ERROR on gate failures or unresolved clarifications

