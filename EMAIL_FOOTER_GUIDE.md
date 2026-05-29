# Email Footer — Production Guide

> Scope: `email_footer.html`  
> Client: K2 Drukarnia Projektów  
> Last audit: 2026-05-29 — **PASS, no issues**

---

## 1. Context & Goals

The email footer must render correctly in:
- Desktop Outlook (Word engine, 2007–2019) — highest priority
- Gmail web and mobile app
- Apple Mail (macOS + iOS)
- Yahoo Mail
- Outlook.com (web)
- Thunderbird (desktop)

All images must be **embedded as base64 data URIs** — no remote URLs. Remote content is blocked by default in many corporate email clients (especially older Outlooks) and causes missing images for recipients.

---

## 2. Issues Found in the Original Template

| Issue | Impact |
|---|---|
| `<style>` block in `<head>` | Stripped by Gmail, Yahoo — styles lost |
| `max-width` + `margin: auto` on outer table only | Outlook ignores CSS — layout stretched to full window width |
| `border-bottom`/`border-top` on `<table>` elements | Outlook renders borders on tables unreliably |
| `margin-bottom` on `<table>` elements | Ignored by Outlook Word engine |
| Remote image URLs (`.webp`, `.png`, etc.) | Blocked in Outlook, Thunderbird, corporate environments |
| `border-radius` on avatar and icon `<td>` wrappers | Ignored by Outlook — circles don't appear |
| Missing HTML `width`/`height` on `<img>` tags | Outlook ignores CSS dimensions — images render at native size |
| `text-decoration-color` on link styles | Not universally supported |
| No `role="presentation"` on layout tables | Accessibility and rendering semantics |
| No `u+.body` Gmail fallback selectors | Mobile stacking unreliable in Gmail app |
| No mobile padding reduction | 40px padding too wide on small screens |

---

## 3. Structural Architecture

### 3.1 Dual-track width system

Outlook and modern clients each need their own width mechanism:

```html
<!--[if mso]>
<table align="center" border="0" cellpadding="0" cellspacing="0" width="800">
<tr><td>
<![endif]-->
<table role="presentation" border="0" cellpadding="0" cellspacing="0" width="100%"
    style="background-color: #ffffff; max-width: 800px; margin: 0 auto;">
  ...
</table>
<!--[if mso]>
</td></tr></table>
<![endif]-->
```

- `<!--[if mso]>` is only parsed by Outlook — ignored by all other clients as an HTML comment.
- Modern clients use CSS `max-width: 800px; margin: 0 auto` to cap and center.
- Both paths produce the same 800px centered card.

### 3.2 Style block placement

```html
<body class="body" style="...">
  <style type="text/css">
    /* ALL styles go here, NOT in <head> */
  </style>
  ...
</body>
```

- Gmail and Yahoo strip `<style>` from `<head>`. Placing it as the **first child of `<body>`** ensures it survives in all clients.
- The `class="body"` on `<body>` is required for the `u+.body` Gmail targeting selector.

### 3.3 Layout via nested tables

Every layout section is a `<tr>` row of the outer table. Inner content uses nested `<table>` elements with:

```html
<table role="presentation" border="0" cellpadding="0" cellspacing="0">
```

- `role="presentation"` — marks tables as layout-only for screen readers.
- `border="0" cellpadding="0" cellspacing="0"` — removes default browser/Outlook spacing.
- Never use `margin` on `<table>` — use `padding` on `<td>` instead.

### 3.4 Dividers

Replace `border-bottom`/`border-top` on tables with dedicated divider rows:

```html
<tr>
  <td style="border-bottom: 1px solid #e2e2e2; height: 1px; font-size: 0; line-height: 0;">&nbsp;</td>
</tr>
```

- `height: 1px; font-size: 0; line-height: 0` — collapses the row to exactly 1px.
- `&nbsp;` — prevents some clients from collapsing empty cells.
- `border` on `<td>` is safe in Outlook. `border` on `<table>` is not.

---

## 4. Outlook Compatibility Rules

| Rule | Reason |
|---|---|
| Use `<!--[if mso]>` conditional comments for Outlook-specific layout | Word engine ignores unknown CSS |
| Never use `max-width` alone — always pair with MSO wrapper `width="NNN"` | `max-width` is ignored by Word engine |
| Never use `margin` on `<table>` | Ignored by Word engine |
| Use `padding` on `<td>` for spacing | Fully supported |
| Never use `border-radius` | Not rendered in Outlook |
| Always set HTML `width` and `height` attributes on `<img>` | Word engine ignores CSS dimensions |
| Use `border="0"` on all `<img>` | Prevents default link borders |
| Use `display: block` on all `<img>` | Removes phantom whitespace below images |
| Fonts: stick to `Arial, Helvetica, sans-serif` or other web-safe stacks | Custom fonts require VML fallbacks |
| `border-left`/`border-right`/`border-bottom`/`border-top` on `<td>` are safe | Preferred over `border` on `<table>` |

---

## 5. Responsive Design

### 5.1 Media queries

```css
@media only screen and (max-width: 600px) {
    .outer-cell {
        padding-left: 20px !important;
        padding-right: 20px !important;
    }
    .mobile-col {
        display: block !important;
        width: 100% !important;
        max-width: 100% !important;
        padding-right: 0 !important;
        padding-left: 0 !important;
        padding-bottom: 15px !important;
        border: none !important;
        box-sizing: border-box !important;
    }
    .mobile-col-last {
        display: block !important;
        width: 100% !important;
        max-width: 100% !important;
        padding-right: 0 !important;
        padding-left: 0 !important;
        padding-bottom: 0 !important;
        border: none !important;
        box-sizing: border-box !important;
    }
}
```

- `.outer-cell` — reduces side padding on small screens.
- `.mobile-col` — stacks `<td>` columns vertically and clears horizontal spacing and borders.
- `.mobile-col-last` — same as `.mobile-col` but no bottom padding (last in stack).
- `!important` — required to override inline styles on `<td>` elements.

### 5.2 Gmail app fallback (`u+.body` selectors)

Gmail's mobile app injects a `<u></u>` before the `<body>` tag in its DOM, creating a unique CSS ancestry:

```css
u + .body .mobile-col {
    display: block !important;
    /* ...same rules as @media block... */
}
u + .body .mobile-col-last {
    display: block !important;
    /* ...same rules as @media block... */
}
```

These selectors are **outside** the `@media` block — they fire unconditionally in the Gmail app (which always renders at mobile width in this context). This ensures stacking works in Gmail even if `@media` is not triggered.

---

## 6. Image Standards

### 6.1 Embedding

All images must be embedded as base64 data URIs:

```html
<img src="data:image/jpeg;base64,/9j/4AAQ..." ...>
```

- Eliminates dependency on remote servers.
- Works in corporate environments with remote content blocked.
- Works offline and in email clients that never fetch remote content.
- **Supported formats:** JPEG and PNG are the safest. Avoid WebP — not supported in older Outlook.

### 6.2 Dimension attributes — the dual requirement

Every `<img>` must have **both** HTML attributes (for Outlook) and CSS dimensions (for modern clients):

```html
<!-- Fixed-size images (icons, avatar): pin both axes -->
<img src="data:..." width="26" height="26"
     style="display: block; width: 26px; height: 26px; border: 0;" alt="Phone">

<!-- Non-square images with known aspect ratio: anchor one axis, auto the other -->
<img src="data:..." width="95" height="38"
     style="display: block; height: 38px; width: auto; border: 0;" alt="Logo">
```

- **Outlook** uses the HTML `width`/`height` attributes exclusively.
- **Modern clients** use the CSS `width`/`height` properties.
- For **square images**: set both CSS axes to the same fixed value — no aspect ratio concern.
- For **non-square images**: set one CSS axis to the display size and the other to `auto` to preserve the natural aspect ratio. Setting both CSS axes to explicit values risks distortion if the HTML attribute values don't exactly match the image's intrinsic proportions. The HTML attrs still tell Outlook the intended size.

### 6.3 Always include

```html
display: block;   /* removes phantom whitespace below inline images */
border: 0;        /* removes default blue border when image is inside a link */
alt="..."         /* required for accessibility and fallback text */
```

---

## 7. CSS Rules & Hygiene

| Property | Rule |
|---|---|
| `border-radius` | **Never use** — invisible in Outlook. Pre-bake circles into image files. |
| `text-decoration-color` | **Never use** — not universally supported. |
| `margin` on `<table>` | **Never use** — ignored by Outlook Word engine. Use `padding` on `<td>`. |
| `max-width` alone | **Never use alone** — always pair with MSO conditional `width="NNN"`. |
| `position`, `float`, `z-index` | **Avoid** — not reliably supported in email clients. |
| `padding` on `<td>` | **Preferred** spacing method — works everywhere. |
| `font-family` | Always specify full stack: `Arial, Helvetica, sans-serif` |
| `font-size`, `color`, `font-weight` | Always inline on text elements — never rely solely on inherited styles. |
| Link `color` | Always set explicitly inline — email clients override default link colors. |

---

## 8. Link Standards

```html
<a href="tel:+48XXXXXXXXX"      style="color: #555555; text-decoration: underline; ...">
<a href="mailto:user@domain.pl" style="color: #555555; text-decoration: underline; ...">
<a href="https://www.domain.pl" style="color: #555555; text-decoration: underline; ...">
```

- Use `tel:` for phone numbers — triggers the dialer on mobile.
- Use `mailto:` for email — triggers the default mail client.
- Always use `https://` for website links.
- `text-decoration: underline` — explicitly set to prevent variation across clients.
- Do **not** use `text-decoration-color` — use `color` for the link itself.

---

## 9. Client-by-Client Compatibility Summary

### Outlook 2007–2019 (Word engine)
- No `@media` support — mobile layout does not stack
- Relies on: MSO conditional wrapper, HTML `width`/`height` on images, `padding` on `<td>`, no `border-radius`
- Result: full-width desktop layout, all content legible

### Gmail Web
- `<style>` in `<body>` supported
- `@media` queries supported
- `u+.body` selectors for Gmail-specific DOM targeting
- Result: full responsive behavior

### Gmail App (iOS/Android)
- Supports `<style>` and `@media`
- `u+.body` selectors as a fallback safety net
- Result: columns stack, padding reduces correctly

### Apple Mail (macOS/iOS)
- Full WebKit/CSS support
- Result: perfect render

### Yahoo Mail
- Respects `<style>` in `<body>` and inline styles
- Result: correct render

### Outlook.com (web)
- Strips `<style>` from `<head>` — not an issue since we use `<body>`
- Partial `@media` support
- Result: correct render

### Thunderbird ≥78
- Full `@media` support
- Result: full responsive behavior

### Thunderbird <78
- No `@media` support
- Falls back to desktop table layout — still fully readable
- Result: acceptable

---

## 10. Production Checklist

Use this before every deployment:

- [ ] All image `src` values are `data:image/jpeg;base64,...` or `data:image/png;base64,...` (no remote URLs)
- [ ] All `<img>` have HTML `width` and `height` attributes
- [ ] All `<img>` have matching (or aspect-ratio-safe) CSS `width`/`height` in `style`
- [ ] All `<img>` have `display: block; border: 0; alt="..."`
- [ ] No `border-radius` anywhere
- [ ] No `text-decoration-color` anywhere
- [ ] No `margin` on any `<table>` element
- [ ] `<style>` block is inside `<body>`, not `<head>`
- [ ] `<body>` has `class="body"`
- [ ] MSO conditional wrapper is present and correctly closed
- [ ] All layout tables have `role="presentation" border="0" cellpadding="0" cellspacing="0"`
- [ ] Dividers use dedicated `<tr><td>` rows with `border-bottom`, not `border` on `<table>`
- [ ] `u+.body` selectors mirror the `@media` stacking rules
- [ ] All `href` values match their display text (no stale or mismatched URLs)
- [ ] Document has exactly one `</html>` closing tag
- [ ] HTML is valid (no duplicate tags, unclosed elements)

---

## 11. What NOT to Do (Common Mistakes)

1. **Putting `<style>` in `<head>`** — Gmail and Yahoo strip it. Always use `<body>`.
2. **Using remote image URLs** — blocked silently. Always embed as base64.
3. **Using WebP images** — not supported in Outlook. Use JPEG or PNG.
4. **Using `border-radius` for circles** — invisible in Outlook. Pre-bake circles into image files in an editor (Affinity, Photoshop, GIMP).
5. **Using `margin` on `<table>`** — ignored by Outlook. Use `<td>` padding.
6. **Relying on `max-width` alone for Outlook centering** — pair with MSO conditional comment.
7. **Using `position: absolute/relative` or `float`** — breaks layout in most email clients.
8. **Setting both explicit CSS `width` and `height` on non-square images** — can distort aspect ratio if values don't match the image's intrinsic proportions. Always use `auto` for the unconstrained axis.
9. **Forgetting `!important` in media query overrides** — inline styles have higher specificity and will win without it.
10. **Only writing `@media` without `u+.body` fallback** — Gmail app may not trigger `@media` as expected; the `u+.body` selector is the insurance.

---

*Guide generated from production refactor session — 2026-05-29*
