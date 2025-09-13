# Changelog

This file records all important changes for JobHunt AI.

## [0.1.6]

### Changed
- すべての.mdファイルの一貫性を確認し、更新しました

### Added
- database-schema.mdにHangfireスキーマテーブルを追加しました
- database-schema.mdにSemantic Kernelエージェント状態管理テーブルを追加しました
- specification.mdでAI駆動レジリエンスの実装詳細を明確化しました

### Fixed
- すべてのドキュメント間で技術スタックの一貫性を確保しました

### Prompts Used
<prompt>
Can you double check all the .md files and go through all of them to make sure that you are not missing anything.
</prompt>

## [0.1.5]

### Changed
- StagehandからPlaywright.NETへの移行を完了しました
- ブラウザ自動化をPlaywright.NET + Semantic Kernelアーキテクチャに更新しました

### Added
- Playwright.NETの統合詳細をarchitecture.mdに追加しました
- AI駆動の自己修復セレクター機能を文書化しました
- ブラウザ自動化のC#ネイティブ実装パターンを追加しました

### Fixed
- TypeScript依存性を排除し、純粋な.NETアーキテクチャを確保しました

### Prompts Used
<prompt>
Similarly, let's consider about stagehand. Is that good? Or is there good alternative?
</prompt>

<prompt>
ok sounds good lets update the docs
</prompt>

## [0.1.4]

### Changed
- すべてのドキュメントをSemantic Kernel + Hangfireアーキテクチャに更新しました
- MastraからSemantic Kernel + Hangfireへの移行を完了しました

### Added
- architecture.mdにSemantic KernelとHangfireの統合詳細を追加しました
- AI エージェントオーケストレーションとジョブスケジューリングの明確な分離を文書化しました

### Fixed
- すべてのドキュメント間でワークフローオーケストレーション参照の一貫性を確保しました

### Prompts Used
<prompt>
OK let's update all the docs to match our findings.
</prompt>

## [0.1.3]

### Changed
- すべてのワークフローオーケストレーションオプションが無料であることを確認しました

### Added
- Semantic Kernel、Hangfire、Mastra、Dify 2.0のライセンスコスト分析を追加しました（すべて$0）
- 開発・運用コストの詳細比較を文書化しました
- アーキテクチャの複雑さが総所有コストに与える影響を分析しました

### Fixed
- フレームワークのライセンスコストに関する誤解を解消しました

### Prompts Used
<prompt>
How about cost? Is there cost on using Semantic Kernel + Hangfire?
</prompt>

## [0.1.2]

### Changed
- MastraとHangfireの根本的な違いを明確化しました（AIエージェントフレームワーク vs ジョブスケジューラー）

### Added
- Semantic Kernel + Hangfireの組み合わせを推奨ソリューションとして文書化しました
- Mastraが提供するAI機能とHangfireでは実現できない機能の詳細比較を追加しました
- .NETエコシステムでMastra相当の機能を実現する実装例を追加しました

### Fixed
- MastraとHangfireの役割の誤解を解消し、それぞれの適切な使用場面を明確化しました

### Prompts Used
<prompt>
Do you know what Mastra does? Does what I wanted Mastra to do compatible with what Hangfire is able to do?
</prompt>

## [0.1.1]

### Changed
- Mastraワークフローオーケストレーションをより適切なソリューションに置き換える推奨事項を追加しました
- Dify 2.0の採用を見送り、現在のアーキテクチャスタックを維持することを決定しました

### Added
- Mastraと代替ソリューション（Temporal、Inngest、Hangfire、BullMQ）の詳細な比較分析を追加しました
- Hangfire + カスタムAIオーケストレーションレイヤーの推奨実装アプローチを文書化しました
- Dify 2.0の機能分析とJobHunt AIへの適用性評価を追加しました

### Fixed
- TypeScript専用のMastraと.NET Coreバックエンドアーキテクチャ間の言語不一致問題を特定しました

### Prompts Used
<prompt>
Title: Considering dify 2.0

Description:Initially, it made sense to perform tasks in code, but I was thinking if it make sense to utilize something like dify 2.0 especially since it has some what of agentic power associated with it. I know it's currently in beta, but want to explore the possibility. Also I'm not saying that you should do it, I need your opinion on if it's worth it, or if it's overkill. Why should or why should I not utilize it.
</prompt>

<prompt>
Why Mastra? I suggested it, but I am not sure if it is the best solution. Help me understand.
</prompt>

## [0.1.0]

### Changed
- AI モデルの使用をコスト最適化のためGemini 2.5 Flashに制限しました
- マルチプロバイダー戦略から単一モデル戦略に変更しました
- フォールバックプロバイダーを削除してコスト管理を改善しました

### Added
- Gemini 2.5 Flashを唯一の承認済みAIモデルとして仕様書に明記しました
- AIサービスセクションにコスト最適化の根拠を追加しました
- tasks.mdファイルを作成して今後のタスクを追跡できるようにしました

### Fixed
- 複数のドキュメント間でAIモデル参照の不一致を修正しました

### Prompts Used
<prompt>
Title: Restrict the AI usage to gemini 2.5 flash

Description:Current requirements does not specify the model to use. Meaning cost is not considered. That should be. though. I would like to use gemini 2.5 flash and only that because of the cost. Update spec
</prompt>

## Changelog Format

The changelog follows the recommendations of @Keep a Changelog.

This project adheres to @Semantic Versioning.

### Meaning of Version Numbers
- Major version: "Changed" for incompatible API changes
- Minor version: "Added" for functionality in a backwards-compatible manner
- Patch version: "Fixed" for backwards-compatible bug fixes