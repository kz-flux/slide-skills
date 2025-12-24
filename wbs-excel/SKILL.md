---
name: wbs-excel
description: WBS（Work Breakdown Structure）をExcelで作成・編集。openpyxlでガントチャート付きWBSを生成。プロジェクト計画、タスク管理、スケジュール作成時に使用。キーワード: WBS, ガントチャート, Excel, タスク一覧, 工程表, スケジュール表
allowed-tools: Read, Write, Edit, Bash, Glob
---

# WBS Excel Skill

プロジェクトのWBS（Work Breakdown Structure）をExcelで作成するためのガイドライン。
openpyxl を使用し、WORKDAY関数によるカレンダー計算、ガントチャート自動生成を行う。

---

## 失敗から学んだ教訓【重要】

### ❌ やってはいけないこと

| 失敗パターン | 問題点 | 正しいアプローチ |
|-------------|--------|-----------------|
| **バッファ無視** | 最終報告日ギリギリで設計 | 最終日の2-3営業日前に完了設計 |
| **祝日考慮漏れ** | 土日のみ除外で日程ズレ | 年末年始・祝日リストを必ず定義 |
| **曜日表示なし** | ユーザーが日付感覚を掴みにくい | ヘッダーに曜日表示（月火水...） |
| **WBS直接編集** | ユーザーの修正を追跡できない | バージョン付きファイル名で保存 |
| **予定日数ハードコード** | 計算式が壊れる | WORKDAY関数で終了日を自動計算 |

### ✅ 必ずやること

1. **祝日リストを最初に定義する**
   - 年末年始（12/29-1/3）
   - 国民の祝日（成人の日、建国記念日等）
   - 企業固有の休日（確認必要）

2. **WORKDAY関数を使う**
   - 開始日 + 予定日数 → 終了日を自動計算
   - 祝日シートを参照させる

3. **バッファを確保する**
   - 最終報告日の2-3営業日前にドキュメントレビュー完了
   - 想定外のリスク用に余裕を持たせる

4. **曜日を表示する**
   - ガントチャートのヘッダーに曜日を追加
   - 日本語表記: 月,火,水,木,金,土,日

---

## 標準構造

### シート構成

| シート名 | 内容 |
|---------|------|
| WBS | メインのタスク一覧 + ガントチャート |
| Holidays | 祝日リスト（WORKDAY関数参照用） |

### 列構成

| 列 | 内容 | 幅 |
|----|------|-----|
| A | No | 4 |
| B | フェーズ | 14 |
| C | 大項目 | 16 |
| D | 詳細タスク | 45 |
| E | 担当 | 10 |
| F | 開始日 | 11 |
| G | 予定日数 | 8 |
| H | 終了日（自動計算） | 11 |
| I以降 | 日付（ガントチャート） | 7 |

---

## コードパターン

### 祝日定義

```python
from datetime import datetime

HOLIDAYS = [
    # 年末年始
    datetime(2025, 12, 29),
    datetime(2025, 12, 30),
    datetime(2025, 12, 31),
    datetime(2026, 1, 1),
    datetime(2026, 1, 2),
    datetime(2026, 1, 3),
    # 祝日
    datetime(2026, 1, 13),   # 成人の日
    datetime(2026, 2, 11),   # 建国記念日
    datetime(2026, 2, 23),   # 天皇誕生日
    datetime(2026, 3, 20),   # 春分の日
]
```

### 曜日表示付きヘッダー

```python
WEEKDAY_JP = ['月', '火', '水', '木', '金', '土', '日']

for i, day in enumerate(business_days):
    weekday = WEEKDAY_JP[day.weekday()]
    header = f"{day.strftime('%m/%d')}({weekday})"
```

### WORKDAY関数（終了日自動計算）

```python
# 終了日セルにWORKDAY関数を設定
# =WORKDAY(開始日, 予定日数-1, Holidays!A:A)
ws.cell(row=row, column=8).value = f'=WORKDAY(F{row},G{row}-1,Holidays!A:A)'
ws.cell(row=row, column=8).number_format = 'YYYY/MM/DD'
```

### ガントチャート条件付き書式

```python
from openpyxl.formatting.rule import FormulaRule
from openpyxl.styles import PatternFill

# 担当別の色
OWNER_COLORS = {
    'FLUX': PatternFill(start_color='C8DCFF', end_color='C8DCFF', fill_type='solid'),
    '東邦ガス': PatternFill(start_color='FFDAC8', end_color='FFDAC8', fill_type='solid'),
    'NTTD': PatternFill(start_color='E0E0E0', end_color='E0E0E0', fill_type='solid'),
}

# 条件付き書式でガントチャート自動着色
formula = f'AND(I$1>=$F{row},I$1<=$H{row})'
rule = FormulaRule(formula=[formula], fill=OWNER_COLORS[owner])
```

---

## バージョニングルール

| 変更内容 | バージョン |
|---------|-----------|
| タスク追加・削除 | v4.3 → v4.4 |
| 日程大幅変更 | v4.x → v5.0 |
| 列構成変更 | v4.x → v5.0 |

**ファイル名例**: `WBS_v4.4.xlsx`

---

## チェックリスト

### 作成時
- [ ] 祝日リストを定義した
- [ ] WORKDAY関数で終了日を自動計算している
- [ ] ヘッダーに曜日が表示されている
- [ ] 最終報告日の2-3営業日前にバッファがある
- [ ] ガントチャートの日付範囲が適切

### 納品時
- [ ] バージョン付きファイル名で保存
- [ ] 既存ファイルを上書きしていない
- [ ] 計算式が正しく動作する（data_only=Falseで確認）

---

## よくある質問

### Q: ユーザーがExcelで日程を変更したらどうなる？
A: 開始日と予定日数を変更すれば、終了日とガントチャートは自動更新される。

### Q: 祝日を追加したい場合は？
A: Holidaysシートに行を追加するだけでOK。WORKDAY関数が自動参照。

### Q: ガントチャートの色がおかしい場合は？
A: 条件付き書式の範囲と数式を確認。開始日・終了日のセル参照が正しいか確認。

---

## 更新履歴

| 日付 | 内容 |
|------|------|
| 2024-12-24 | v1.0.0: 初版。東邦ガスWBS作成の教訓を反映 |
