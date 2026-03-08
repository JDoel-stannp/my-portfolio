<script>
  import { onMount, onDestroy } from "svelte";
  import { gsap } from "gsap";

  const steps = [
    { type: "input", text: "console.log('Hello World')" },
    { type: "output", text: "Hello World" },
    { type: "cursor" },
  ];

  let lines = [];
  let cursorEl;
  let screenEl;
  let ctx;

  onMount(() => {
    ctx = gsap.context(() => {
      const tl = gsap.timeline({ delay: 0.4 });

      steps.forEach((step, stepIdx) => {
        if (step.type === "cursor") {
          tl.add(() => {
            lines = [...lines, { type: "cursor", text: "" }];
            if (screenEl) screenEl.scrollTop = screenEl.scrollHeight;
          });
          tl.add(() => {
            if (cursorEl) {
              gsap.to(cursorEl, {
                opacity: 0,
                repeat: -1,
                yoyo: true,
                duration: 0.53,
                ease: "steps(1)",
              });
            }
          });
          return;
        }

        // Push empty line
        tl.add(() => {
          lines = [...lines, { type: step.type, text: "" }];
        });

        // Type chars
        step.text.split("").forEach((char) => {
          tl.add(() => {
            lines = lines.map((l, i) =>
              i === stepIdx ? { ...l, text: l.text + char } : l,
            );
            if (screenEl) screenEl.scrollTop = screenEl.scrollHeight;
          }, "+=0.055");
        });

        // Pause then "run" — output appears after Enter
        if (step.type === "input") {
          tl.add(() => {}, "+=0.4");
        } else {
          tl.add(() => {}, "+=0.2");
        }
      });
    });
  });

  onDestroy(() => ctx?.revert());
</script>

<div class="min-h-screen bg-zinc-800 flex items-center justify-center p-8">
  <!-- Monitor -->
  <div class="flex flex-col items-center">
    <!-- Bezel -->
    <div
      class="w-[520px] rounded-xl p-5 pb-4
                bg-gradient-to-br from-stone-300 via-stone-400 to-stone-500
                shadow-[inset_0_2px_4px_rgba(255,255,255,0.55),inset_0_-2px_4px_rgba(0,0,0,0.2),6px_8px_24px_rgba(0,0,0,0.6),0_0_0_2px_#9a8f83]"
    >
      <!-- Screen area -->
      <div
        class="relative rounded-md overflow-hidden border-4 border-stone-800 h-64
                  bg-black shadow-[inset_0_0_40px_rgba(0,255,80,0.07)]"
      >
        <!-- Scanlines -->
        <div
          class="absolute inset-0 z-10 pointer-events-none"
          style="background: repeating-linear-gradient(to bottom, transparent 0px, transparent 3px, rgba(0,0,0,0.18) 3px, rgba(0,0,0,0.18) 4px)"
        ></div>

        <!-- Terminal content -->
        <div
          bind:this={screenEl}
          class="h-full overflow-y-auto p-4 scrollbar-none font-mono text-sm leading-relaxed"
        >
          {#each lines as line, i}
            <div class="flex items-start gap-2 min-h-[1.6rem]">
              <!-- Prompt -->
              {#if line.type === "input" || line.type === "cursor"}
                <span
                  class="text-emerald-400 select-none shrink-0"
                  style="text-shadow: 0 0 6px #34d399, 0 0 14px #10b981"
                >
                  &gt;
                </span>
              {:else}
                <span class="text-zinc-600 select-none shrink-0 w-[1ch]"></span>
              {/if}

              <!-- Text -->
              <span
                class="{line.type === 'output'
                  ? 'text-white'
                  : 'text-emerald-400'} whitespace-pre"
                style={line.type !== "output"
                  ? "text-shadow: 0 0 6px #34d399, 0 0 14px #10b981"
                  : ""}
              >
                {line.text}
              </span>

              <!-- Blinking cursor on last line -->
              {#if line.type === "cursor"}
                <span
                  bind:this={cursorEl}
                  class="text-emerald-400 leading-none"
                  style="text-shadow: 0 0 8px #34d399">█</span
                >
              {/if}
            </div>
          {/each}

          <!-- CTA Button -->
          {#if lines.length === steps.length}
            <div class="mt-4 flex">
              <a
                href="https://github.com"
                target="_blank"
                rel="noopener noreferrer"
                class="inline-flex items-center gap-2 px-4 py-2 rounded font-mono text-xs
                        text-emerald-400 border border-emerald-400/40 bg-emerald-400/5
                        hover:bg-emerald-400/15 hover:border-emerald-400/70
                        hover:shadow-[0_0_12px_rgba(52,211,153,0.2)]
                        transition-all duration-300 group"
                style="text-shadow: 0 0 6px #34d399"
              >
                <span
                  class="text-emerald-600 group-hover:text-emerald-400 transition-colors"
                  >&gt;</span
                >
                See my Svelte code &amp; work
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  class="w-3 h-3 opacity-60 group-hover:translate-x-1 group-hover:opacity-100 transition-all duration-300"
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
          {/if}
        </div>
      </div>

      <!-- Brand strip -->
      <div class="mt-3 flex items-center justify-between px-1">
        <span
          class="text-[10px] tracking-[4px] uppercase text-stone-500 font-sans"
        >
          COMPURETRO 486
        </span>
        <!-- Power LED -->
        <div class="flex items-center gap-1.5">
          <div
            class="w-2 h-2 rounded-full bg-emerald-400 animate-pulse
                      shadow-[0_0_6px_#34d399,0_0_12px_#10b981]"
          ></div>
          <span
            class="text-[8px] tracking-widest uppercase text-stone-500 font-sans"
            >Power</span
          >
        </div>
      </div>
    </div>

    <!-- Neck -->
    <div
      class="w-20 h-7 bg-gradient-to-b from-stone-400 to-stone-500
                shadow-md"
      style="clip-path: polygon(15% 0%, 85% 0%, 100% 100%, 0% 100%)"
    ></div>

    <!-- Base -->
    <div
      class="w-56 h-5 rounded-b-lg
                bg-gradient-to-b from-stone-300 to-stone-500
                shadow-[0_4px_12px_rgba(0,0,0,0.4),inset_0_1px_2px_rgba(255,255,255,0.4)]"
    ></div>

    <!-- CTA Button -->
    <a
      href="https://github.com"
      target="_blank"
      rel="noopener noreferrer"
      class="mt-8 inline-flex items-center gap-2 px-6 py-3 rounded-lg font-mono text-sm
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
</div>

<style>
  .scrollbar-none {
    scrollbar-width: none;
  }
  .scrollbar-none::-webkit-scrollbar {
    display: none;
  }
</style>
