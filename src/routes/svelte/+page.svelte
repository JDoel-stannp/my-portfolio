<script>
  import RetroTyping from "$lib/components/RetroTyping.svelte";
</script>

<div
  class="min-h-screen flex flex-col items-center justify-center gap-8 bg-zinc-800"
>
  <RetroTyping />
  <a
    href="https://github.com"
    target="_blank"
    rel="noopener noreferrer"
    class="inline-flex items-center gap-2 px-6 py-3 rounded-lg font-mono text-sm
           text-emerald-400 border border-emerald-400/40 bg-emerald-400/5
           hover:bg-emerald-400/15 hover:border-emerald-400/70 hover:shadow-[0_0_18px_rgba(52,211,153,0.2)]
           transition-all duration-300 group"
  >
    <span
      class="text-emerald-600 group-hover:text-emerald-400 transition-colors"
      >&gt;</span
    >
    See my Svelte code &amp; work
    <svg
      xmlns="http://www.w3.org/2000/svg"
      class="w-4 h-4 opacity-60 group-hover:translate-x-1 group-hover:opacity-100 transition-all duration-300"
      fill="none"
      viewBox="0 0 24 24"
      stroke="currentColor"
      stroke-width="2"
    >
      <path
        stroke-linecap="round"
        stroke-linejoin="round"
        d="M17 8l4 4m0 0l-4 4m4-4H3"
      />
    </svg>
  </a>
</div>
