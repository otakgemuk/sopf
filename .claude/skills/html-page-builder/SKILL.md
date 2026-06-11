---
name: html-page-builder
description: "Builds or updates HTML pages for the SOPF static site following the canonical page structure. Use when creating a new firm page, updating an existing firm page, editing promo boxes, updating account tables, or making structural changes to any page in otakgemuk/sopf. Triggers on: 'build firm page', 'update html', 'add account row', 'update promo box', 'new page for {firm}', 'fix the html for {firm}'."
---

# HTML Page Builder

Builds and updates SOPF static site HTML pages to the canonical standard.

## Canonical Page Structure

All firm pages follow this exact structure. Reference: prop-firms/alpha-futures.html in otakgemuk/sopf.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <!-- charset, viewport, title, meta description, theme-color -->
  <!-- Google Fonts: DM Sans + DM Mono -->
  <link rel="stylesheet" href="../shared.css">
</head>
<body>
  <nav class="nav"> <!-- standard 4-link nav + code pill + hamburger -->
  <style> <!-- page-specific styles only — never duplicate shared.css vars -->
  <div class="breadcrumb"> <!-- Home > Prop Firms > {FirmName} -->
  <div class="firm-hero"> <!-- logo img + firm-meta (h1 + p) -->
  <div class="firm-body"> <!-- 2-col grid: firm-info + firm-sidebar -->
    <div class="firm-info">
      <div class="info-card"> <!-- Firm Overview — 10 info-rows -->
      <div class="info-card"> <!-- Account Options — acct-table -->
    </div>
    <div class="firm-sidebar">
      <div class="promo-box"> <!-- best offer + code + btn-claim -->
      <div class="side-card"> <!-- Platforms -->
      <div class="side-card"> <!-- Compare All Firms -->
      <div class="side-card"> <!-- All Promos -->
    </div>
  </div>
  <div class="newsletter">
  <div class="disclaimer">
  <footer class="footer">
</body>
```

## CSS Variable Usage
Always use shared.css variables. Never hardcode colours.
```css
/* Correct */
color: var(--text-primary);
background: var(--surface-card);
border: .5px solid var(--border-default);

/* Wrong */
color: #ffffff;
background: #1a1a2e;
```

## Account Table Structure
```html
<table class="acct-table">
  <thead>
    <tr>
      <th>Account</th><th>Size</th><th>Contracts</th>
      <th>Price (MOT)</th><th>Target</th><th>Drawdown</th>
      <th>P/D</th><th>Daily Loss</th><th>Days to Pass</th>
      <th>Days to $</th><th>Action</th>
    </tr>
  </thead>
  <tbody>
    <!-- one <tr> per account size -->
    <!-- btn-buy links to affiliate URL with target="_blank" -->
  </tbody>
</table>
```

## Promo Box Structure
```html
<div class="promo-box">
  <h3>Best Current Offer</h3>
  <div class="promo-off">{X}% OFF</div>
  <div class="promo-exp">Expires: {date or "While it lasts"}</div>
  <div class="promo-code">{CODE}</div>
  <a class="btn-claim" href="{affiliate_url}" target="_blank">Claim Deal &#8599;</a>
</div>
```

## File Naming
```
prop-firms/myfundedfutures.html
prop-firms/nexgen-protrader-funding.html
prop-firms/apex-trader-funding.html
```
Slugify: lowercase, hyphens, no special characters.

## Update vs New
- **New page**: use alpha-futures.html as template, replace all firm-specific content
- **Update**: surgical edits only — identify the specific block that changed, update only that block
- **Promo-only**: update only the .promo-box div and the btn-buy hrefs

## Quality Rules
- No broken relative paths — always `../shared.css`, `../index.html`, `../promos/index.html`
- All links to affiliate URLs must be absolute with https://
- No smart quotes in HTML attributes or visible text
- img onerror fallback: `onerror="this.src=''"` on all firm logos
