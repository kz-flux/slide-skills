# python-pptx コードパターン集

## 基本セットアップ

```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.enum.text import PP_ALIGN
from pptx.enum.shapes import MSO_SHAPE
from pptx.dml.color import RGBColor  # ※RgbColorではない（大文字注意）

# NTTDテンプレート読み込み
prs = Presentation(r'C:\Users\KazuhideAkitake\Desktop\NTTDテンプレver1.2.pptx')

# スライドサイズ（16:9）
# prs.slide_width = Inches(13.333)  # テンプレートから継承
# prs.slide_height = Inches(7.5)
```

---

## スライド追加パターン

### Master/Layout選択
```python
# Master 0: 表紙系（濃紺背景）
cover_master = prs.slide_masters[0]
layout_cover = cover_master.slide_layouts[0]  # イノベーションカーブ

# Master 2: メインコンテンツ（白背景）
main_master = prs.slide_masters[2]
layout_title_1content = main_master.slide_layouts[1]  # タイトルと1コンテンツ
layout_blank_footer = main_master.slide_layouts[6]    # 白紙（フッター有）

# Master 3: 中扉
divider_master = prs.slide_masters[3]
layout_divider = divider_master.slide_layouts[0]      # 白背景

# スライド追加
slide = prs.slides.add_slide(layout_blank_footer)
```

---

## シェイプパターン

### 背景（枠線なし）
```python
bg = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, 0, 0, prs.slide_width, prs.slide_height)
bg.fill.solid()
bg.fill.fore_color.rgb = RGBColor(0x1a, 0x1a, 0x2e)
bg.line.fill.background()  # 枠線なし
```

### アクセントライン
```python
accent = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, x, y, Inches(0.06), Inches(2))
accent.fill.solid()
accent.fill.fore_color.rgb = RGBColor(0x00, 0xd4, 0xff)
accent.line.fill.background()
```

### カード（情報ボックス）
```python
# カード背景（角丸ではなく通常の四角形）
card = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, x, y, w, h)
card.fill.solid()
card.fill.fore_color.rgb = RGBColor(0xf8, 0xf8, 0xf8)
card.line.color.rgb = RGBColor(0xe0, 0xe0, 0xe0)  # ボーダー

# 上部アクセントライン
accent = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, x, y, w, Inches(0.06))
accent.fill.solid()
accent.fill.fore_color.rgb = RGBColor(0x00, 0x55, 0xff)
accent.line.fill.background()
```

### 矢羽根（スケジュール用）
```python
# くぼみのない矢羽根
shape = slide.shapes.add_shape(MSO_SHAPE.PENTAGON, left, top, width, height)
# 注意: CHEVRONはくぼみあり、PENTAGONはくぼみなし
```

---

## テキストパターン

### テキストボックス
```python
text_box = slide.shapes.add_textbox(Inches(1), Inches(2), Inches(8), Inches(1))
tf = text_box.text_frame
tf.word_wrap = True

p = tf.paragraphs[0]
p.text = "タイトルテキスト"
p.font.size = Pt(24)
p.font.bold = True
p.font.name = "Noto Sans JP"  # フォント明示指定
p.font.color.rgb = RGBColor(0x1a, 0x1a, 0x2e)
p.alignment = PP_ALIGN.CENTER
```

### 複数段落
```python
tf = text_box.text_frame
tf.word_wrap = True

# 最初の段落
p1 = tf.paragraphs[0]
p1.text = "見出し"
p1.font.size = Pt(18)
p1.font.bold = True

# 追加段落
p2 = tf.add_paragraph()
p2.text = "本文テキスト"
p2.font.size = Pt(12)
```

---

## チャートパターン

### バーチャート（シェイプで構築）
```python
bars = [("2021", 25), ("2022", 40), ("2023", 55), ("2024", 78)]
max_val = max(v for _, v in bars)
max_height = Inches(4)
base_y = Inches(6)
bar_width = Inches(1.2)

for i, (label, val) in enumerate(bars):
    x = Inches(1.5 + i * 1.5)
    bar_height = (val / max_val) * max_height
    bar_y = base_y - bar_height

    # バー本体
    bar = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, x, bar_y, bar_width, bar_height)
    bar.fill.solid()
    bar.fill.fore_color.rgb = RGBColor(0x00, 0x55, 0xff)
    bar.line.fill.background()

    # 値ラベル
    val_box = slide.shapes.add_textbox(x, bar_y - Inches(0.4), bar_width, Inches(0.4))
    p = val_box.text_frame.paragraphs[0]
    p.text = str(val)
    p.font.size = Pt(14)
    p.font.bold = True
    p.alignment = PP_ALIGN.CENTER
```

---

## カラーパレット

### NTTD標準
```python
NTTD_BLUE = RGBColor(0x00, 0x55, 0xff)     # ブランドカラー
NTTD_DARK = RGBColor(0x1a, 0x1a, 0x2e)     # 濃紺背景
NTTD_TEXT = RGBColor(0x43, 0x43, 0x43)     # 通常テキスト
NTTD_GRAY = RGBColor(0x99, 0x99, 0x99)     # 補足テキスト
NTTD_WHITE = RGBColor(0xff, 0xff, 0xff)    # 白
```

### アクセントカラー
```python
ACCENT_TEAL = RGBColor(0x4e, 0xcd, 0xc4)   # ティール
ACCENT_PINK = RGBColor(0xe9, 0x45, 0x60)   # ピンク
ACCENT_RED = RGBColor(0xcc, 0x00, 0x00)    # 赤
```

---

## フォント設定

| 用途 | フォント | サイズ |
|------|----------|--------|
| メインタイトル | Noto Sans JP Bold | 28-44pt |
| サブタイトル | Noto Sans JP | 18-20pt |
| 見出し | Noto Sans JP Bold | 14-18pt |
| 本文 | Noto Sans JP | 10-12pt |
| 補足 | Noto Sans JP | 8-10pt |
| 数字・英語 | Montserrat | 各種 |

---

## 保存
```python
output_path = r"C:\Users\KazuhideAkitake\Downloads\output.pptx"
prs.save(output_path)
print(f"Created: {output_path}")
```
