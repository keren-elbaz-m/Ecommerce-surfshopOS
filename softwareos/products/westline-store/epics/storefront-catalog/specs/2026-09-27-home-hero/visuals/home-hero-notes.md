# Home Hero: design notes

Taken from `home.html` (the design export `index.html`). Base64 image data has been replaced with `DATAURI`. The three background images are `.bg-1`, `.bg-2` and `.bg-3` in the original file (they get extracted to `cms/seed/home-hero/` in T3).

Behavior summary:
- 3 slides, each with a headline (`h1.display`) and one `.btn-primary` CTA. No subtext in the design.
- The active slide's background zooms to `scale(1.08) translate(-2.5%,-1%)` over 11s, and its text slides in from `translateX(56px)` over 1s.
- The track moves with `transform .3s cubic-bezier(.4,0,.2,1)` and loops using cloned first and last slides, snapping at the ends on `transitionend`, with a 400ms fallback unlock.
- Autoplay every 5s. It pauses on mouseenter only when `(hover: hover) and (pointer: fine)` matches, and on focusin. Autoplay is off under reduced motion.
- Touch swipe threshold is 40px, with no `preventDefault`, so vertical scrolling still works.
- Design positions: slide 1 `center 55%`, slide 2 `center 50%`, slide 3 `center 40%`. Per-slide focal point is out of scope; the build uses center.

## CSS

```css
  :root{
    --ink:#101828;
    --white:#FFFFFF;
    --surface-2:#F9F9F9;
    --muted:#667085;
    --border:#D0D5DD;
    --horizon:#155EEF;
    --horizon-deep:#0E40A3;
    --horizon-ink:#F5F9FA;
    --sand:#94A3B8;
    --radius:9px;
    --radius-pill:20px;
    --maxw:1240px;
  }
  *{box-sizing:border-box;}
  /* HERO */
  .hero{
    position:relative; height:100vh; overflow:hidden; z-index:0; padding-block:0;
  }
  .hero::before{
    content:''; position:absolute; top:0; left:0; right:0; height:180px; z-index:4;
    background:linear-gradient(180deg, rgba(8,16,20,.5) 0%, rgba(8,16,20,0) 100%);
    pointer-events:none;
  }
  .hero-track{
    display:flex; height:100%; width:100%;
    transition:transform .3s cubic-bezier(.4,0,.2,1);
  }
  .hero-track.no-transition{transition:none;}
  .slide{
    position:relative; flex:0 0 100%; width:100%; height:100%;
    display:flex; align-items:flex-end; justify-content:flex-start; overflow:hidden;
  }
  .slide .bg{
    position:absolute; inset:0; z-index:0;
    transform-origin:20% 80%; transform:scale(1) translate(0,0);
    transition:transform 11s ease-out;
  }
  .slide .scrim{
    position:absolute; inset:0; z-index:1;
    background:linear-gradient(100deg, rgba(10,20,24,.6) 0%, rgba(10,20,24,.28) 42%, rgba(10,20,24,.05) 68%);
  }
  .slide .content{
    position:relative; z-index:2; color:#fff; max-width:760px; padding-bottom:70px;
    margin-inline:0;
    opacity:0; transform:translateX(56px);
    transition:opacity 1s ease-out, transform 1s ease-out;
  }
  .slide.is-active .bg{ transform:scale(1.08) translate(-2.5%,-1%); }
  .slide.is-active .content{ opacity:1; transform:translateX(0); }
  .slide.reset-anim .bg, .slide.reset-anim .content{ transition:none !important; }
  @media (prefers-reduced-motion: reduce){
    .slide .bg, .slide .content{ transition-duration:.01s !important; }
  }
  .slide h1.display{ font-size:clamp(28px,4.2vw,56px); margin:0 0 24px; }

  .bg-1{
    background-image:url("DATAURI");
    background-size:cover;
    background-position:center 55%;
    background-repeat:no-repeat;
  }
  .bg-2{
    background-image:url("DATAURI");
    background-size:cover;
    background-position:center 50%;
    background-repeat:no-repeat;
  }
  .bg-3{
    background-image:url("DATAURI");
    background-size:cover;
    background-position:center 40%;
    background-repeat:no-repeat;
  }

  .btn{
    font-family:'Inter',sans-serif; font-weight:700; font-size:14px;
    letter-spacing:0.02em; text-transform:uppercase;
    padding:14px 26px; border-radius:var(--radius); border:2px solid transparent;
    display:inline-flex; align-items:center; gap:8px; text-decoration:none;
  }
  .btn-primary{background:var(--horizon); color:var(--horizon-ink);}
  .btn-primary:hover{background:var(--horizon-deep);}
  .btn-outline{background:transparent; border-color:#fff; color:#fff;}
  .btn-outline:hover{background:rgba(255,255,255,.12);}
  .btn-ink{background:var(--ink); color:var(--white);}
  .btn-ink:hover{background:#000;}
  .cta-row{display:flex; gap:14px; flex-wrap:wrap;}

  .hero-arrow{
    position:absolute; top:50%; transform:translateY(-50%); z-index:3;
    background:rgba(18,33,42,.35); border:1px solid rgba(255,255,255,.35); color:#fff;
    width:44px; height:44px; border-radius:50%; display:flex; align-items:center; justify-content:center;
  }
  .hero-arrow:hover{background:rgba(18,33,42,.55);}
  .hero-prev{left:20px;} .hero-next{right:20px;}
  .hero-arrow svg{width:18px;height:18px;}
  @media (max-width:640px){
    .hero-arrow{width:36px; height:36px;}
    .hero-prev{left:12px;} .hero-next{right:12px;}
    .hero-arrow svg{width:15px;height:15px;}
  }

  .hero-bottom{
    position:absolute; left:0; right:0; bottom:56px; z-index:5; pointer-events:none;
    display:flex; justify-content:center;
  }
  .hero-dots{pointer-events:auto;}
  @media (max-width:640px){ .hero-bottom{ bottom:40px; } }

```

## Markup

```html
  <section class="hero" id="heroCarousel" aria-roledescription="carousel" aria-label="Featured">
    <div class="hero-track" id="heroTrack">
    <div class="slide" aria-hidden="false">
      <div class="bg bg-1"></div>
      <div class="scrim"></div>
      <div class="container content">
        <h1 class="display">Westline</h1>
        <div class="cta-row"><a class="btn btn-primary" href="catalog.html">Shop Now</a></div>
      </div>
    </div>
    <div class="slide" aria-hidden="true">
      <div class="bg bg-2"></div>
      <div class="scrim"></div>
      <div class="container content">
        <h1 class="display">Find your Surfboard</h1>
        <div class="cta-row"><a class="btn btn-primary" href="#finder">Take the Finder</a></div>
      </div>
    </div>
    <div class="slide" aria-hidden="true">
      <div class="bg bg-3"></div>
      <div class="scrim"></div>
      <div class="container content">
        <h1 class="display">The Dawn Patrol<br>Collection</h1>
        <div class="cta-row"><a class="btn btn-primary" href="catalog.html">Shop the Collection</a></div>
      </div>
    </div>
    </div>

    <button class="hero-arrow hero-prev" aria-label="Previous slide"><svg viewBox="0 0 24 24" stroke="currentColor" stroke-width="2" fill="none"><path d="M15 18l-6-6 6-6"/></svg></button>
    <button class="hero-arrow hero-next" aria-label="Next slide"><svg viewBox="0 0 24 24" stroke="currentColor" stroke-width="2" fill="none"><path d="M9 6l6 6-6 6"/></svg></button>
    <div class="hero-bottom">
      <div class="hero-dots slider-dots on-dark" role="tablist" aria-label="Slides">
        <button class="slider-dot" aria-current="true" aria-label="Slide 1"></button>
        <button class="slider-dot" aria-current="false" aria-label="Slide 2"></button>
        <button class="slider-dot" aria-current="false" aria-label="Slide 3"></button>
      </div>
    </div>
```

## Carousel JS

```js
(function(){
  var hero = document.getElementById('heroCarousel');
  var track = document.getElementById('heroTrack');
  var dots = Array.prototype.slice.call(hero.querySelectorAll('.slider-dot'));
  var reduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

  // Real slides, in order.
  var real = Array.prototype.slice.call(track.querySelectorAll('.slide'));
  var count = real.length;

  // Clone the first and last slide so next()/prev() can always animate
  // one step in the true direction, then snap invisibly at the ends —
  // this is what makes the loop feel continuous instead of jumping back.
  var firstClone = real[0].cloneNode(true);
  var lastClone = real[count - 1].cloneNode(true);
  firstClone.setAttribute('aria-hidden', 'true');
  lastClone.setAttribute('aria-hidden', 'true');
  track.insertBefore(lastClone, real[0]);
  track.appendChild(firstClone);

  var pos = 1; // track position; 1..count map to real[0..count-1]
  var timer = null;
  var animating = false;
  var animatingFallback = null;

  function realIndexFromPos(p){ return ((p - 1) + count) % count; }

  function paint(realIdx){
    real.forEach(function(s, i){
      var active = i === realIdx;
      s.setAttribute('aria-hidden', active ? 'false' : 'true');
      if(!active && s.classList.contains('is-active')){
        // instantly reset this slide's zoom/text state so it starts fresh next time it's shown
        s.classList.add('reset-anim');
        s.classList.remove('is-active');
        void s.offsetWidth;
        s.classList.remove('reset-anim');
      }
    });
    real[realIdx].classList.add('is-active');
    dots.forEach(function(d, i){ d.setAttribute('aria-current', i === realIdx ? 'true' : 'false'); });
  }

  function setPos(p, animate){
    track.classList.toggle('no-transition', !animate);
    if(!animate){ void track.offsetWidth; } // flush so the snap is truly instant
    track.style.transform = 'translateX(-' + (p * 100) + '%)';
    pos = p;
  }

  setPos(1, false); // start on real slide 0, no animation on load
  paint(0);

  track.addEventListener('transitionend', function(e){
    if(e.propertyName !== 'transform') return;
    clearTimeout(animatingFallback);
    if(pos === 0){ setPos(count, false); }
    else if(pos === count + 1){ setPos(1, false); }
    animating = false;
  });

  function goTo(targetPos){
    if(animating) return;
    animating = true;
    setPos(targetPos, true);
    paint(realIndexFromPos(targetPos));
    // Safety net: transitionend normally clears `animating`, but if it's ever
    // missed (a stalled/backgrounded tab, an interrupted transition), the
    // carousel would otherwise stay locked and never advance again. This
    // force-clears the lock shortly after the transition should have finished.
    clearTimeout(animatingFallback);
    animatingFallback = setTimeout(function(){ animating = false; }, 400);
  }
  function next(){ goTo(pos + 1); }
  function prev(){ goTo(pos - 1); }
  function goToReal(i){ goTo(i + 1); }

  function start(){
    stop();
    if(reduced) return;
    timer = setInterval(next, 5000);
  }
  function stop(){ if(timer){ clearInterval(timer); timer = null; } }

  hero.querySelector('.hero-next').addEventListener('click', function(){ next(); start(); });
  hero.querySelector('.hero-prev').addEventListener('click', function(){ prev(); start(); });
  dots.forEach(function(d, i){ d.addEventListener('click', function(){ goToReal(i); start(); }); });

  // Only pause-on-hover for pointers that actually support persistent hover.
  // Touch devices can fire a synthetic "mouseenter" after a tap with no
  // matching "mouseleave" (real mobile browsers do this for compatibility),
  // which would otherwise stop() the timer with nothing left to start() it
  // again — the carousel would look "stuck" forever after the first tap.
  var supportsHover = window.matchMedia('(hover: hover) and (pointer: fine)').matches;
  if(supportsHover){
    hero.addEventListener('mouseenter', stop);
    hero.addEventListener('mouseleave', start);
  }
  hero.addEventListener('focusin', stop);
  hero.addEventListener('focusout', start);

  // Touch swipe (mobile): track horizontal drag distance, advance on release
  // if it clears a threshold. No preventDefault on the move — vertical page
  // scroll during a mostly-vertical touch still works normally.
  var touchStartX = 0, touchStartY = 0, touchDeltaX = 0, isTouching = false;
  var SWIPE_THRESHOLD = 40;

  track.addEventListener('touchstart', function(e){
    if(e.touches.length !== 1) return;
    isTouching = true;
    touchDeltaX = 0;
    touchStartX = e.touches[0].clientX;
    touchStartY = e.touches[0].clientY;
    stop();
  }, {passive:true});

  track.addEventListener('touchmove', function(e){
    if(!isTouching || e.touches.length !== 1) return;
    touchDeltaX = e.touches[0].clientX - touchStartX;
  }, {passive:true});

  track.addEventListener('touchend', function(){
    if(!isTouching) return;
    isTouching = false;
    if(Math.abs(touchDeltaX) > SWIPE_THRESHOLD){
      if(touchDeltaX < 0){ next(); } else { prev(); }
    }
    start();
  });

  start();
})();
```
