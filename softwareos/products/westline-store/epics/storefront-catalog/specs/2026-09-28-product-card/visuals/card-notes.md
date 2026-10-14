# Product card: design notes

These are extracted from `westline-site-desing/index.html`, the Featured gear section. Base64 image data is replaced with `DATAURI`.

## Design → prop mapping

| Design | Card |
|---|---|
| `.prod-card` border 1px `--border`, radius 9px, white | `<article>` shell |
| `.prod-card-link` overlay + `.visually-hidden` "View <name>" | overlay `next/link` to `product.href`, `sr-only` text |
| `.prod-media` aspect 4:5, bg `--surface-2` (#F9F9F9) | image area |
| `img` `object-fit: contain`; `style="object-fit:cover"` on the wetsuit, boardshort and traction pad | `product.imageFit` (`contain` for Category `surfboards`, otherwise `cover`) |
| stacked `img`, `.is-active` visible, others `opacity:0`, `.25s` fade | `ProductCardMedia` active index |
| first `img` `alt` = name; extras `alt=""`, `loading="lazy"` | AC6 |
| `.prod-photo-dots`, only when there are 2+ photos, max 4, "Show photo N", `aria-current` | AC5 |
| `.prod-wish` top-right 36px, `aria-pressed`, label toggle, filled when pressed | AC8 (controlled via `isFavorite` / `onToggleFavorite`) |
| `.prod-body h3` 16px/700; `.price` 16px/400 | name `<h3>`, `formatPrice` |
| `.tag`, `.spec`, `.btn-add` (styled, not used in the canonical markup) | out of scope |
| `.prod-slider`, `.prod-arrow`, `.prod-dots` (Featured gear slider) | out of scope (a Home section spec) |

## CSS

```css
  /* Shared slider dots: current slide is a wide pill. Add .on-dark over photos/video */
  .slider-dots{display:flex; align-items:center; justify-content:center; gap:8px;}
  .slider-dots[hidden]{display:none;}
  .slider-dot{
    position:relative; width:8px; height:8px; padding:0; border:none; border-radius:999px; cursor:pointer;
    background:var(--border); transition:width .2s ease, background-color .2s ease;
  }
  .slider-dot::before{content:''; position:absolute; inset:-10px -4px;}
  .slider-dot[aria-current="true"]{width:22px; background:var(--ink);}
  .slider-dots.on-dark .slider-dot{background:rgba(255,255,255,.45);}
  .slider-dots.on-dark .slider-dot:hover{background:rgba(255,255,255,.75);}
  .slider-dots.on-dark .slider-dot[aria-current="true"]{background:#fff;}
  .prod-card{position:relative; border:1px solid var(--border); border-radius:var(--radius); overflow:hidden; background:var(--white);}
  .prod-card-link{position:absolute; inset:0; z-index:1;}
  .visually-hidden{position:absolute; width:1px; height:1px; overflow:hidden; clip:rect(0,0,0,0); white-space:nowrap;}
  .prod-media{position:relative; aspect-ratio:4/5;}
  .prod-media img{position:absolute; inset:0; width:100%; height:100%; object-fit:contain; display:block; transition:opacity .25s ease;}
  /* Extra photos stack on the first; dots swap which one is shown */
  .prod-media img:not(.is-active){opacity:0;}
  .prod-photo-dots{
    position:absolute; left:50%; bottom:12px; transform:translateX(-50%); z-index:2;
    padding:6px 8px; border-radius:999px; background:rgba(255,255,255,.85); box-shadow:0 1px 4px rgba(16,24,40,.12);
  }
  .prod-photo-dots .slider-dot{background:rgba(16,24,40,.25);}
  .prod-photo-dots .slider-dot[aria-current="true"]{background:var(--ink);}
  .prod-wish{
    position:absolute; top:10px; right:10px; z-index:2; width:36px; height:36px; border-radius:50%;
    background:var(--white); border:1px solid var(--border); box-shadow:0 1px 4px rgba(16,24,40,.10);
    display:flex; align-items:center; justify-content:center; color:var(--horizon);
    transition:border-color .15s ease;
  }
  .prod-wish:hover{border-color:var(--horizon);}
  .prod-wish svg{width:17px; height:17px; stroke:currentColor; fill:none; stroke-width:2;}
  .prod-wish[aria-pressed="true"] svg{fill:currentColor;}
  .p1{background:var(--surface-2);}
  .p2{background:var(--surface-2);}
  .p3{background:var(--surface-2);}
  .p4{background:var(--surface-2);}
  .prod-body{padding:16px;}
  .prod-body .tag{margin-bottom:10px; display:inline-flex;}
  .prod-body h3{font-size:16px; margin:0 0 6px; font-weight:700;}
  .spec{
    font-family:'JetBrains Mono',monospace; font-size:11.5px; color:var(--muted); margin:0 0 12px;
  }
  .prod-foot{display:flex; align-items:center; justify-content:space-between; gap:10px;}
  .prod-foot .price{font-weight:400; font-size:16px;}
  .btn-add{
    position:relative; z-index:2;
```

## Markup (first card)

```html
        <div class="prod-card">
          <a class="prod-card-link" href="product-surfboard-tideline.html"><span class="visually-hidden">View Tideline 6'0" Performance Shortboard</span></a>
          <div class="prod-media p1">
            <img class="is-active" src="DATAURI" alt="Tideline 6'0&quot; Performance Shortboard">
            <img src="DATAURI" alt="" loading="lazy">
            <button class="prod-wish" type="button" aria-label="Add to wishlist" aria-pressed="false"><svg viewBox="0 0 24 24"><path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"/></svg></button>
            <div class="slider-dots prod-photo-dots" hidden></div>
          </div>
          <div class="prod-body">
            <h3>Tideline 6'0" Performance Shortboard</h3>
            <div class="prod-foot">
              <span class="price">$829</span>
            </div>
          </div>
```

## Behavior (JS)

```js
  // Product card: photo dots (only when a card has 2+ photos, max 4) and wishlist toggle
  Array.prototype.forEach.call(document.querySelectorAll('.prod-card'), function(card){
    var photos = Array.prototype.slice.call(card.querySelectorAll('.prod-media img'), 0, 4);
    var photoDots = card.querySelector('.prod-photo-dots');
    if(photoDots && photos.length > 1){
      photoDots.hidden = false;
      photos.forEach(function(photo, i){
        var dot = document.createElement('button');
        dot.type = 'button';
        dot.className = 'slider-dot';
        dot.setAttribute('aria-label', 'Show photo ' + (i + 1));
        dot.setAttribute('aria-current', i === 0 ? 'true' : 'false');
        dot.addEventListener('click', function(){
          photos.forEach(function(p, j){ p.classList.toggle('is-active', j === i); });
          Array.prototype.forEach.call(photoDots.children, function(d, j){
            d.setAttribute('aria-current', j === i ? 'true' : 'false');
          });
        });
        photoDots.appendChild(dot);
      });
    }

    var wish = card.querySelector('.prod-wish');
    if(wish){
      wish.addEventListener('click', function(){
        var on = wish.getAttribute('aria-pressed') !== 'true';
        wish.setAttribute('aria-pressed', on ? 'true' : 'false');
        wish.setAttribute('aria-label', on ? 'Remove from wishlist' : 'Add to wishlist');
      });
    }
  });
  // Featured gear slider: arrows appear only when there is room to scroll that way;
  // one dot per snap position (4 cards with 2 visible = 3 dots), current one highlighted
```
