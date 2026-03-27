---
name: lse-brand
description: "Apply Lucky Strike Entertainment brand guidelines to any new document — Word reports (.docx), PowerPoint decks (.pptx), Excel spreadsheets (.xlsx), PDFs, or HTML reports/dashboards. Use this skill whenever the user asks to create a branded document, report, presentation, spreadsheet, or HTML page for Lucky Strike Entertainment (LSE). Also trigger when the user mentions 'company branding', 'branded report', 'LSE template', 'Lucky Strike style', or asks for any document that should look like it comes from Lucky Strike Entertainment. This skill should be used alongside the appropriate format skill (docx, pptx, xlsx, or pdf) — it provides the brand layer, while the format skill handles the file mechanics. For HTML outputs, this skill contains everything needed."
---

# Lucky Strike Entertainment — Brand Guidelines for Documents

This skill defines how every document created for Lucky Strike Entertainment should look and feel. For Office formats (docx, pptx, xlsx) and PDF, it works as a branding layer on top of the format-specific skills — load the relevant format skill alongside this one. For HTML outputs (reports, dashboards, email-friendly pages), this skill is self-contained.

## Brand Overview

Lucky Strike Entertainment is a premium bowling, entertainment, and dining venue chain. The brand is energetic, modern, and celebratory. Their tagline is **"One Place. Every Celebration."** They are part of the Bowlero Corp family (alongside Bowlero and AMF brands), but documents should feature only the Lucky Strike Entertainment identity unless told otherwise.

## Color Palette

These are the exact colors to use. The primary palette covers most needs; the secondary palette is for supporting elements.

### Primary

| Name | Hex | RGB | Usage |
|------|-----|-----|-------|
| Lucky Strike Red | `#DB3434` | 219, 52, 52 | Primary accent — headings, buttons, highlights, chart emphasis |
| Charcoal | `#171717` | 23, 23, 23 | Primary dark backgrounds, header bars, dark text on light |
| White | `#FFFFFF` | 255, 255, 255 | Text on dark backgrounds, card/page backgrounds |

### Secondary

| Name | Hex | RGB | Usage |
|------|-----|-----|-------|
| Deep Red | `#BD2D2D` | 189, 45, 45 | Hover states, secondary accent, dark-mode highlights |
| Light Gray | `#F2F2F2` | 242, 242, 242 | Section backgrounds, table alternating rows, subtle dividers |
| Medium Gray | `#727272` | 114, 114, 114 | Secondary body text, captions, footnotes |
| Dark Gray | `#2B2B2B` | 43, 43, 43 | Body text on light backgrounds |

### Color usage principles

- Red is the star. Use it for the most important element on each page — a heading, a key metric, or a call-to-action — but don't overdo it. If everything is red, nothing stands out.
- Charcoal and white are the workhorses. Most text, backgrounds, and structural elements use these.
- Light Gray provides breathing room. Use it for alternating table rows, background sections, or sidebars.
- In charts and data visualizations, lead with Lucky Strike Red for the primary data series, then use Charcoal, Medium Gray, and Deep Red for secondary series.

## Typography

| Element | Font | Weight | Size guidance |
|---------|------|--------|---------------|
| Headings (H1) | Wix Madefor Text | 600 (SemiBold) | 24–32pt in docs, 36–44pt in slides |
| Headings (H2) | Wix Madefor Text | 600 (SemiBold) | 18–22pt in docs, 28–32pt in slides |
| Headings (H3) | Wix Madefor Text | 600 (SemiBold) | 14–16pt in docs, 22–26pt in slides |
| Body text | Wix Madefor Text | 400 (Regular) | 11–12pt in docs, 16–18pt in slides |
| Captions / fine print | Wix Madefor Text | 400 (Regular) | 9–10pt in docs, 12–14pt in slides |

**Fallback fonts:** If Wix Madefor Text is not available in the output environment, use **Inter** as first fallback or **Arial** as second fallback. These share a similar clean, modern sans-serif feel.

## Logo

The Lucky Strike logo is the wordmark "LUCKY ✕ STRIKE" (with a decorative X between the words). It comes in two variants:

- **White version** — for use on dark (Charcoal) backgrounds
- **Dark version** — for use on white or light backgrounds

### Logo URL

The official logo is available at:
```
https://www.luckystrikeent.com/assets/lucky-strike-entertainment-logo
```

**For HTML reports/dashboards:** Always use this URL directly in an `<img>` tag. Do NOT download, base64-encode, inline as SVG, or otherwise convert the logo — reference the URL as-is. Example:
```html
<img src="https://www.luckystrikeent.com/assets/lucky-strike-entertainment-logo" alt="Lucky Strike Entertainment" />
```

**For Office formats (docx, pptx, xlsx) and PDF:** If the logo image cannot be embedded at generation time, use the styled text fallback: render "LUCKY ✕ STRIKE" in Wix Madefor Text, SemiBold, with appropriate sizing, colored white-on-charcoal or charcoal-on-white as needed.

### Logo placement rules

- **Cover pages / title slides:** Centered or top-left, with generous spacing around it
- **Headers:** Top-left corner, small (roughly 1.5 inches wide in docs, scaled proportionally in slides)
- **Footers:** Optional — if present, keep it small and paired with the page number
- Do not stretch, recolor, or rotate the logo. Maintain its original aspect ratio.

## Document Structure by Format

### Word Documents (.docx)

Read the **docx** skill for file creation mechanics. Apply these brand elements:

- **Cover page:** Charcoal background with the logo text centered in white, document title in Lucky Strike Red below it, date and author in white or Light Gray at the bottom
- **Headers:** Thin Lucky Strike Red horizontal rule under the header area. Logo text left-aligned, document title right-aligned, both in 9pt Medium Gray
- **Footers:** Page number centered in Medium Gray, optional "Lucky Strike Entertainment — Confidential" text
- **Heading 1:** Lucky Strike Red, 24pt SemiBold, with a 2pt Lucky Strike Red bottom border
- **Heading 2:** Charcoal, 18pt SemiBold
- **Heading 3:** Dark Gray, 14pt SemiBold
- **Body text:** Dark Gray, 11pt Regular
- **Tables:** Charcoal header row with white text, alternating white and Light Gray body rows, Lucky Strike Red for any key/highlight values
- **Margins:** 1 inch all sides (standard)

### PowerPoint Decks (.pptx)

Read the **pptx** skill for file creation mechanics. Apply these brand elements:

- **Title slide:** Full Charcoal background, logo text centered top-third, presentation title large in white, subtitle in Lucky Strike Red, date/presenter in Light Gray at bottom
- **Section divider slides:** Lucky Strike Red background, section title in white, centered
- **Content slides:** White background, slide title in Charcoal (SemiBold), body text in Dark Gray. A thin Lucky Strike Red accent line under the title area
- **Charts / data slides:** Use the brand color sequence for data series: Red → Charcoal → Medium Gray → Deep Red → Light Gray
- **Footer bar:** Thin Charcoal strip at the very bottom with slide number in white on the right
- **Images:** Use rounded corners where possible to match the brand's rounded button aesthetic

### Excel Spreadsheets (.xlsx)

Read the **xlsx** skill for file creation mechanics. Apply these brand elements:

- **Title row:** Charcoal background, white text, SemiBold, merged across relevant columns
- **Header row:** Lucky Strike Red background, white text, SemiBold
- **Alternating rows:** White and Light Gray
- **Key metrics / totals row:** Charcoal background, white text, SemiBold
- **Accent values:** Lucky Strike Red text for values that need to stand out (e.g., growth percentages, targets met)
- **Borders:** Thin Light Gray borders between cells; avoid heavy black grid lines
- **Charts:** Follow the same color sequence as slides (Red → Charcoal → Medium Gray → Deep Red)
- **Tab names:** Clear, short, professional (e.g., "Summary", "Revenue", "Q1 Detail")

### HTML Reports & Dashboards

HTML is great for reports that need to be viewed in a browser, shared via email, or embedded somewhere. Since there's no separate HTML skill needed, everything you need is here.

**Page setup:**
```html
<link href="https://fonts.googleapis.com/css2?family=Wix+Madefor+Text:wght@400;600&display=swap" rel="stylesheet">
```

Apply these brand elements:

- **Page wrapper:** `max-width: 960px; margin: 0 auto; font-family: 'Wix Madefor Text', Inter, Arial, sans-serif; color: #2B2B2B;`
- **Header bar:** Full-width Charcoal (`#171717`) bar with the logo as an `<img>` tag sourced directly from `https://www.luckystrikeent.com/assets/lucky-strike-entertainment-logo` (do NOT base64-encode or recreate in code), left-aligned, height ~40px, and optionally the report title right-aligned in Light Gray
- **H1:** Lucky Strike Red (`#DB3434`), font-weight 600, with a 2px `#DB3434` bottom border and padding below
- **H2:** Charcoal (`#171717`), font-weight 600
- **H3:** Dark Gray (`#2B2B2B`), font-weight 600
- **Body text:** Dark Gray (`#2B2B2B`), font-weight 400, line-height 1.6
- **Links:** Lucky Strike Red, no underline by default, underline on hover
- **Tables:** Charcoal header row with white text; alternating `#FFFFFF` and `#F2F2F2` body rows; thin `#F2F2F2` borders; `border-collapse: collapse`
- **Cards / callout boxes:** White background, `1px solid #F2F2F2` border, `border-radius: 12px` (matching the brand's rounded aesthetic), subtle `box-shadow`, with a `4px solid #DB3434` left border for emphasis cards
- **Buttons / CTAs:** Lucky Strike Red background, white text, `border-radius: 100px` (pill shape), padding `12px 28px`, font-weight 600
- **Key metrics / KPI blocks:** Large number in Lucky Strike Red, label in Medium Gray below, arranged in a flex row
- **Footer:** Light Gray background, centered text in Medium Gray, copyright line "© 2026 Lucky Strike Entertainment"
- **Responsive:** Use CSS grid or flexbox; ensure it reads well from 600px to 1200px wide

**CSS variable block** (include at the top of every HTML report's `<style>` section for easy reference):
```css
:root {
  --lse-red: #DB3434;
  --lse-deep-red: #BD2D2D;
  --lse-charcoal: #171717;
  --lse-dark-gray: #2B2B2B;
  --lse-medium-gray: #727272;
  --lse-light-gray: #F2F2F2;
  --lse-white: #FFFFFF;
  --lse-font: 'Wix Madefor Text', Inter, Arial, sans-serif;
  --lse-radius: 12px;
  --lse-radius-pill: 100px;
}
```

### PDFs

Read the **pdf** skill for file creation mechanics. Follow the same visual rules as Word documents above — cover page, headers, footers, heading styles, and table formatting all carry over directly.

## Brand Voice (for text content)

When generating text content within documents (e.g., executive summaries, descriptions, section intros):

- **Tone:** Confident, energetic, approachable — not corporate-stuffy. Think "premium fun."
- **Perspective:** First-person plural ("we", "our") when speaking as Lucky Strike Entertainment
- **Keep it concise:** Short sentences, active voice, clear structure
- Avoid jargon unless it's industry-standard for the audience
- The tagline "One Place. Every Celebration." can appear on cover pages or closing slides but don't force it everywhere
