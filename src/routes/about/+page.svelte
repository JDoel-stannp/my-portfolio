<script lang="ts">
  import { onMount } from "svelte";

  interface Skill {
    label: string;
  }

  interface TimelineItem {
    year: string;
    title: string;
    desc: string;
  }

  const skills: Skill[] = [
    { label: "Svelte 4" },
    { label: "SvelteKit" },
    { label: "TypeScript" },
    { label: "CSS / Motion" },
    { label: "SVGator" },
    { label: "GSAP" },
    { label: "Animation" },
    { label: "Ableton Live" },
    { label: "Figma" },
    { label: "Node.js" },
    { label: "Git" },
    { label: "Claude" },
    { label: "ChatGPT" },
  ];

  const timeline: TimelineItem[] = [
    {
      year: "2022",
      title: "Self-Taught Beginnings",
      desc: "Started coding from scratch using Codecademy, Udemy and YouTube tutorials — building a solid foundation in web development.",
    },
    {
      year: "2022–2023",
      title: "One-on-One Tutoring",
      desc: "Completed 3 months of personalised tutoring with a University Professor of Software Development, accelerating my learning significantly.",
    },
    {
      year: "2023–Present",
      title: "Apprenticeship at Stannp",
      desc: "Joined Stannp — The Direct Mail Platform — as an apprentice developer. Building dynamic, responsive and animated frontends and growing every day.",
    },
    {
      year: "2025",
      title: "Featured on SVGator",
      desc: "Selected by SVGator to be featured on their website and social media channels as a showcase of creative work produced with their tool.",
    },
  ];

  let visible: boolean = false;
  let visibleItems: boolean[] = timeline.map(() => false);
  let timelineEl: HTMLElement;

  onMount(() => {
    setTimeout(() => (visible = true), 100);

    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            timeline.forEach((_, i) => {
              setTimeout(() => {
                visibleItems[i] = true;
                visibleItems = [...visibleItems];
              }, i * 800);
            });
            observer.disconnect();
          }
        });
      },
      { threshold: 0.2 },
    );

    observer.observe(timelineEl);

    return () => observer.disconnect();
  });
</script>

<svelte:head>
  <title>About — James Doel</title>
</svelte:head>

<section class="max-w-5xl mx-auto px-6 pt-32 pb-24">
  <!-- Header -->
  <div
    class="transition-all duration-700 {visible
      ? 'opacity-100 translate-y-0'
      : 'opacity-0 translate-y-6'}"
  >
    <p class="text-xs tracking-[0.2em] uppercase text-accent mb-3">Who I Am</p>
    <h1 class="font-serif text-[clamp(2.5rem,6vw,4rem)] mb-4">About Me</h1>
    <div
      class="w-12 h-0.5 bg-gradient-to-r from-accent to-accent2 rounded mb-10"
    ></div>
  </div>

  <!-- Bio + Portrait -->
  <div
    class="grid grid-cols-1 md:grid-cols-[240px_1fr] gap-12 mb-28
    transition-all duration-700 delay-150
    {visible ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-6'}"
  >
    <!-- Portrait -->
    <div class="flex flex-col items-center gap-4">
      <img
        src="/Image.webp"
        alt="James Doel"
        class="rounded-xl w-full max-w-[240px] object-cover"
      />
      <div class="text-center">
        <p class="font-medium text-sm">James Doel</p>
        <p class="text-muted text-xs mt-0.5">
          Svelte Dev · Animator · Musician
        </p>
      </div>
    </div>

    <!-- Bio text -->
    <div
      class="flex flex-col gap-5 text-[#aaaaaa] text-[1.05rem] leading-relaxed"
    >
      <p>
        I'm a Svelte Developer, Animator and amateur Music enthusiast with a
        passion for creating dynamic, responsive and animated frontends. I
        currently work for
        <a
          href="https://www.stannp.com"
          target="_blank"
          rel="noopener noreferrer"
          class="text-accent hover:underline"
          >Stannp — The Direct Mail Platform</a
        >, where I've been completing my apprenticeship for the past 2 years.
      </p>
      <p>
        I have a deep passion for bringing designs to life exactly to
        specification, with a focus on animation tooling, automation and
        marketing insights. My work caught the attention of the team at SVGator,
        who selected me to be featured on their website and social media
        channels as a showcase of creative work produced with their tool.
      </p>
      <p>
        I began coding 3 years ago, completely self-taught through Codecademy,
        Udemy and YouTube. I then had the privilege of one-on-one tutoring with
        a University Professor of Software Development for 3 months, which
        really accelerated my growth. Since joining Stannp I've learned
        enormously and valued every moment of the journey.
      </p>
    </div>
  </div>

  <!-- Skills -->
  <div
    class="mb-24 transition-all duration-700 delay-200
    {visible ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-6'}"
  >
    <h2 class="font-serif text-2xl mb-8">Skills & Tools</h2>
    <div class="flex flex-wrap gap-3">
      {#each skills as { label }}
        <span
          class="px-5 py-2 bg-card border border-border rounded-full text-sm text-muted
          hover:border-accent hover:text-white transition-all duration-200 cursor-default"
        >
          {label}
        </span>
      {/each}
    </div>
  </div>

  <!-- Timeline -->
  <div
    class="transition-all duration-700 delay-300
    {visible ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-6'}"
  >
    <h2 class="font-serif text-2xl mb-10">My Journey</h2>

    <div bind:this={timelineEl}>
      {#each timeline as { year, title, desc }, i}
        <div class="flex gap-8" style="align-items:stretch;">
          <!-- Dot + Line -->
          <div class="flex flex-col items-center flex-shrink-0 w-14">
            <div
              class="w-14 h-14 rounded-full flex items-center justify-center flex-shrink-0
              border-[3px] border-accent"
              style="background:#1a1a2e;
                box-shadow:0 0 20px rgba(167,139,250,0.6);
                transition: opacity 0.4s ease, transform 0.4s ease;
                opacity:{visibleItems[i] ? 1 : 0};
                transform:{visibleItems[i] ? 'scale(1)' : 'scale(0.4)'};"
            >
              <div
                class="w-4 h-4 rounded-full bg-accent"
                style="box-shadow:0 0 8px rgba(167,139,250,0.9);"
              ></div>
            </div>
            {#if i < timeline.length - 1}
              <div
                class="w-[3px] flex-1"
                style="background:linear-gradient(to bottom, #a78bfa, #60a5fa);
                  transform-origin: top;
                  transition: transform 0.6s ease 0.3s;
                  transform:scaleY({visibleItems[i] ? 1 : 0});"
              ></div>
            {/if}
          </div>

          <!-- Text -->
          <div
            class="flex-1 pt-3 pb-16"
            style="transition: opacity 0.5s ease 0.15s, transform 0.5s ease 0.15s;
              opacity:{visibleItems[i] ? 1 : 0};
              transform:{visibleItems[i]
              ? 'translateY(0)'
              : 'translateY(12px)'};"
          >
            <p
              class="text-[0.7rem] tracking-[0.2em] uppercase text-accent mb-1.5"
            >
              {year}
            </p>
            <h3 class="font-serif text-2xl mb-2">{title}</h3>
            <p class="text-sm leading-relaxed text-muted">{desc}</p>
          </div>
        </div>
      {/each}
    </div>
  </div>
</section>
