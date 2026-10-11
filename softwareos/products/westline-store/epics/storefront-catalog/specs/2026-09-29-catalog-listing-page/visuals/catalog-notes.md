# Catalog listing: design notes

These are extracted from `westline-site-desing/catalog.html`. All of its CSS is in one inline `<style>` block. Base64 image data and long SVG paths are stripped, and the header, mega-menu and footer are covered by site-nav.

## Design → implementation

| Design | Implementation |
|---|---|
| `.container` max 1240px, padding 24px (16px ≤480px) | `mx-auto max-w-[1240px] px-6 max-[480px]:px-4` |
| `.cat-header` padding 28/20px, bottom border | header section |
| `.breadcrumb` JetBrains Mono 12px uppercase .04em muted, `/` separator at .5 opacity, `.current` ink | breadcrumb, all items except the last are links |
| `.cat-header h1` Poppins 800 uppercase, clamp(30px,4.5vw,52px), line-height .95 | H1 = category name or "All Products" |
| `a.finder-callout` next to the H1 | **hidden** |
| `.catalog` padding 36/80px; `.catalog-grid` 248px + 1fr, gap 40px | same |
| `.catalog-toolbar` bottom border, padding-bottom 20px, margin-bottom 24px; `.result-count` 14px muted "34 results" | "<N> results" |
| `.mobile-filter-btn` (≤900px), `.sort-select` | **hidden** |
| sidebar at ≤900px = off-canvas drawer opened by Filter | **hidden at ≤900px** (no drawer); the site-nav hamburger covers mobile browsing |
| `.sidebar-title` "Categories", Mono 12px .08em muted | same |
| `.cat-group` > `button.cat-parent` 14.5px/700 + `.chev` 14px (rotates 180° when open; parent turns horizon when open) | accordion group per Strapi category |
| `.cat-children a` 13.5px muted, padding-left 14px, 2px transparent left border; `.is-active` horizon, 600, horizon border | links; active gets `aria-current="page"` (an a11y addition) |
| Clothing: "Shop All Clothing" + `.cat-subgroup` Men / Women (`button.cat-subparent` 13.5px/700, `.chev-sm` 12px; `.cat-subchildren a` 13px, padding-left 28px) with "All Men" first | same, gender subgroups from `buildNavMenu` columns |
| Wetsuits: flat All / Men / Women links | same (NavLabel) |
| "Filter by" `.sidebar-block` (Skill Level, Length, Volume, Price range, Clear all) | **hidden**; Categories becomes the last block (no bottom border) |
| `.cat-grid-products` 4 cols, gap 20px; 3 cols ≤1080px; 2 cols gap 14px ≤760px | same |
| `.prod-card` flex column (price aligned to the bottom) | shared `ProductCard` |
| `.prod-wish` heart | **hidden** (no favorite props) |
| `.pagination` centered, gap 8px, margin-top 48px; 36×36 buttons, 1px border, radius 9px, 13px/600; current = ink fill, white text; Next chevron only | `next/link`s; Prev chevron added (not rendered on page 1), Next not rendered on the last page; hidden when there is 1 page |
| no empty state | "No products here yet" + "All <Category>" link (decided in shaping) |

## CSS

```css
.breadcrumb{display:flex; gap:8px; align-items:center; font-family:'JetBrains Mono',monospace; font-size:12px; color:var(--muted); text-transform:uppercase; letter-spacing:.04em;}
.breadcrumb a{text-decoration:none; color:var(--muted);} .breadcrumb a:hover{color:var(--horizon);}
.breadcrumb .sep{opacity:.5;} .breadcrumb .current{color:var(--ink);}
.cat-header{padding-block:28px 20px; border-bottom:1px solid var(--border);}
.cat-header-row{display:flex; align-items:flex-end; justify-content:space-between; gap:24px; flex-wrap:wrap; margin-top:10px;}
.cat-header h1{font-size:clamp(30px,4.5vw,52px);}
.display{font-family:'Poppins',sans-serif; font-weight:800; text-transform:uppercase; letter-spacing:0.01em; line-height:0.95; text-wrap:balance; margin:0;}
.catalog{padding-block:36px 80px;}
.catalog-grid{display:grid; grid-template-columns:248px 1fr; gap:40px; align-items:start;}
.catalog-toolbar{display:flex; align-items:center; justify-content:space-between; gap:16px; flex-wrap:wrap; padding-bottom:20px; margin-bottom:24px; border-bottom:1px solid var(--border);}
.result-count{font-size:14px; color:var(--muted);}
.sidebar-title{font-family:'JetBrains Mono',monospace; font-size:12px; text-transform:uppercase; letter-spacing:.08em; color:var(--muted); margin:0 0 14px;}
.cat-tree{list-style:none; margin:0; padding:0;} .cat-group{margin-bottom:2px;}
.cat-parent{width:100%; display:flex; align-items:center; justify-content:space-between; background:none; border:none; padding:10px 0; font-size:14.5px; font-weight:700; color:var(--ink); text-align:left;}
.cat-parent .chev{width:14px; height:14px; stroke:var(--muted); fill:none; stroke-width:2; transition:transform .2s;}
.cat-group.is-open .cat-parent .chev{transform:rotate(180deg);} .cat-group.is-open .cat-parent{color:var(--horizon);}
.cat-children{list-style:none; margin:0 0 10px; padding:0 0 0 2px;} .cat-children li{margin-bottom:2px;}
.cat-children a{display:block; padding:6px 0 6px 14px; font-size:13.5px; color:var(--muted); text-decoration:none; border-left:2px solid transparent;}
.cat-children a:hover{color:var(--ink);}
.cat-children a.is-active{color:var(--horizon); border-left-color:var(--horizon); font-weight:600;}
.cat-subparent{width:100%; display:flex; align-items:center; justify-content:space-between; background:none; border:none; padding:6px 0 6px 14px; font-size:13.5px; font-weight:700; color:var(--ink); text-align:left;}
.cat-subparent .chev-sm{width:12px; height:12px; stroke:var(--muted); fill:none; stroke-width:2; transition:transform .2s; flex-shrink:0;}
.cat-subgroup.is-open .cat-subparent .chev-sm{transform:rotate(180deg);} .cat-subgroup.is-open .cat-subparent{color:var(--horizon);}
.cat-subchildren{list-style:none; margin:0 0 6px; padding:0;}
.cat-subchildren a{display:block; padding:6px 0 6px 28px; font-size:13px; color:var(--muted); text-decoration:none; border-left:2px solid transparent;}
.cat-subchildren a.is-active{color:var(--horizon); border-left-color:var(--horizon); font-weight:600;}
.cat-grid-products{display:grid; grid-template-columns:repeat(4,1fr); gap:20px;}
@media (max-width:1080px){ .cat-grid-products{grid-template-columns:repeat(3,1fr);} }
@media (max-width:760px){ .cat-grid-products{grid-template-columns:1fr 1fr; gap:14px;} }
.pagination{display:flex; justify-content:center; align-items:center; gap:8px; margin-top:48px;}
.pagination button{min-width:36px; height:36px; border:1px solid var(--border); background:var(--white); border-radius:var(--radius); font-size:13px; font-weight:600; color:var(--ink);}
.pagination button[aria-current="true"]{background:var(--ink); border-color:var(--ink); color:#fff;}
.pagination button:hover:not([aria-current="true"]){border-color:var(--ink);}
@media (max-width:900px){ .catalog-grid{grid-template-columns:1fr;} /* sidebar → drawer (not built) */ }
:root{--ink:#101828; --white:#FFFFFF; --surface-2:#F9F9F9; --muted:#667085; --border:#D0D5DD; --horizon:#155EEF; --horizon-deep:#0E40A3; --radius:9px; --maxw:1240px;}
```

## Markup

```html
<section class="cat-header"><div class="container">
  <nav class="breadcrumb" aria-label="Breadcrumb"><a href="index.html">Home</a><span class="sep">/</span><span class="current">Surfboards</span></nav>
  <div class="cat-header-row"><h1 class="display">Surfboards</h1><a class="finder-callout">…</a></div>
</div></section>
<section class="catalog"><div class="container catalog-grid">
  <aside class="catalog-sidebar">
    <div class="sidebar-block"><h2 class="sidebar-title">Categories</h2>
      <ul class="cat-tree">
        <li class="cat-group is-open"><button class="cat-parent" aria-expanded="true">Surfboards <svg class="chev"><path d="M6 9l6 6 6-6"/></svg></button>
          <ul class="cat-children"><li><a class="is-active">All Surfboards</a></li><li><a>Performance Shortboard</a></li>…</ul></li>
        <li class="cat-group"><button class="cat-parent" aria-expanded="false">Clothing …</button>
          <ul class="cat-children" hidden><li><a>Shop All Clothing</a></li>
            <li class="cat-subgroup"><button class="cat-subparent" aria-expanded="false">Men <svg class="chev-sm">…</svg></button>
              <ul class="cat-subchildren" hidden><li><a>All Men</a></li><li><a>T-Shirts &amp; Tanks</a></li>…</ul></li>
            …Women…</ul></li>
      </ul></div>
    <div class="sidebar-block"><!-- Filter by: hidden --></div>
  </aside>
  <div class="catalog-main">
    <div class="catalog-toolbar"><span class="result-count">34 results</span><div class="toolbar-actions"><!-- Filter btn + sort: hidden --></div></div>
    <div class="cat-grid-products">….prod-card…</div>
    <nav class="pagination" aria-label="Pagination"><button aria-current="true">1</button><button>2</button><button>3</button><button aria-label="Next page"><svg><path d="M9 6l6 6-6 6"/></svg></button></nav>
  </div>
</div></section>
```

## Behavior (JS)

- Group toggles flip `.is-open`, `aria-expanded` and the `hidden` attribute on the child list. Groups are independent (opening one doesn't close the others), and sub-toggles `stopPropagation()`. On load only the active group is open.
- The drawer JS (`#openFilters`, backdrop, `body{overflow:hidden}`) isn't built.
