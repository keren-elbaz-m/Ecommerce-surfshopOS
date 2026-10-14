# Site nav: design notes

These are extracted from `westline-site-desing/index.html`. Base64 image data and long SVG paths are stripped.

## Design → implementation

| Design | Implementation |
|---|---|
| `.site-nav` fixed, transparent over the hero; `.solid` after 40px of scroll (white, border, 14px padding) | Always the **solid** look, `sticky top-0`, fixed at **64px** tall (the hero depends on it). The transparent variant is out of scope |
| `.word` Poppins 800 22px, 0.02em | wordmark → `/` |
| `.nav-links` 13.5px/600 uppercase 0.04em, gap 28px, hover horizon | `DesktopMenu` items from Strapi |
| `.mega-menu` fixed full width under the nav, white, border-top, shadow; opens on `.mega-open` (hover, 250ms close delay) or `:focus-within` | `absolute inset-x-0 top-full`, the same hover-intent, plus Escape to close (not in the design; an a11y addition) |
| `.mega-col` + `.mega-all` ("All Surfboards", bold + bottom border) | first column link "All <Category Name>" |
| `.mega-col-head` JetBrains Mono 11px uppercase muted ("Men" / "Women" under Clothing) | column heads for the Clothing group (linked to their category) |
| `.mega-promo` (Finder promo) | **hidden** |
| `.is-finder` "Surfboard Finder" link, `.mp-plain` Finder | **hidden** |
| `.nav-icons` Search / Account / Wishlist / Cart + `.cart-count` | Search and cart count **hidden**. Account keeps the existing auth behavior. Wishlist → `/account#wishlist`, Cart → `/cart` |
| `.nav-toggle` hamburger at ≤860px, `aria-expanded` | `MobileMenu` hamburger |
| `.mobile-panel` ink, full height, `.mp-group` rows + `.mp-toggle` chevron (`aria-expanded`, rotates 90°), `.mp-sub` links, `.mp-sub-head` | `MobileMenu` panel below the header; accordion per item; link click closes the panel |
| Kids (Wetsuits), Gear (Accessories) | not in Strapi, so not shown |
| Short labels "T-Shirts & Tanks", "Men", … | `Subcategory.NavLabel` (falls back to `Name`) |

## CSS

```css
  .site-nav{
    position:fixed; top:0; left:0; right:0; z-index:50;
    padding-block:18px; transition:background .25s ease, border-color .25s ease, padding .25s ease;
    border-bottom:1px solid transparent;
  }
  .site-nav .container{display:flex; align-items:center; justify-content:space-between; gap:20px;}
  .site-nav .word{
    font-family:'Poppins',sans-serif; font-weight:800; font-size:22px; letter-spacing:0.02em; color:#fff; text-decoration:none;
  }
  .nav-links{display:flex; gap:28px; list-style:none; margin:0; padding:0;}
  .nav-links a{
    font-size:13.5px; font-weight:600; text-decoration:none; color:#fff;
    text-transform:uppercase; letter-spacing:0.04em; opacity:.92;
  }
  .nav-links a.is-finder{color:#EAF4F7;}
  .nav-icons{display:flex; align-items:center; gap:16px;}
  .nav-icons button, .nav-icons a{background:none; border:none; padding:6px; color:#fff; display:flex; text-decoration:none;}
  .nav-icons .icon{width:20px; height:20px; stroke:currentColor; fill:none; stroke-width:1.7;}
  .cart-count{
    font-family:'JetBrains Mono',monospace; font-size:10px; background:var(--horizon); color:#fff;
    border-radius:50%; width:16px; height:16px; display:flex; align-items:center; justify-content:center;
    position:relative; left:-14px; top:-8px;
  }
  .nav-icons .nav-toggle{display:none; background:none; border:none; color:#fff;}
  .mobile-panel{
    display:none; position:fixed; top:0; left:0; right:0; z-index:49; background:var(--ink);
    padding:96px 24px 24px; flex-direction:column; gap:20px;
  }
  .mobile-panel.open{display:flex;}
  .mobile-panel a{color:#fff; text-decoration:none; font-size:16px; text-transform:uppercase; letter-spacing:.04em; font-weight:600;}

  .site-nav.solid{ background:var(--white); border-color:var(--border); padding-block:14px; }
  .site-nav.solid .word, .site-nav.solid .nav-links a, .site-nav.solid .nav-icons button, .site-nav.solid .nav-icons a{ color:var(--ink); }

  @media (max-width:860px){
    .nav-links{display:none;}
    .nav-icons .nav-toggle{display:flex; padding:6px;}
  }


  /* Mega-menu nav */
  .nav-links a:hover{color:var(--horizon);}
  .nav-item{position:relative;}
  .mega-menu{
    display:none; position:fixed; left:0; right:0; top:var(--nav-h, 84px);
    background:var(--white); border-top:1px solid var(--border);
    box-shadow:0 20px 40px rgba(18,33,42,.14); z-index:60;
  }
  .nav-item.mega-open .mega-menu, .nav-item:focus-within .mega-menu{display:block;}
  .mega-menu-inner{max-width:var(--maxw); margin-inline:auto; padding:36px 24px 40px; display:flex; gap:56px;}
  .mega-col{display:flex; flex-direction:column; gap:2px; min-width:180px;}
  .mega-col-head{
    font-family:'JetBrains Mono',monospace; font-size:11px; text-transform:uppercase;
    letter-spacing:.1em; color:var(--muted); margin:14px 0 4px;
  }
  .mega-col-head:first-child{margin-top:0;}
  .mega-col a{color:var(--ink); text-decoration:none; font-size:14.5px; padding:7px 0; text-transform:none; letter-spacing:0; font-weight:500;}
  .mega-col a:hover{color:var(--horizon);}
  .mega-col a.mega-all{font-weight:700; border-bottom:1px solid var(--border); padding-bottom:12px; margin-bottom:6px;}
  .mega-promo{
    margin-left:auto; flex:0 0 260px; background:var(--surface-2); border-radius:var(--radius);
    padding:24px; display:flex; flex-direction:column; gap:8px;
  }
  .mega-promo .eyebrow{color:var(--horizon); display:block;}
  .mega-promo h4{font-family:'Poppins',sans-serif; font-weight:800; text-transform:uppercase; font-size:20px; margin:0; letter-spacing:.01em; color:var(--ink);}
  .mega-promo p{font-size:13px; color:var(--muted); margin:0; line-height:1.5;}
  .mega-promo .btn{margin-top:6px; align-self:flex-start; padding:10px 18px; font-size:12.5px;}

  /* Mobile nav accordion */
  .mobile-panel{overflow-y:auto; gap:0; bottom:0;}
  .mp-group{border-bottom:1px solid rgba(255,255,255,.14); padding:4px 0;}
  .mp-row{display:flex; align-items:center; justify-content:space-between; gap:12px;}
  .mp-row > a{flex:1; padding:14px 0;}
  .mp-toggle{background:none; border:none; color:#fff; padding:14px 4px; display:flex; align-items:center; cursor:pointer;}
  .mp-chevron{display:inline-block; transition:transform .2s ease; font-size:20px; line-height:1;}
  .mp-toggle[aria-expanded="true"] .mp-chevron{transform:rotate(90deg);}
  .mp-sub{display:none; flex-direction:column; gap:0; padding:0 0 14px 14px;}
  .mp-sub.open{display:flex;}
  .mp-sub a{font-size:14px; font-weight:500; text-transform:none; letter-spacing:0; opacity:.85; padding:9px 0;}
  .mp-sub-head{font-family:'JetBrains Mono',monospace; font-size:11px; text-transform:uppercase; letter-spacing:.1em; opacity:.55; margin:10px 0 2px;}
  .mp-sub-head:first-child{margin-top:4px;}
  .mp-plain{display:block; padding:14px 0;}
```

## Markup (desktop nav + first mega-menu, icons, first mobile group)

```html
<header class="site-nav" id="siteNav">
  <div class="container">
    <a class="word" href="#">WESTLINE</a>
    <ul class="nav-links">
      <li class="nav-item">
        <a href="catalog.html">Surfboards</a>
        <div class="mega-menu">
          <div class="mega-menu-inner container">
            <div class="mega-col">
              <a class="mega-all" href="catalog.html">All Surfboards</a>
              <a href="catalog.html">Performance Shortboard</a>
              <a href="catalog.html">Longboard</a>
              <a href="catalog.html">Fish &amp; Twinfin</a>
              <a href="catalog.html">Fun Board</a>
              <a href="catalog.html">Soft-top / Beginner</a>
            </div>
            <div class="mega-promo">
              <span class="eyebrow">Not sure where to start?</span>
              <h4>Take the 2-Minute Finder</h4>
              <p>Answer five quick questions and we'll match you to the right board.</p>
              <a class="btn btn-primary" href="#finder">Start the Finder</a>
            </div>
          </div>
        </div>
      </li>
      <li class="nav-item">
        <a href="#">Accessories</a>
    <div class="nav-icons">
      <button aria-label="Search"><svg class="icon" viewBox="0 0 24 24"><circle cx="11" cy="11" r="7"/><line x1="21" y1="21" x2="16.5" y2="16.5"/></svg></button>
      <a href="login.html" aria-label="Account"><svg class="icon" viewBox="0 0 24 24"><circle cx="12" cy="8" r="4"/><path d="M4 21c1.6-4 5-6 8-6s6.4 2 8 6"/></svg></a>
      <a href="profile.html#wishlist" aria-label="Wishlist"><svg class="icon" viewBox="0 0 24 24"><path d="…"/></svg></a>
      <a aria-label="Cart" href="cart.html" style="position:relative;">
        <svg class="icon" viewBox="0 0 24 24"><path d="M6 8h12l-1 12H7L6 8z"/><path d="M9 8V6a3 3 0 0 1 6 0v2"/></svg>
        <span class="cart-count" style="display:none;">0</span>
      </a>
      <button class="nav-toggle" aria-label="Menu" aria-expanded="false" id="navToggle">
        <svg width="22" height="22" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.8" fill="none"><line x1="3" y1="6" x2="21" y2="6"/><line x1="3" y1="12" x2="21" y2="12"/><line x1="3" y1="18" x2="21" y2="18"/></svg>
      </button>
    </div>
  </div>
</header>

<nav class="mobile-panel" id="mobilePanel">
  <div class="mp-group">
    <div class="mp-row">
      <a href="catalog.html">Surfboards</a>
      <button class="mp-toggle" aria-expanded="false" aria-label="Toggle Surfboards submenu"><span class="mp-chevron">&rsaquo;</span></button>
    </div>
    <div class="mp-sub">
      <a href="catalog.html">All Surfboards</a>
      <a href="catalog.html">Performance Shortboard</a>
      <a href="catalog.html">Longboard</a>
      <a href="catalog.html">Fish &amp; Twinfin</a>
      <a href="catalog.html">Fun Board</a>
      <a href="catalog.html">Soft-top / Beginner</a>
```

## Behavior (JS)

```js
  var toggle = document.getElementById('navToggle');
  var panel = document.getElementById('mobilePanel');
  toggle.addEventListener('click', function(){
    var open = panel.classList.toggle('open');
    toggle.setAttribute('aria-expanded', open ? 'true' : 'false');
  });
})();

  var nav = document.getElementById('siteNav');
  function updateNavHeight(){
    if(nav){ nav.style.setProperty('--nav-h', nav.offsetHeight + 'px'); }
  }
  updateNavHeight();
  window.addEventListener('resize', updateNavHeight);
  document.addEventListener('scroll', updateNavHeight, {passive:true});

  Array.prototype.forEach.call(document.querySelectorAll('.mp-toggle'), function(btn){
    btn.addEventListener('click', function(){
      var group = btn.closest('.mp-group');
      var sub = group ? group.querySelector('.mp-sub') : null;
      if(!sub) return;
      var open = sub.classList.toggle('open');
      btn.setAttribute('aria-expanded', open ? 'true' : 'false');
    });
  });

  // Mega-menu hover-intent: a short close delay bridges the dead-zone gap
  // between the nav-item's hover box and the mega-menu panel below it, so
  // moving the mouse down into the menu doesn't disestablish hover mid-transit.
  Array.prototype.forEach.call(document.querySelectorAll('.nav-item'), function(item){
    var closeTimer = null;
    item.addEventListener('mouseenter', function(){
      clearTimeout(closeTimer);
      item.classList.add('mega-open');
    });
    item.addEventListener('mouseleave', function(){
      clearTimeout(closeTimer);
      closeTimer = setTimeout(function(){
        item.classList.remove('mega-open');
      }, 250);
    });
```
