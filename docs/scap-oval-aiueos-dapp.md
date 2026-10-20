# app-scap（OVAL definition）— itonami（dapp）上の分散型設計

状態: 第 1 スライス実装（2026-10-20）。正本はこの文書。
追跡: Linear JUN-128（親 JUN-116）。

## 位置づけ

app-scap は SCAP（Security Content Automation Protocol）—— CVE / OVAL /
SCAP content のデータ面（clean-architecture の TS: ports / resources、
**zod schema が唯一の正本**）。本スライスは **OVAL Definition の検証と
criteria 評価**を判定コア化。

## このスライス: OVAL 判定コア

| 層 | 実体 | 状態 |
|---|---|---|
| 判定コア guest | `aiueos/scap_oval.cljk` — class-ok（**vulnerability / compliance / inventory の閉じた集合**）/ criteria-eval（**AND = すべて真 / OR = 1 つでも真、空 criteria は AND 空虚に真・OR 偽**）/ criterion-ok（testRef と comment の両方）/ field-ok（id/title/description は長さ 1 以上） | **実装済み・実測** |
| 記述子 | `aiueos/scap-oval.edn` | **実装済み** |
| parity | `resources/schemas/zod/scap/oval_parity_test.cljk` — 正本 zod schema の規約一致検査（enum 値・operator・criterion フィールド・min(1)） | **実装済み・実測** |

実測: `amu test aiueos/scap_oval.cljk` → **21/21**（jvm-kir / js / wasm）。
parity → 11 assertions / 0 failures。

## 正本の正確な規則（guest はこれに追従 — zod schema の明文）

- `ovalClassSchema` = enum ["vulnerability", "compliance", "inventory"]。
- `ovalCriteriaSchema` = {operator: enum ["AND","OR"], criterion: [{testRef, comment}]}。
  guest の criteria-eval はこの operator 意味論（AND/OR + 空集合の扱い）を
  判定する。**空 criteria の AND は空虚に真**（schema は空配列を許すため、
  評価側の意味論として固定）。
- `ovalDefinitionSchema`: id / title / description は `z.string().min(1)`。

## 実測メモ

- parity ファイルの文字列リテラル内のエスケープ引用符が kbb の reader で
  壊れた —— ピン留めは引用符無しの部分一致に変更（guest の docstring 規約
  「引用符禁止」と同じクラスの問題）。
- zod 実行 parity は npm 依存のため未実施 —— ソース一致検査で代替
  （実行 parity は正本の vitest 側が担う）。

## 未実装境界

- OVAL test / state / object の評価（criteria の葉の実判定）。
- CVE data / scapContent / integration 各 zod schema の判定化。
- GraphQL adapter / projection の判定。
- mon（DNA）化、aiueos session 載せ（fleet 共通）。
