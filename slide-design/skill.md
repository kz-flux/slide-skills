---
name: slide-design
description: PowerPointスライドを作成・編集する際のデザインガイドライン。python-pptxでスライドを作成するとき、pptxファイルを編集するとき、プレゼンテーション資料を作成するとき、テンプレートに合わせたスライドを作成するときに使用する。
allowed-tools: Read, Write, Edit, Bash, Glob
---

# スライドデザイン Skill

詳細は以下を参照:
- [reference.md](reference.md) - テンプレート詳細、レイアウト情報
- [code-patterns.md](code-patterns.md) - python-pptxコードパターン
- [components.md](components.md) - 再利用可能なコンポーネント

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

## カラーパレットテンプレート

プロジェクト開始時にカラークラスを定義する:

```python
from pptx.dml.color import RGBColor

class ProjectColors:
    """プロジェクト固有のカラーパレット"""
    # メインカラー
    PRIMARY = RGBColor(0x00, 0x55, 0xFF)       # ブランドカラー
    PRIMARY_DARK = RGBColor(0x00, 0x3D, 0xB3)  # 濃いバージョン
    PRIMARY_LIGHT = RGBColor(0x66, 0x99, 0xFF) # 薄いバージョン

    # アクセント（1色のみ）
    ACCENT = RGBColor(0x00, 0x96, 0x88)        # ティール

    # テキスト
    TEXT_DARK = RGBColor(0x33, 0x33, 0x33)     # 本文
    TEXT_LIGHT = RGBColor(0x75, 0x75, 0x75)    # 補足

    # 背景・ボーダー
    BG_LIGHT = RGBColor(0xFA, 0xFA, 0xFA)      # 薄い背景
    BORDER = RGBColor(0xE0, 0xE0, 0xE0)        # ボーダー
    WHITE = RGBColor(0xFF, 0xFF, 0xFF)

    # 警告（必要な場合のみ）
    ALERT = RGBColor(0xE5, 0x39, 0x35)         # 赤
```

---

## コンポーネント一覧

### 基本コンポーネント

| コンポーネント | 用途 | 詳細 |
|---------------|------|------|
| `add_content_slide()` | スライド基本構造 | ヘッダー + タイトル + ページ番号 |
| `add_key_message_box()` | キーメッセージ | 強調ボックス |
| `add_info_card()` | KPIカード | 数値表示 |
| `add_bullet_list()` | 箇条書き | リスト表示 |
| `add_table()` | テーブル | データ表示 |

### 拡張コンポーネント

| コンポーネント | 用途 | 詳細 |
|---------------|------|------|
| `add_kpi_row()` | KPIカード横並び | 2-4列対応 |
| `add_rank_card()` | ランクカード | バッジ付き |
| `add_timeline()` | タイムライン | フェーズ表示 |
| `add_comparison_columns()` | 2カラム比較 | Before/After |

詳細コードは [components.md](components.md) を参照。

---

## チェックリスト

- [ ] 全オブジェクトが編集可能
- [ ] 角丸四角形を使っていない
- [ ] フォント: Noto Sans JP (日本語) / Montserrat (英語)
- [ ] カラー: 3色以内
- [ ] スライドマスターが適切に選択されている
- [ ] グリッド整列（同列要素のズレなし）

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
from pptx.enum.text import PP_ALIGN
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
| 2024-12-24 | v2.0.0: カラーパレットテンプレート追加、コンポーネント一覧追加 |
| 2024-12-23 | v1.0.0: Skill簡潔化、詳細をreference.md/code-patterns.mdに分離 |
