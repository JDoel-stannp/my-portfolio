<script lang="ts">
  import showreel from "$lib/assets/Showcase-Reel-Banners.mp4?url";
  import { gsap } from "gsap";
  import { ScrollTrigger } from "gsap/ScrollTrigger";
  import RubiksCube from "$lib/components/RubiksCube.svelte";
  import ParticleField from "$lib/components/ParticleField.svelte";
  import DNAHelix from "$lib/components/DNAHelix.svelte";
  import { onMount, tick } from "svelte";

  gsap.registerPlugin(ScrollTrigger);

  const words: string[] = ["Motion", "Design", "Interaction", "Animation"];
  let wordIndex: number = 0;
  let currentWord: string = words[0];

  let wordEl: HTMLElement;
  let heroTitleEl: HTMLElement;
  let heroSubEl: HTMLElement;
  let heroBadgeEl: HTMLElement;

  let card0: HTMLElement;
  let card1: HTMLElement;
  let card2: HTMLElement;

  let stat0: HTMLElement;
  let stat1: HTMLElement;
  let stat2: HTMLElement;

  let videoEl: HTMLVideoElement;

  let reveal0: HTMLElement;
  let reveal1: HTMLElement;
  let reveal2: HTMLElement;
  let reveal3: HTMLElement;
  let reveal4: HTMLElement;

  onMount(async () => {
    await tick();
    await new Promise((r) => setTimeout(r, 100));

    // ── Hero entrance ──────────────────────────────────────────────
    if (heroBadgeEl && heroTitleEl && heroSubEl) {
      gsap
        .timeline()
        .from(
          heroBadgeEl,
          { opacity: 0, y: 20, duration: 0.6, ease: "power3.out" },
          0.2,
        )
        .from(
          heroTitleEl,
          { opacity: 0, y: 40, duration: 0.8, ease: "power3.out" },
          0.4,
        )
        .from(
          heroSubEl,
          { opacity: 0, y: 20, duration: 0.6, ease: "power3.out" },
          0.8,
        );
    }

    // ── Cycling word ───────────────────────────────────────────────
    const interval = setInterval(() => {
      if (!wordEl) return;
      gsap.to(wordEl, {
        opacity: 0,
        y: -20,
        duration: 0.35,
        ease: "power2.in",
        onComplete: () => {
          wordIndex = (wordIndex + 1) % words.length;
          currentWord = words[wordIndex];
          gsap.fromTo(
            wordEl,
            { opacity: 0, y: 20 },
            { opacity: 1, y: 0, duration: 0.45, ease: "power2.out" },
          );
        },
      });
    }, 2000);

    // ── Scroll: stagger cards ──────────────────────────────────────
    const cards = [card0, card1, card2].filter(Boolean);
    if (cards.length) {
      gsap.from(cards, {
        scrollTrigger: { trigger: cards[0], start: "top 85%" },
        opacity: 0,
        y: 60,
        duration: 0.7,
        stagger: 0.15,
        ease: "power3.out",
      });
    }

    // ── Scroll: stat counters ──────────────────────────────────────
    [
      { el: stat0, target: 24 },
      { el: stat1, target: 130 },
      { el: stat2, target: 3 },
    ].forEach(({ el, target }) => {
      if (!el) return;
      gsap.from(
        { val: 0 },
        {
          scrollTrigger: { trigger: el, start: "top 90%" },
          val: target,
          duration: 1.8,
          ease: "power2.out",
          onUpdate: function () {
            if (el)
              el.textContent = Math.round(this.targets()[0].val).toString();
          },
        },
      );
    });

    // ── Scroll: section reveals ────────────────────────────────────
    [reveal0, reveal1, reveal2, reveal3, reveal4].forEach((el) => {
      if (!el) return;
      gsap.from(el, {
        scrollTrigger: { trigger: el, start: "top 88%" },
        opacity: 0,
        y: 40,
        duration: 0.8,
        ease: "power3.out",
      });
    });

    // ── Video autoplay on scroll ───────────────────────────────────
    if (videoEl) {
      const videoObserver = new IntersectionObserver(
        (entries) => {
          entries.forEach((entry) => {
            if (entry.isIntersecting) videoEl.play();
            else videoEl.pause();
          });
        },
        { threshold: 0.5 },
      );
      videoObserver.observe(videoEl);
    }

    return () => clearInterval(interval);
  });
</script>

<svelte:head>
  <title>Animation — James Doel</title>
</svelte:head>

<!-- ── HERO ──────────────────────────────────────────────────────────── -->
<section
  class="min-h-screen flex flex-col items-center justify-center text-center px-6 pt-16 relative overflow-hidden"
>
  <div
    class="absolute inset-0 pointer-events-none"
    style="background: radial-gradient(ellipse 60% 50% at 50% 40%, rgba(167,139,250,0.08) 0%, transparent 70%);"
  ></div>

  <div bind:this={heroBadgeEl}>
    <p class="text-xs tracking-[0.2em] uppercase text-accent mb-6">
      Motion & Design
    </p>
  </div>

  <div bind:this={heroTitleEl}>
    <h1 class="font-serif text-[clamp(3rem,9vw,6.5rem)] leading-none mb-6">
      I Create<br />
      <span
        bind:this={wordEl}
        class="inline-block"
        style="color:#a78bfa; min-width:320px;"
      >
        {currentWord}
      </span>
    </h1>
  </div>

  <div bind:this={heroSubEl}>
    <p class="text-[#aaaaaa] text-lg max-w-xl mx-auto mb-10">
      Bringing interfaces to life through motion, timing and interaction —
      crafted with GSAP, SVGator and CSS.
    </p>
    <a
      href="#work"
      class="inline-flex items-center gap-2 px-6 py-3 rounded-full text-sm font-medium text-white
        border border-border hover:border-accent transition-all duration-200"
    >
      See My Work <span style="color:#a78bfa">↓</span>
    </a>
  </div>
</section>

<!-- ── STATS ─────────────────────────────────────────────────────────── -->
<section class="max-w-5xl mx-auto px-6 pb-24">
  <div style="display:grid; grid-template-columns:repeat(3,1fr); gap:2rem;">
    <div class="rounded-2xl p-8 text-center border border-border bg-card">
      <p class="font-serif text-5xl text-accent mb-2" bind:this={stat0}>0</p>
      <p class="text-muted text-sm">Projects Shipped</p>
    </div>
    <div class="rounded-2xl p-8 text-center border border-border bg-card">
      <p class="font-serif text-5xl text-accent mb-2" bind:this={stat1}>0</p>
      <p class="text-muted text-sm">Animations Created</p>
    </div>
    <div class="rounded-2xl p-8 text-center border border-border bg-card">
      <p class="font-serif text-5xl text-accent mb-2" bind:this={stat2}>0</p>
      <p class="text-muted text-sm">Years Experience</p>
    </div>
  </div>
</section>

<!-- ── WORK ──────────────────────────────────────────────────────────── -->
<section id="work" class="max-w-5xl mx-auto px-6 pb-24">
  <div bind:this={reveal0}>
    <p class="text-xs tracking-[0.2em] uppercase text-accent mb-3">What I Do</p>
    <h2 class="font-serif text-[clamp(2rem,5vw,3rem)] mb-4">Animation Work</h2>
    <div
      class="w-12 h-0.5 bg-gradient-to-r from-accent to-accent2 rounded mb-12"
    ></div>
  </div>

  <div class="grid grid-cols-1 sm:grid-cols-3 gap-6">
    <div
      bind:this={card0}
      class="rounded-xl p-8 border border-border bg-card hover:border-accent hover:-translate-y-1 transition-all duration-200 cursor-default"
    >
      <span class="text-4xl mb-5 block">✨</span>
      <h3 class="font-serif text-xl mb-2">UI Micro-interactions</h3>
      <p class="text-muted text-sm leading-relaxed">
        Subtle animations that make interfaces feel alive and responsive.
      </p>
    </div>
    <div
      bind:this={card1}
      class="rounded-xl p-8 border border-border bg-card hover:border-accent hover:-translate-y-1 transition-all duration-200 cursor-default"
    >
      <span class="text-4xl mb-5 block">🎞</span>
      <h3 class="font-serif text-xl mb-2">Motion Graphics</h3>
      <p class="text-muted text-sm leading-relaxed">
        Title sequences, brand animations and animated illustrations.
      </p>
    </div>
    <div
      bind:this={card2}
      class="rounded-xl p-8 border border-border bg-card hover:border-accent hover:-translate-y-1 transition-all duration-200 cursor-default"
    >
      <span class="text-4xl mb-5 block">🌀</span>
      <h3 class="font-serif text-xl mb-2">WebGL & SVG</h3>
      <p class="text-muted text-sm leading-relaxed">
        Explorations using Three.js, SVGator and custom shaders.
      </p>
    </div>
  </div>
</section>

<!-- ── PARTICLE FIELD ─────────────────────────────────────────────────── -->
<section class="max-w-5xl mx-auto px-6 pb-24">
  <div bind:this={reveal1}>
    <p class="text-xs tracking-[0.2em] uppercase text-accent mb-3">Three.js</p>
    <h2 class="font-serif text-[clamp(2rem,5vw,3rem)] mb-4">Particle Field</h2>
    <div
      class="w-12 h-0.5 bg-gradient-to-r from-accent to-accent2 rounded mb-12"
    ></div>
    <ParticleField />
  </div>
</section>

<!-- ── RUBIK'S CUBE ───────────────────────────────────────────────────── -->
<section class="max-w-5xl mx-auto px-6 pb-24">
  <div bind:this={reveal3}>
    <p class="text-xs tracking-[0.2em] uppercase text-accent mb-3">Three.js</p>
    <h2 class="font-serif text-[clamp(2rem,5vw,3rem)] mb-4">Interactive 3D</h2>
    <div
      class="w-12 h-0.5 bg-gradient-to-r from-accent to-accent2 rounded mb-12"
    ></div>
    <RubiksCube />
  </div>
</section>

<!-- ── DIAMOND ────────────────────────────────────────────────────────── -->
<section class="max-w-5xl mx-auto px-6 pb-24">
  <div bind:this={reveal2}>
    <p class="text-xs tracking-[0.2em] uppercase text-accent mb-3">Three.js</p>
    <h2 class="font-serif text-[clamp(2rem,5vw,3rem)] mb-4">DNA Helix</h2>
    <div
      class="w-12 h-0.5 bg-gradient-to-r from-accent to-accent2 rounded mb-12"
    ></div>
    <DNAHelix />
  </div>
</section>

<!-- ── SHOWREEL ───────────────────────────────────────────────────────── -->
<section class="max-w-5xl mx-auto px-6 pb-32">
  <div bind:this={reveal4}>
    <p class="text-xs tracking-[0.2em] uppercase text-accent mb-3">Showreel</p>
    <h2 class="font-serif text-[clamp(2rem,5vw,3rem)] mb-4">
      See It In Motion
    </h2>
    <div
      class="w-12 h-0.5 bg-gradient-to-r from-accent to-accent2 rounded mb-12"
    ></div>
    <div
      class="rounded-2xl border border-border overflow-hidden"
      style="height:360px;"
    >
      <video
        bind:this={videoEl}
        src={showreel}
        controls
        muted
        loop
        playsinline
        class="w-full h-full object-cover rounded-2xl"
      >
        <track kind="captions" />
      </video>
    </div>
  </div>
</section>
