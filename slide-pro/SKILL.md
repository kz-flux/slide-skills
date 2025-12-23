---
name: slide-pro
description: コンサルレベルのスライド作成を一気通貫でサポート。情報設計からpython-pptx実装まで全工程をカバー。PowerPointスライド作成、プレゼン資料、提案書作成時に使用。キーワード: スライド、pptx、PowerPoint、プレゼン、提案書、コンサル
---

# Slide Pro - 統合スライド作成 Skill

slide-master（情報設計）+ slide-design（技術実装）の統合版。

---

## ワークフロー

```
1. 目的確認 → 誰に何を伝えるか
2. 構成設計 → ストーリーライン、メッセージ
3. デザインすり合わせ → 2-3パターン提示
4. 実装 → python-pptxでネイティブオブジェクト
5. セルフレビュー → チェックリスト確認
6. 納品
```

---

## 情報設計原則

### 必須
| 原則 | チェック |
|------|---------|
| 1スライド1メッセージ | タイトル＝結論 |
| So What? | 示唆が明確 |
| 構造化 | テキスト羅列NG |

### 推奨
| 原則 | チェック |
|------|---------|
| ピラミッド構造 | 結論→根拠→詳細 |
| 3の法則 | 項目3-5つ |
| MECE | 漏れなく重複なく |
| アクションドリブン | Next Actionあり |

---

## 技術ルール

### 編集可能性
python-pptxでネイティブオブジェクト生成。HTML→画像変換は禁止。

### 角丸四角形は禁止
```python
# OK
MSO_SHAPE.RECTANGLE

# NG
MSO_SHAPE.ROUNDED_RECTANGLE
```

### テンプレート
```python
prs = Presentation(r'C:\Users\KazuhideAkitake\Desktop\NTTDテンプレver1.2.pptx')
layout = prs.slide_masters[2].slide_layouts[6]  # 白紙（フッター有）
```

---

## 色使いルール

### 禁止: パステル多色

```python
# NG: 派手なパステルの組み合わせ
RGBColor(200, 230, 201)  # パステルグリーン
RGBColor(187, 222, 251)  # パステルブルー
RGBColor(225, 190, 231)  # パステルパープル
```

### 推奨: モノトーン + アクセント1色

```python
# OK: グレー系 + メイン + アクセント1色
BG_LIGHT = RGBColor(250, 250, 250)
GRAY = RGBColor(235, 235, 235)
PRIMARY = RGBColor(0, 85, 164)    # ダークブルー
ACCENT = RGBColor(0, 150, 136)    # ティール（1色のみ）
TEXT_DARK = RGBColor(33, 33, 33)
TEXT_LIGHT = RGBColor(117, 117, 117)
```

### 判断フロー
```
強調したい数字 → アクセント色OK
区別したいカテゴリ → グレー濃淡で対応
彩りが欲しい → NG（余計）
```

---

## スライドパターン

### エグゼクティブサマリー
```
[タイトルバー: 結論1文]
[大きな数字: インパクト]
[3カラム: 根拠カード]
```

### 課題スライド
```
[タイトル: So What?含む]
[3カラム: 課題カード（MECE）]
[インパクトバー: 影響額]
[ボトムライン + Next Action]
```

### ロードマップ
```
[タイトル: 全体像]
[3フェーズ: 時系列（左→右）]
[サマリーバー: 投資/リターン]
[承認事項]
```

---

## チェックリスト

### 情報設計
- [ ] タイトル＝結論
- [ ] 1スライド1メッセージ
- [ ] So What?が明確
- [ ] 項目3-5つ

### 技術
- [ ] 全オブジェクトが編集可能
- [ ] 角丸四角形なし

### 色
- [ ] パステル多色使いなし
- [ ] メイン+アクセント1色

---

## クイックリファレンス

```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.enum.shapes import MSO_SHAPE
from pptx.dml.color import RGBColor
from pptx.enum.text import PP_ALIGN

PRIMARY = RGBColor(0, 85, 164)
ACCENT = RGBColor(0, 150, 136)
TEXT_DARK = RGBColor(33, 33, 33)
BG_LIGHT = RGBColor(250, 250, 250)
FONT_MAIN = "游ゴシック"
```

---

## 連携

| Skill | 役割 |
|-------|------|
| slide-master | 情報設計のみ |
| slide-design | 技術実装のみ |
| **slide-pro** | 両方統合 |

---

## 更新履歴

| 日付 | 内容 |
|------|------|
| 2024-12-23 | 初版。統合版作成、パステル多色禁止 |
