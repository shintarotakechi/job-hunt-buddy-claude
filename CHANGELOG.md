# Changelog

This file records all important changes for JobHunt AI.

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