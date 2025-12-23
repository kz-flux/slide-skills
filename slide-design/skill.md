---
name: slide-design
description: PowerPointスライドを作成・編集する際のデザインガイドライン。python-pptxでスライドを作成するとき、pptxファイルを編集するとき、プレゼンテーション資料を作成するとき、テンプレートに合わせたスライドを作成するときに使用する。
allowed-tools: Read, Write, Edit, Bash, Glob
---

# スライドデザイン Skill

詳細は以下を参照:
- [reference.md](reference.md) - テンプレート詳細、レイアウト情報
- [code-patterns.md](code-patterns.md) - python-pptxコードパターン

---

## 核心ルール【必須】

### 1. 編集可能性を最優先
- **python-pptx でネイティブオブジェクトを生成**（HTML→画像変換は避ける）
- 全てのテキスト・シェイプが編集可能であること

### 2. 角丸四角形は禁止
```python
# ✅ 正しい
MSO_SHAPE.RECTANGLE

# ❌ 禁止
MSO_SHAPE.ROUNDED_RECTANGLE
```

### 3. NTTDテンプレート使用
- テンプレート: `C:\Users\KazuhideAkitake\Desktop\NTTDテンプレver1.2.pptx`
- スライドタイプに応じてMaster/Layoutを選択

| 目的 | Master | Layout |
|------|--------|--------|
| 表紙 | 0 | イノベーションカーブ |
| コンテンツ | 2 | タイトルと1コンテンツ or 白紙（フッター有） |
| 中扉 | 3 | 白背景 |

### 4. 構造で見せる
- テキスト羅列ではなくテーブルで比較
- 複雑な図はMermaid→画像→手動挿入

---

## ワークフロー

```
1. テンプレート確認
   └→ NTTDテンプレート or 別テンプレートか確認

2. デザインすり合わせ
   └→ 2-3パターン提示 → ユーザー承認

3. 実装
   └→ python-pptxでネイティブオブジェクト生成

4. レビュー（3スライド以上の場合）
   └→ サブエージェントでレビュー → 改善

5. 納品
   └→ チェックリスト確認 → ファイル出力
```

---

## チェックリスト

- [ ] 全オブジェクトが編集可能
- [ ] 角丸四角形を使っていない
- [ ] フォント: Noto Sans JP (日本語) / Montserrat (英語)
- [ ] カラー: 3色以内
- [ ] スライドマスターが適切に選択されている

---

## サブエージェントレビュー

3スライド以上生成時、自動でレビュー実行:

```
Task tool:
  prompt: "生成したPPTXをレビュー。情報設計、視覚階層、一貫性、編集可能性を確認"
  subagent_type: "general-purpose"
```

---

## クイックリファレンス

### 基本import
```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.enum.shapes import MSO_SHAPE
from pptx.dml.color import RGBColor  # ※RgbColorではない
```

### テンプレート読み込み
```python
prs = Presentation(r'C:\Users\KazuhideAkitake\Desktop\NTTDテンプレver1.2.pptx')
layout = prs.slide_masters[2].slide_layouts[6]  # 白紙（フッター有）
slide = prs.slides.add_slide(layout)
```

### シェイプ追加
```python
shape = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, x, y, w, h)
shape.fill.solid()
shape.fill.fore_color.rgb = RGBColor(0x00, 0x55, 0xff)
shape.line.fill.background()  # 枠線なし
```

詳細コードは [code-patterns.md](code-patterns.md) を参照。

---

## 更新履歴
| 日付 | 内容 |
|------|------|
| 2024-12-23 | Skill簡潔化、詳細をreference.md/code-patterns.mdに分離 |
| 2024-12-23 | NTTDテンプレート情報追加、角丸四角形禁止ルール追加 |
