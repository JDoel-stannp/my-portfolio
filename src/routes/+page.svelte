<script lang="ts">
  import { onMount } from 'svelte';

  interface Discipline {
    emoji: string;
    title: string;
    desc: string;
    href: string;
  }

  const disciplines: Discipline[] = [
    { emoji: '⚡', title: 'Svelte Dev',  href: '/svelte',    desc: 'Building reactive, performant UIs with SvelteKit — clean code, great DX.' },
    { emoji: '🎬', title: 'Animation',   href: '/animation', desc: 'Motion design and interactive animation that brings interfaces to life.' },
    { emoji: '🎵', title: 'Music',       href: '/music',     desc: 'Original compositions spanning ambient, electronic and acoustic genres.' },
  ];

  let visible: boolean = false;
  onMount(() => setTimeout(() => (visible = true), 100));
</script>

<svelte:head>
  <title>James Doel — Creative Developer</title>
</svelte:head>

<!-- Hero -->
<section class="relative min-h-screen flex items-center overflow-hidden pt-16">

  <!-- Background orb -->
  <div class="absolute top-1/2 right-[-10%] -translate-y-1/2 w-[600px] h-[600px] rounded-full
    bg-[radial-gradient(circle,rgba(167,139,250,0.12)_0%,transparent_70%)]
    transition-all duration-[1200ms] pointer-events-none
    {visible ? 'opacity-100 scale-100' : 'opacity-0 scale-75'}">
  </div>

  <div class="relative z-10 max-w-5xl mx-auto px-6">
    <div class="max-w-2xl transition-all duration-700
      {visible ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-6'}">

      <p class="text-xs tracking-[0.2em] uppercase text-accent mb-3">Available for work</p>

      <h1 class="font-serif text-[clamp(3rem,8vw,5.5rem)] leading-none mb-4
        bg-gradient-to-br from-white via-white to-accent bg-clip-text text-transparent">
        James Doel
      </h1>

      <h2 class="font-sans text-[clamp(1rem,2.5vw,1.25rem)] font-light text-muted mb-6">
        Svelte Developer
        <span class="text-accent mx-1">/</span>
        Animator
        <span class="text-accent mx-1">/</span>
        Musician
      </h2>

      <p class="text-[#aaaaaa] text-lg max-w-xl mb-10 leading-relaxed">
        I build fast, beautiful web experiences and bring ideas to life through
        motion and sound. Based somewhere creative.
      </p>

      <div class="flex flex-wrap gap-4">
        <a href="/svelte"
          class="px-7 py-3 bg-accent text-white rounded-md text-sm font-medium
            hover:opacity-85 hover:-translate-y-px transition-all duration-200">
          View My Work
        </a>
        <a href="/contact"
          class="px-7 py-3 border border-border rounded-md text-sm text-muted
            hover:border-accent hover:text-white transition-all duration-200">
          Get In Touch
        </a>
      </div>

    </div>
  </div>
</section>

<!-- Disciplines -->
<section class="max-w-5xl mx-auto px-6 pb-24">
  <p class="text-xs tracking-[0.2em] uppercase text-accent mb-3">What I Do</p>
  <h2 class="font-serif text-[clamp(2rem,5vw,3rem)] mb-4">
    Three Passions,<br>One Creative Vision
  </h2>
  <div class="w-12 h-0.5 bg-gradient-to-r from-accent to-accent2 rounded mb-10"></div>

  <div class="grid grid-cols-1 sm:grid-cols-3 gap-6">
    {#each disciplines as { emoji, title, desc, href }}
      <a {href}
        class="group block bg-card border border-border rounded-xl p-8
          hover:border-accent hover:-translate-y-1 hover:glow
          transition-all duration-200">
        <span class="text-4xl mb-4 block">{emoji}</span>
        <h3 class="font-serif text-xl mb-2">{title}</h3>
        <p class="text-muted text-sm leading-relaxed">{desc}</p>
      </a>
    {/each}
  </div>
</section>