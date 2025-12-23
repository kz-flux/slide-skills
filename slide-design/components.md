# スライドコンポーネント集

再利用可能なスライドコンポーネントのコードパターン。
**最低品質の担保**が目的。内容に応じて調整・組み合わせること。

---

## 基本コンポーネント

### add_content_slide() - スライド基本構造

```python
def add_content_slide(
    prs: Presentation,
    title: str,
    page_number: int = None
) -> tuple:
    """
    コンテンツスライドの基本レイアウトを追加

    Returns:
        tuple: (slide, content_top) - スライドとコンテンツ開始位置
    """
    blank_layout = prs.slide_layouts[6]
    slide = prs.slides.add_slide(blank_layout)

    # ヘッダーライン
    header_line = slide.shapes.add_shape(
        MSO_SHAPE.RECTANGLE,
        Inches(0), Inches(0),
        prs.slide_width, Inches(0.08)
    )
    header_line.fill.solid()
    header_line.fill.fore_color.rgb = ProjectColors.PRIMARY
    header_line.line.fill.background()

    # タイトル
    title_box = slide.shapes.add_textbox(
        Inches(0.5), Inches(0.3),
        Inches(12), Inches(0.6)
    )
    tf = title_box.text_frame
    p = tf.paragraphs[0]
    p.text = title
    p.font.size = Pt(24)
    p.font.bold = True
    p.font.color.rgb = ProjectColors.TEXT_DARK
    p.font.name = "Noto Sans JP"

    # ページ番号
    if page_number:
        page_box = slide.shapes.add_textbox(
            Inches(12.5), Inches(7),
            Inches(0.5), Inches(0.3)
        )
        tf = page_box.text_frame
        p = tf.paragraphs[0]
        p.text = str(page_number)
        p.font.size = Pt(10)
        p.font.color.rgb = ProjectColors.TEXT_LIGHT
        p.alignment = PP_ALIGN.RIGHT

    content_top = Inches(1.1)
    return slide, content_top
```

---

### add_key_message_box() - キーメッセージ

```python
def add_key_message_box(
    slide,
    message: str,
    left: float = 0.5,
    top: float = 1.1,
    width: float = 12.333,
    height: float = 0.6
) -> None:
    """キーメッセージボックスを追加（ハイライト用）"""
    box = slide.shapes.add_shape(
        MSO_SHAPE.RECTANGLE,
        Inches(left), Inches(top),
        Inches(width), Inches(height)
    )
    box.fill.solid()
    box.fill.fore_color.rgb = ProjectColors.BG_LIGHT
    box.line.color.rgb = ProjectColors.PRIMARY
    box.line.width = Pt(1.5)

    tf = box.text_frame
    tf.word_wrap = True
    tf.paragraphs[0].alignment = PP_ALIGN.LEFT
    p = tf.paragraphs[0]
    p.text = message
    p.font.size = Pt(14)
    p.font.bold = True
    p.font.color.rgb = ProjectColors.PRIMARY
    p.font.name = "Noto Sans JP"

    # 垂直中央揃え
    tf.paragraphs[0].space_before = Pt(8)
```

---

### add_info_card() - KPIカード（単体）

```python
def add_info_card(
    slide,
    title: str,
    value: str,
    subtitle: str = "",
    left: float = 0,
    top: float = 0,
    width: float = 3,
    height: float = 1.5,
    accent_color: RGBColor = None
) -> None:
    """情報カード（KPI表示用）を追加"""
    if accent_color is None:
        accent_color = ProjectColors.PRIMARY

    # カード背景
    card = slide.shapes.add_shape(
        MSO_SHAPE.RECTANGLE,
        Inches(left), Inches(top),
        Inches(width), Inches(height)
    )
    card.fill.solid()
    card.fill.fore_color.rgb = ProjectColors.WHITE
    card.line.color.rgb = ProjectColors.BORDER
    card.line.width = Pt(1)

    # 上部のアクセントライン
    accent_line = slide.shapes.add_shape(
        MSO_SHAPE.RECTANGLE,
        Inches(left), Inches(top),
        Inches(width), Inches(0.06)
    )
    accent_line.fill.solid()
    accent_line.fill.fore_color.rgb = accent_color
    accent_line.line.fill.background()

    # タイトル
    title_box = slide.shapes.add_textbox(
        Inches(left + 0.15), Inches(top + 0.15),
        Inches(width - 0.3), Inches(0.3)
    )
    tf = title_box.text_frame
    p = tf.paragraphs[0]
    p.text = title
    p.font.size = Pt(10)
    p.font.color.rgb = ProjectColors.TEXT_LIGHT
    p.font.name = "Noto Sans JP"

    # 値
    value_box = slide.shapes.add_textbox(
        Inches(left + 0.15), Inches(top + 0.45),
        Inches(width - 0.3), Inches(0.6)
    )
    tf = value_box.text_frame
    p = tf.paragraphs[0]
    p.text = value
    p.font.size = Pt(28)
    p.font.bold = True
    p.font.color.rgb = accent_color
    p.font.name = "Noto Sans JP"

    # サブタイトル
    if subtitle:
        sub_box = slide.shapes.add_textbox(
            Inches(left + 0.15), Inches(top + 1.1),
            Inches(width - 0.3), Inches(0.3)
        )
        tf = sub_box.text_frame
        p = tf.paragraphs[0]
        p.text = subtitle
        p.font.size = Pt(9)
        p.font.color.rgb = ProjectColors.TEXT_LIGHT
        p.font.name = "Noto Sans JP"
```

---

## 拡張コンポーネント

### add_kpi_row() - KPIカード横並び

```python
def add_kpi_row(
    slide,
    items: list[dict],
    top: float = 2.0,
    card_width: float = 3.5,
    card_height: float = 1.5,
    gap: float = 0.3
) -> None:
    """
    KPIカードを横並びで配置

    Args:
        items: [{"label": "...", "value": "...", "sub": "...", "color": RGBColor}]
        top: 配置開始Y座標
        card_width: カード幅
        card_height: カード高さ
        gap: カード間の隙間
    """
    start_left = 0.5
    for i, item in enumerate(items):
        left = start_left + i * (card_width + gap)
        add_info_card(
            slide,
            title=item.get("label", ""),
            value=item.get("value", ""),
            subtitle=item.get("sub", ""),
            left=left,
            top=top,
            width=card_width,
            height=card_height,
            accent_color=item.get("color", ProjectColors.PRIMARY)
        )
```

**使用例**:
```python
add_kpi_row(slide, [
    {"label": "総合スコア", "value": "2.32", "sub": "/ 5.0"},
    {"label": "成熟度", "value": "Lv.2", "sub": "部分的導入"},
    {"label": "業界順位", "value": "平均以下", "sub": "製造業", "color": ProjectColors.ALERT},
])
```

---

### add_rank_card() - ランクカード（バッジ付き）

```python
def add_rank_card(
    slide,
    rank: int,
    title: str,
    category: str,
    description: str,
    solutions: list[str] = None,
    left: float = 0,
    top: float = 0,
    width: float = 4,
    height: float = 2.5
) -> None:
    """ランクバッジ付きの課題カードを追加"""
    # ランクに応じた色
    rank_colors = {
        1: ProjectColors.PRIMARY,
        2: ProjectColors.PRIMARY_LIGHT,
        3: ProjectColors.ACCENT,
    }
    badge_color = rank_colors.get(rank, ProjectColors.TEXT_LIGHT)

    # カード背景
    card = slide.shapes.add_shape(
        MSO_SHAPE.RECTANGLE,
        Inches(left), Inches(top),
        Inches(width), Inches(height)
    )
    card.fill.solid()
    card.fill.fore_color.rgb = ProjectColors.WHITE
    card.line.color.rgb = ProjectColors.BORDER
    card.line.width = Pt(1)

    # ランクバッジ（円）
    badge = slide.shapes.add_shape(
        MSO_SHAPE.OVAL,
        Inches(left + 0.15), Inches(top + 0.15),
        Inches(0.4), Inches(0.4)
    )
    badge.fill.solid()
    badge.fill.fore_color.rgb = badge_color
    badge.line.fill.background()

    # ランク番号
    rank_box = slide.shapes.add_textbox(
        Inches(left + 0.15), Inches(top + 0.2),
        Inches(0.4), Inches(0.35)
    )
    tf = rank_box.text_frame
    p = tf.paragraphs[0]
    p.text = str(rank)
    p.font.size = Pt(16)
    p.font.bold = True
    p.font.color.rgb = ProjectColors.WHITE
    p.alignment = PP_ALIGN.CENTER

    # タイトル
    title_box = slide.shapes.add_textbox(
        Inches(left + 0.65), Inches(top + 0.15),
        Inches(width - 0.8), Inches(0.4)
    )
    tf = title_box.text_frame
    tf.word_wrap = True
    p = tf.paragraphs[0]
    p.text = title
    p.font.size = Pt(12)
    p.font.bold = True
    p.font.color.rgb = ProjectColors.TEXT_DARK

    # カテゴリタグ
    cat_box = slide.shapes.add_textbox(
        Inches(left + 0.15), Inches(top + 0.6),
        Inches(width - 0.3), Inches(0.25)
    )
    tf = cat_box.text_frame
    p = tf.paragraphs[0]
    p.text = category
    p.font.size = Pt(9)
    p.font.color.rgb = badge_color

    # 説明
    desc_box = slide.shapes.add_textbox(
        Inches(left + 0.15), Inches(top + 0.9),
        Inches(width - 0.3), Inches(0.8)
    )
    tf = desc_box.text_frame
    tf.word_wrap = True
    p = tf.paragraphs[0]
    p.text = description[:100] + "..." if len(description) > 100 else description
    p.font.size = Pt(9)
    p.font.color.rgb = ProjectColors.TEXT_LIGHT

    # 推奨ソリューション
    if solutions:
        sol_box = slide.shapes.add_textbox(
            Inches(left + 0.15), Inches(top + 1.8),
            Inches(width - 0.3), Inches(0.6)
        )
        tf = sol_box.text_frame
        tf.word_wrap = True
        for i, sol in enumerate(solutions[:2]):
            if i == 0:
                p = tf.paragraphs[0]
            else:
                p = tf.add_paragraph()
            p.text = f"→ {sol}"
            p.font.size = Pt(8)
            p.font.color.rgb = ProjectColors.TEXT_DARK
```

---

### add_timeline() - タイムライン（フェーズ矢羽根）

```python
def add_timeline(
    slide,
    phases: list[dict],
    top: float = 2.0,
    height: float = 1.0
) -> None:
    """
    タイムラインを横並びで配置

    Args:
        phases: [{"name": "Phase 1", "period": "0-6ヶ月", "items": ["項目1", "項目2"]}]
        top: 配置開始Y座標
        height: 矢羽根の高さ
    """
    num_phases = len(phases)
    total_width = 12  # スライド幅 - マージン
    arrow_width = total_width / num_phases
    start_left = 0.5

    phase_colors = [
        ProjectColors.PRIMARY,
        ProjectColors.PRIMARY_LIGHT,
        ProjectColors.ACCENT,
    ]

    for i, phase in enumerate(phases):
        left = start_left + i * arrow_width
        color = phase_colors[i % len(phase_colors)]

        # 矢羽根シェイプ
        arrow = slide.shapes.add_shape(
            MSO_SHAPE.PENTAGON,
            Inches(left), Inches(top),
            Inches(arrow_width - 0.1), Inches(height)
        )
        arrow.fill.solid()
        arrow.fill.fore_color.rgb = color
        arrow.line.fill.background()

        # フェーズ名
        name_box = slide.shapes.add_textbox(
            Inches(left + 0.2), Inches(top + 0.2),
            Inches(arrow_width - 0.4), Inches(0.4)
        )
        tf = name_box.text_frame
        p = tf.paragraphs[0]
        p.text = phase.get("name", f"Phase {i+1}")
        p.font.size = Pt(14)
        p.font.bold = True
        p.font.color.rgb = ProjectColors.WHITE

        # 期間
        period_box = slide.shapes.add_textbox(
            Inches(left + 0.2), Inches(top + 0.55),
            Inches(arrow_width - 0.4), Inches(0.3)
        )
        tf = period_box.text_frame
        p = tf.paragraphs[0]
        p.text = phase.get("period", "")
        p.font.size = Pt(10)
        p.font.color.rgb = ProjectColors.WHITE

        # 詳細項目（矢羽根の下）
        items = phase.get("items", [])
        if items:
            items_box = slide.shapes.add_textbox(
                Inches(left), Inches(top + height + 0.2),
                Inches(arrow_width - 0.1), Inches(1.5)
            )
            tf = items_box.text_frame
            tf.word_wrap = True
            for j, item in enumerate(items[:4]):
                if j == 0:
                    p = tf.paragraphs[0]
                else:
                    p = tf.add_paragraph()
                p.text = f"• {item}"
                p.font.size = Pt(9)
                p.font.color.rgb = ProjectColors.TEXT_DARK
```

**使用例**:
```python
add_timeline(slide, [
    {"name": "Phase 1", "period": "0-6ヶ月", "items": ["基盤整備", "戦略策定"]},
    {"name": "Phase 2", "period": "6-18ヶ月", "items": ["パイロット", "PoC"]},
    {"name": "Phase 3", "period": "18-36ヶ月", "items": ["全社展開", "最適化"]},
])
```

---

### add_comparison_columns() - 2カラム比較

```python
def add_comparison_columns(
    slide,
    left_title: str,
    left_items: list[str],
    right_title: str,
    right_items: list[str],
    top: float = 2.0
) -> None:
    """2カラムの比較レイアウトを追加"""
    col_width = 5.8
    col_height = 4.0
    left_x = 0.5
    right_x = 6.8

    for col_x, title, items, color in [
        (left_x, left_title, left_items, ProjectColors.TEXT_LIGHT),
        (right_x, right_title, right_items, ProjectColors.PRIMARY)
    ]:
        # カラムヘッダー
        header = slide.shapes.add_shape(
            MSO_SHAPE.RECTANGLE,
            Inches(col_x), Inches(top),
            Inches(col_width), Inches(0.5)
        )
        header.fill.solid()
        header.fill.fore_color.rgb = color
        header.line.fill.background()

        header_text = slide.shapes.add_textbox(
            Inches(col_x + 0.2), Inches(top + 0.1),
            Inches(col_width - 0.4), Inches(0.4)
        )
        tf = header_text.text_frame
        p = tf.paragraphs[0]
        p.text = title
        p.font.size = Pt(14)
        p.font.bold = True
        p.font.color.rgb = ProjectColors.WHITE

        # カラム本体
        body = slide.shapes.add_shape(
            MSO_SHAPE.RECTANGLE,
            Inches(col_x), Inches(top + 0.5),
            Inches(col_width), Inches(col_height)
        )
        body.fill.solid()
        body.fill.fore_color.rgb = ProjectColors.BG_LIGHT
        body.line.color.rgb = ProjectColors.BORDER

        # 項目リスト
        items_box = slide.shapes.add_textbox(
            Inches(col_x + 0.2), Inches(top + 0.7),
            Inches(col_width - 0.4), Inches(col_height - 0.4)
        )
        tf = items_box.text_frame
        tf.word_wrap = True
        for i, item in enumerate(items):
            if i == 0:
                p = tf.paragraphs[0]
            else:
                p = tf.add_paragraph()
            p.text = f"• {item}"
            p.font.size = Pt(11)
            p.font.color.rgb = ProjectColors.TEXT_DARK
            p.space_after = Pt(6)
```

**使用例**:
```python
add_comparison_columns(
    slide,
    left_title="Before（現状）",
    left_items=["手動での提案書作成", "2-3日かかる", "属人化"],
    right_title="After（導入後）",
    right_items=["AI自動生成", "8秒で完了", "標準化"],
)
```

---

## 組み合わせ例

### エグゼクティブサマリー

```python
slide, content_top = add_content_slide(prs, "エグゼクティブサマリー", page_number=2)

# キーメッセージ
add_key_message_box(slide, "DX推進により年間1.5億円のコスト削減と売上向上を実現")

# KPIカード
add_kpi_row(slide, [
    {"label": "総合スコア", "value": "2.32", "sub": "/ 5.0"},
    {"label": "期待ROI", "value": "300%", "sub": "3年間"},
    {"label": "回収期間", "value": "2年", "sub": ""},
], top=2.0)

# 課題カード
for i, challenge in enumerate(top3_challenges):
    add_rank_card(
        slide,
        rank=i+1,
        title=challenge["title"],
        category=challenge["category"],
        description=challenge["description"],
        solutions=challenge["solutions"],
        left=0.5 + i * 4.2,
        top=4.0,
        width=4,
        height=2.8
    )
```

---

## 更新履歴

| 日付 | 内容 |
|------|------|
| 2024-12-24 | v1.0.0: 初版。基本・拡張コンポーネント |
