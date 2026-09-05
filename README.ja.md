# Curling Stone Placement JSON

[English](README.md)

アプリケーションやツール間で、カーリングストーン配置を交換するためのJSON Text Formatです。

フォーマットは **v1.1** で、v1.0との互換性を扱います。

## 言語方針

英語版をcanonical specification（正本）とします。日本語版はofficial translation（公式翻訳）として提供します。この方針は日英のMCE実装仕様書にも適用します。翻訳の相違は英語本文を基準にレビュー・訂正します。

## 文書一覧

| 文書 | English | 日本語 |
| --- | --- | --- |
| 形式・スケール・精度・互換性 | [Specification](spec/JSON_Text_Format_v1.1.md) | [一般仕様書](spec/JSON_Text_Format_v1.1.ja.md) |
| MCE v1.3.0 Reader／Writerの対応範囲と動作 | [Implementation Specification](implementations/MCE_v1.3.0.md) | [実装仕様書](implementations/MCE_v1.3.0.ja.md) |

## 一般仕様書とアプリの対応範囲

v1.1はv1.0の構造にhouseRadiusを追加します。その数値が座標スケールを定め、unitは任意の非空ラベルです。一般仕様書では半径比による正規化と適切な座標分解能を推奨します。

一般仕様書は、有効なデータとその意味を定義します。Readerの対応範囲、Writerの整形、不正入力からの回復動作は、各実装仕様書で別途記録します。アプリ固有の動作は上記の実装仕様書を参照してください。

## ライセンス

本文書は [Creative Commons Attribution-NoDerivatives 4.0 International（CC BY-ND 4.0）](https://creativecommons.org/licenses/by-nd/4.0/) の下で提供します。

クレジットは **RC spica Factory** としてください。

このライセンスは、本リポジトリの仕様書・ドキュメントに適用します。本仕様の実装、または本仕様に準拠したJSONデータの作成、利用、変更、交換を制限するものではありません。アプリケーション、Reader、Writer、ツール等の実装、および準拠JSONデータの編集、変換、保存、配布も制限しません。改変禁止（NoDerivatives）の条件はドキュメントに適用され、これらの実装やJSONデータには適用されません。

ドキュメントの原文は、同ライセンスの条件に従って共有・再配布できます。同ライセンスは、改変したドキュメントを派生版として共有・再配布する権利を許諾するものではありません。

ライセンス表示は [LICENSE](LICENSE)、条件の全文は [公式ライセンス本文](https://creativecommons.org/licenses/by-nd/4.0/legalcode.en) を参照してください。
