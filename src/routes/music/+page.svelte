<script lang="ts">
  import { onMount } from "svelte";
  import { gsap } from "gsap";

  type Track = { name: string; url: string; duration: string };

  const rawTracks = [
    { name: "Break", url: "/break.wav" },
    { name: "Wobbler", url: "/wobbler.wav" },
    { name: "Derelict", url: "/derelict.wav" },
    { name: "LOVE", url: "/LOVE.m4a.mp4" },
  ];

  // Pre-populate tracks without duration, then fill in async
  let tracks: Track[] = rawTracks.map((t) => ({ ...t, duration: "--:--" }));
  let currentIdx = -1;
  let isPlaying = false;
  let progress = 0;
  let currentTime = "0:00";
  let volume = 0.8;

  let canvasEl: HTMLCanvasElement;
  let heatmapEl: HTMLCanvasElement;
  let playerEl: HTMLElement;

  let audioCtx: AudioContext;
  let analyser: AnalyserNode;
  let analyser2: AnalyserNode;
  let sourceNode: MediaElementAudioSourceNode | null = null;
  let audioEl: HTMLAudioElement;
  let rafId: number;

  const palette = ["#a78bfa", "#60a5fa", "#f472b6", "#34d399", "#fb923c"];
  let colourIdx = 0;

  function hexToRgb(h: string) {
    return {
      r: parseInt(h.slice(1, 3), 16),
      g: parseInt(h.slice(3, 5), 16),
      b: parseInt(h.slice(5, 7), 16),
    };
  }
  function lerp(a: number, b: number, t: number) {
    return Math.round(a + (b - a) * t);
  }
  function fmt(s: number) {
    const m = Math.floor(s / 60);
    return `${m}:${Math.floor(s % 60)
      .toString()
      .padStart(2, "0")}`;
  }

  // ── Heatmap ───────────────────────────────────────────────────────────
  function drawHeatmap() {
    if (!analyser2 || !heatmapEl) return;
    const W = heatmapEl.offsetWidth;
    const H = heatmapEl.offsetHeight;
    const ctx = heatmapEl.getContext("2d")!;
    const buf = new Uint8Array(analyser2.frequencyBinCount);
    analyser2.getByteFrequencyData(buf);
    const img = ctx.getImageData(2, 0, W - 2, H);
    ctx.putImageData(img, 0, 0);
    ctx.clearRect(W - 2, 0, 2, H);
    const bins = buf.length;
    const colH = H / bins;
    for (let i = 0; i < bins; i++) {
      const val = buf[bins - 1 - i];
      if (val < 4) continue;
      const t = val / 255;
      let r, g, b;
      if (t < 0.25) {
        r = lerp(10, 100, t / 0.25);
        g = lerp(10, 30, t / 0.25);
        b = lerp(30, 180, t / 0.25);
      } else if (t < 0.5) {
        const tt = (t - 0.25) / 0.25;
        r = lerp(100, 167, tt);
        g = lerp(30, 100, tt);
        b = lerp(180, 250, tt);
      } else if (t < 0.75) {
        const tt = (t - 0.5) / 0.25;
        r = lerp(167, 244, tt);
        g = lerp(100, 114, tt);
        b = lerp(250, 182, tt);
      } else {
        const tt = (t - 0.75) / 0.25;
        r = lerp(244, 255, tt);
        g = lerp(114, 220, tt);
        b = lerp(182, 100, tt);
      }
      ctx.fillStyle = `rgba(${r},${g},${b},${0.7 + t * 0.3})`;
      ctx.fillRect(W - 2, i * colH, 2, Math.ceil(colH) + 1);
    }
  }

  // ── Oscilloscope ──────────────────────────────────────────────────────
  function draw() {
    rafId = requestAnimationFrame(draw);
    if (!analyser || !canvasEl) return;
    drawHeatmap();

    const W = canvasEl.offsetWidth;
    const H = canvasEl.offsetHeight;
    const ctx = canvasEl.getContext("2d")!;
    const buf = new Uint8Array(analyser.fftSize);
    analyser.getByteTimeDomainData(buf);

    ctx.fillStyle = "rgba(10,10,12,0.18)";
    ctx.fillRect(0, 0, W, H);

    let energy = 0;
    for (let i = 0; i < buf.length; i++) energy += Math.abs(buf[i] - 128);
    energy /= buf.length;
    if (energy > 6) colourIdx = (colourIdx + 0.012) % palette.length;

    const from = hexToRgb(palette[Math.floor(colourIdx) % palette.length]);
    const to = hexToRgb(palette[Math.ceil(colourIdx) % palette.length]);
    const t = colourIdx % 1;
    const col = `rgb(${lerp(from.r, to.r, t)},${lerp(from.g, to.g, t)},${lerp(from.b, to.b, t)})`;

    ctx.shadowBlur = 18;
    ctx.shadowColor = col;

    const sliceW = W / (buf.length - 1);

    ctx.beginPath();
    ctx.lineWidth = 2;
    ctx.strokeStyle = col;
    ctx.lineJoin = "round";
    ctx.lineCap = "round";
    let x = 0;
    for (let i = 0; i < buf.length; i++) {
      const y = ((buf[i] / 128.0) * H) / 2;
      if (i === 0) {
        ctx.moveTo(x, y);
      } else {
        const px = x - sliceW,
          py = ((buf[i - 1] / 128.0) * H) / 2,
          cx = (px + x) / 2;
        ctx.bezierCurveTo(cx, py, cx, y, x, y);
      }
      x += sliceW;
    }
    ctx.stroke();

    ctx.save();
    ctx.translate(0, H);
    ctx.scale(1, -1);
    ctx.globalAlpha = 0.3;
    ctx.beginPath();
    ctx.strokeStyle = col;
    ctx.lineJoin = "round";
    ctx.lineCap = "round";
    x = 0;
    for (let i = 0; i < buf.length; i++) {
      const y = ((buf[i] / 128.0) * H) / 2;
      if (i === 0) {
        ctx.moveTo(x, y);
      } else {
        const px = x - sliceW,
          py = ((buf[i - 1] / 128.0) * H) / 2,
          cx = (px + x) / 2;
        ctx.bezierCurveTo(cx, py, cx, y, x, y);
      }
      x += sliceW;
    }
    ctx.stroke();
    ctx.restore();
    ctx.globalAlpha = 1;
    ctx.shadowBlur = 0;
  }

  // ── Audio ─────────────────────────────────────────────────────────────
  function setupAudio() {
    if (!audioCtx) {
      audioCtx = new AudioContext();
      analyser = audioCtx.createAnalyser();
      analyser.fftSize = 4096;
      analyser.smoothingTimeConstant = 0.92;
      analyser.connect(audioCtx.destination);
      analyser2 = audioCtx.createAnalyser();
      analyser2.fftSize = 1024;
      analyser2.smoothingTimeConstant = 0.8;
    }
    if (sourceNode) {
      sourceNode.disconnect();
      sourceNode = null;
    }
    sourceNode = audioCtx.createMediaElementSource(audioEl);
    sourceNode.connect(analyser);
    sourceNode.connect(analyser2);
  }

  async function playTrack(idx: number) {
    if (idx < 0 || idx >= tracks.length) return;
    currentIdx = idx;
    audioEl.src = tracks[idx].url;
    audioEl.volume = volume;
    if (!sourceNode) setupAudio();
    if (audioCtx.state === "suspended") await audioCtx.resume();
    await audioEl.play();
    isPlaying = true;
    gsap.fromTo(
      playerEl,
      { scale: 0.97, opacity: 0.7 },
      { scale: 1, opacity: 1, duration: 0.4, ease: "back.out(1.5)" },
    );
  }

  function togglePlay() {
    if (currentIdx < 0 && tracks.length) {
      playTrack(0);
      return;
    }
    if (isPlaying) {
      audioEl.pause();
      isPlaying = false;
    } else {
      audioEl.play();
      isPlaying = true;
    }
  }

  function prev() {
    playTrack((currentIdx - 1 + tracks.length) % tracks.length);
  }
  function next() {
    playTrack((currentIdx + 1) % tracks.length);
  }

  function seek(e: MouseEvent) {
    const ratio = e.offsetX / (e.currentTarget as HTMLElement).clientWidth;
    if (audioEl.duration) audioEl.currentTime = ratio * audioEl.duration;
  }

  function resizeCanvas() {
    if (canvasEl) {
      canvasEl.width = canvasEl.offsetWidth * window.devicePixelRatio;
      canvasEl.height = canvasEl.offsetHeight * window.devicePixelRatio;
      canvasEl
        .getContext("2d")!
        .scale(window.devicePixelRatio, window.devicePixelRatio);
    }
    if (heatmapEl) {
      heatmapEl.width = heatmapEl.offsetWidth;
      heatmapEl.height = heatmapEl.offsetHeight;
    }
  }

  onMount(() => {
    audioEl = new Audio();
    audioEl.crossOrigin = "anonymous";

    // Load durations and update in place
    rawTracks.forEach(({ url }, i) => {
      const tmp = new Audio(url);
      tmp.addEventListener("loadedmetadata", () => {
        tracks[i] = { ...tracks[i], duration: fmt(tmp.duration) };
        tracks = [...tracks];
      });
    });

    audioEl.addEventListener("timeupdate", () => {
      progress = audioEl.duration
        ? (audioEl.currentTime / audioEl.duration) * 100
        : 0;
      currentTime = fmt(audioEl.currentTime);
    });
    audioEl.addEventListener("ended", () => next());

    resizeCanvas();
    window.addEventListener("resize", resizeCanvas);
    draw();

    return () => {
      cancelAnimationFrame(rafId);
      window.removeEventListener("resize", resizeCanvas);
      audioEl.pause();
      audioCtx?.close();
    };
  });
</script>

<svelte:head>
  <title>Music — James Doel</title>
</svelte:head>

<!-- ── VISUALISERS + PLAYER ─────────────────────────────────────────────── -->
<section class="max-w-5xl mx-auto px-6 pt-24 pb-24 w-full">
  <!-- Oscilloscope -->
  <div
    class="relative w-full rounded-2xl overflow-hidden border border-border mb-4"
    style="height:280px; background:#0a0a0c;"
  >
    <canvas bind:this={canvasEl} class="w-full h-full"></canvas>
    {#if currentIdx < 0}
      <div
        class="absolute inset-0 flex items-center justify-center pointer-events-none"
      >
        <p class="text-muted text-sm tracking-widest uppercase">
          Select a track to begin
        </p>
      </div>
    {:else}
      <div class="absolute bottom-4 left-5 pointer-events-none">
        <p class="text-xs tracking-[0.2em] uppercase text-accent mb-0.5">
          Now Playing
        </p>
        <p class="text-white font-serif text-lg">{tracks[currentIdx]?.name}</p>
      </div>
    {/if}
  </div>

  <!-- Frequency heatmap -->
  <div
    class="relative w-full rounded-2xl overflow-hidden border border-border mb-8"
    style="height:160px; background:#0a0a0c;"
  >
    <canvas bind:this={heatmapEl} class="w-full h-full"></canvas>
    <div
      class="absolute inset-y-0 right-3 flex flex-col justify-between py-2 pointer-events-none"
    >
      <span class="text-[10px] text-muted font-mono">High</span>
      <span class="text-[10px] text-muted font-mono">Mid</span>
      <span class="text-[10px] text-muted font-mono">Bass</span>
    </div>
    <div class="absolute top-2 left-4 pointer-events-none">
      <p class="text-[10px] tracking-[0.2em] uppercase text-muted">
        Frequency Heatmap
      </p>
    </div>
    {#if currentIdx < 0}
      <div
        class="absolute inset-0 flex items-center justify-center pointer-events-none"
      >
        <p class="text-muted text-xs tracking-widest uppercase">
          Waiting for audio…
        </p>
      </div>
    {/if}
  </div>

  <!-- Player controls -->
  <div
    bind:this={playerEl}
    class="rounded-2xl border border-border bg-card p-6 mb-8"
  >
    <div
      class="w-full h-1 bg-border rounded-full mb-4 cursor-pointer"
      on:click={seek}
    >
      <div
        class="h-full rounded-full transition-all duration-100"
        style="width:{progress}%; background: linear-gradient(90deg, #a78bfa, #60a5fa);"
      ></div>
    </div>
    <div class="flex items-center justify-between mb-5">
      <span class="text-muted text-xs font-mono">{currentTime}</span>
      <span class="text-muted text-xs font-mono"
        >{currentIdx >= 0 ? tracks[currentIdx]?.duration : "0:00"}</span
      >
    </div>
    <div class="flex items-center justify-center gap-6">
      <button
        on:click={prev}
        class="w-10 h-10 flex items-center justify-center rounded-full border border-border
                     text-muted hover:text-white hover:border-accent transition-all duration-200"
      >
        <svg viewBox="0 0 24 24" fill="currentColor" class="w-4 h-4"
          ><path d="M6 6h2v12H6zm3.5 6 8.5 6V6z" /></svg
        >
      </button>
      <button
        on:click={togglePlay}
        class="w-14 h-14 flex items-center justify-center rounded-full text-white
                     hover:scale-105 hover:shadow-[0_0_24px_rgba(167,139,250,0.4)] transition-all duration-200"
        style="background: linear-gradient(135deg, #a78bfa, #60a5fa);"
      >
        {#if isPlaying}
          <svg viewBox="0 0 24 24" fill="currentColor" class="w-6 h-6"
            ><path d="M6 19h4V5H6v14zm8-14v14h4V5h-4z" /></svg
          >
        {:else}
          <svg viewBox="0 0 24 24" fill="currentColor" class="w-6 h-6"
            ><path d="M8 5v14l11-7z" /></svg
          >
        {/if}
      </button>
      <button
        on:click={next}
        class="w-10 h-10 flex items-center justify-center rounded-full border border-border
                     text-muted hover:text-white hover:border-accent transition-all duration-200"
      >
        <svg viewBox="0 0 24 24" fill="currentColor" class="w-4 h-4"
          ><path
            d="M6 18l8.5-6L6 6v12zm2.5-6 5.5 4-5.5 4V6z M16 6h2v12h-2z"
          /></svg
        >
      </button>
    </div>
    <div class="flex items-center gap-3 mt-5 justify-center">
      <svg
        viewBox="0 0 24 24"
        fill="currentColor"
        class="w-4 h-4 text-muted shrink-0"
      >
        <path
          d="M3 9v6h4l5 5V4L7 9H3zm13.5 3A4.5 4.5 0 0 0 14 7.97v8.05c1.48-.73 2.5-2.25 2.5-4.02z"
        />
      </svg>
      <input
        type="range"
        min="0"
        max="1"
        step="0.01"
        bind:value={volume}
        on:input={() => {
          if (audioEl) audioEl.volume = volume;
        }}
        class="w-28 accent-accent cursor-pointer"
      />
    </div>
  </div>

  <!-- Track selection grid -->
  <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
    {#each tracks as track, i}
      {@const active = currentIdx === i}
      <button
        on:click={() => playTrack(i)}
        class="relative group rounded-2xl border p-5 text-left transition-all duration-300 overflow-hidden
                     {active
          ? 'border-accent bg-accent/5 shadow-[0_0_28px_rgba(167,139,250,0.15)]'
          : 'border-border bg-card hover:border-accent/50 hover:shadow-[0_0_18px_rgba(167,139,250,0.08)]'}"
      >
        <!-- Animated background blob when active -->
        {#if active}
          <div
            class="absolute -top-8 -right-8 w-32 h-32 rounded-full blur-2xl pointer-events-none"
            style="background: radial-gradient(circle, rgba(167,139,250,0.2), transparent 70%);"
          ></div>
        {/if}

        <!-- Top row: number + playing indicator -->
        <div class="flex items-center justify-between mb-4">
          <span class="text-xs font-mono text-muted">
            {String(i + 1).padStart(2, "0")}
          </span>
          {#if active && isPlaying}
            <!-- Animated bars -->
            <span class="flex gap-0.5 items-end h-4">
              {#each [1, 2, 3, 2] as h, j}
                <span
                  class="w-0.5 rounded-full bg-accent animate-bounce"
                  style="height:{h * 4}px; animation-delay:{j * 0.1}s;"
                ></span>
              {/each}
            </span>
          {:else if active}
            <!-- Paused icon -->
            <svg
              viewBox="0 0 24 24"
              fill="currentColor"
              class="w-3.5 h-3.5 text-accent"
            >
              <path d="M6 19h4V5H6v14zm8-14v14h4V5h-4z" />
            </svg>
          {:else}
            <!-- Play icon on hover -->
            <svg
              viewBox="0 0 24 24"
              fill="currentColor"
              class="w-3.5 h-3.5 text-muted opacity-0 group-hover:opacity-100 transition-opacity duration-200"
            >
              <path d="M8 5v14l11-7z" />
            </svg>
          {/if}
        </div>

        <!-- Waveform decoration -->
        <div class="flex items-end gap-px mb-4 h-8">
          {#each Array(28) as _, j}
            {@const h =
              20 + Math.sin((j + i * 7) * 0.8) * 14 + Math.cos(j * 0.4) * 8}
            <div
              class="flex-1 rounded-sm transition-all duration-300"
              style="height:{h}%; background: {active
                ? `rgba(167,139,250,${0.3 + (h / 100) * 0.7})`
                : `rgba(255,255,255,${0.05 + (h / 100) * 0.1})`};"
            ></div>
          {/each}
        </div>

        <!-- Track name -->
        <p
          class="font-serif text-lg leading-tight mb-1 transition-colors duration-200
                  {active
            ? 'text-accent'
            : 'text-white group-hover:text-white'}"
        >
          {track.name}
        </p>

        <!-- Duration -->
        <p class="text-xs font-mono text-muted">{track.duration}</p>
      </button>
    {/each}
  </div>
</section>
