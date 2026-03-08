<script lang="ts">
  import { onMount } from "svelte";
  import * as THREE from "three";

  let mountEl: HTMLDivElement;

  const COLORS: Record<string, number> = {
    R: 0x009b48, // green  +x
    L: 0x0051a8, // blue   -x
    U: 0xffffff, // white  +y
    D: 0xffd500, // yellow -y
    F: 0xff5722, // orange +z
    B: 0xc41e3a, // red    -z
  };

  const moves = [
    { label: "U", key: "u" },
    { label: "U'", key: "U" },
    { label: "R", key: "r" },
    { label: "R'", key: "R" },
    { label: "F", key: "f" },
    { label: "F'", key: "F" },
    { label: "L", key: "l" },
    { label: "L'", key: "L" },
    { label: "D", key: "d" },
    { label: "D'", key: "D" },
    { label: "B", key: "b" },
    { label: "B'", key: "B" },
  ];

  type Move = { axis: "x" | "y" | "z"; layer: number; dir: number };
  const moveHistory: Move[] = [];

  function triggerMove(
    axis: "x" | "y" | "z",
    layer: number,
    dir: number,
    record = true,
  ) {
    if (record) moveHistory.push({ axis, layer, dir });
    window.dispatchEvent(
      new CustomEvent("rubik-move", { detail: { axis, layer, dir } }),
    );
  }

  function scramble() {
    moveHistory.length = 0;
    const scrambleMoves: Move[] = [
      { axis: "y", layer: 1, dir: -1 },
      { axis: "y", layer: -1, dir: 1 },
      { axis: "x", layer: 1, dir: -1 },
      { axis: "x", layer: -1, dir: 1 },
      { axis: "z", layer: 1, dir: -1 },
      { axis: "z", layer: -1, dir: 1 },
    ];
    let i = 0;
    const count = 20;
    const next = () => {
      if (i >= count) return;
      const m = scrambleMoves[Math.floor(Math.random() * scrambleMoves.length)];
      triggerMove(m.axis, m.layer, m.dir);
      i++;
      setTimeout(next, 320);
    };
    next();
  }

  function solve() {
    if (moveHistory.length === 0) return;
    const toUndo = [...moveHistory].reverse();
    moveHistory.length = 0;
    let i = 0;
    const next = () => {
      if (i >= toUndo.length) return;
      const m = toUndo[i];
      triggerMove(m.axis, m.layer, -m.dir, false);
      i++;
      setTimeout(next, 320);
    };
    next();
  }

  function triggerKey(key: string) {
    const map: Record<string, Move> = {
      u: { axis: "y", layer: 1, dir: -1 },
      U: { axis: "y", layer: 1, dir: 1 },
      d: { axis: "y", layer: -1, dir: 1 },
      D: { axis: "y", layer: -1, dir: -1 },
      f: { axis: "z", layer: 1, dir: -1 },
      F: { axis: "z", layer: 1, dir: 1 },
      b: { axis: "z", layer: -1, dir: 1 },
      B: { axis: "z", layer: -1, dir: -1 },
      l: { axis: "x", layer: -1, dir: 1 },
      L: { axis: "x", layer: -1, dir: -1 },
      r: { axis: "x", layer: 1, dir: -1 },
      R: { axis: "x", layer: 1, dir: 1 },
    };
    const m = map[key];
    if (m) triggerMove(m.axis, m.layer, m.dir);
  }

  onMount(async () => {
    await new Promise((r) => setTimeout(r, 50));

    const W = mountEl.clientWidth || 600;
    const H = mountEl.clientHeight || 400;

    // ── Renderer ──────────────────────────────────────────────────
    const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(W, H);
    renderer.shadowMap.enabled = true;
    mountEl.appendChild(renderer.domElement);

    // ── Scene & Camera ────────────────────────────────────────────
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(40, W / H, 0.1, 100);
    camera.position.set(6, 5, 7);
    camera.lookAt(0, 0, 0);

    // ── Lighting ──────────────────────────────────────────────────
    scene.add(new THREE.AmbientLight(0xffffff, 0.6));
    const sun = new THREE.DirectionalLight(0xffffff, 1.4);
    sun.position.set(8, 12, 10);
    sun.castShadow = true;
    scene.add(sun);
    const fill = new THREE.DirectionalLight(0x6688cc, 0.5);
    fill.position.set(-8, 2, -6);
    scene.add(fill);
    const rim = new THREE.DirectionalLight(0xffffff, 0.2);
    rim.position.set(0, -8, -6);
    scene.add(rim);

    // ── Build cube ────────────────────────────────────────────────
    const SIZE = 1.0;
    const GAP = 1.05;
    const cubeGroup = new THREE.Group();
    scene.add(cubeGroup);
    const initQ = new THREE.Quaternion().setFromEuler(
      new THREE.Euler(0.4, 0.6, 0),
    );
    cubeGroup.quaternion.copy(initQ);

    const cubies: THREE.Group[] = [];

    function makeMat(color: number | null) {
      return color != null
        ? new THREE.MeshStandardMaterial({
            color,
            roughness: 0.3,
            metalness: 0.05,
          })
        : new THREE.MeshStandardMaterial({
            color: 0x0a0a0a,
            roughness: 0.9,
            metalness: 0.1,
          });
    }

    for (let x = -1; x <= 1; x++) {
      for (let y = -1; y <= 1; y++) {
        for (let z = -1; z <= 1; z++) {
          const g = new THREE.Group();
          g.position.set(x * GAP, y * GAP, z * GAP);

          // Black body
          const body = new THREE.Mesh(
            new THREE.BoxGeometry(SIZE * 0.92, SIZE * 0.92, SIZE * 0.92),
            new THREE.MeshStandardMaterial({ color: 0x111111, roughness: 0.8 }),
          );
          g.add(body);

          // Stickers
          const stickers: [
            number,
            number,
            number,
            number,
            number,
            number,
            string,
          ][] = [
            [SIZE / 2 + 0.001, 0, 0, 0, Math.PI / 2, 0, x === 1 ? "R" : ""],
            [-SIZE / 2 - 0.001, 0, 0, 0, -Math.PI / 2, 0, x === -1 ? "L" : ""],
            [0, SIZE / 2 + 0.001, 0, -Math.PI / 2, 0, 0, y === 1 ? "U" : ""],
            [0, -SIZE / 2 - 0.001, 0, Math.PI / 2, 0, 0, y === -1 ? "D" : ""],
            [0, 0, SIZE / 2 + 0.001, 0, 0, 0, z === 1 ? "F" : ""],
            [0, 0, -SIZE / 2 - 0.001, 0, Math.PI, 0, z === -1 ? "B" : ""],
          ];

          stickers.forEach(([px, py, pz, rx, ry, rz, face]) => {
            if (!face) return;
            const s = new THREE.Mesh(
              new THREE.PlaneGeometry(SIZE * 0.82, SIZE * 0.82),
              makeMat(COLORS[face] ?? null),
            );
            s.position.set(px, py, pz);
            s.rotation.set(rx, ry, rz);
            g.add(s);
          });

          cubies.push(g);
          cubeGroup.add(g);
        }
      }
    }

    // ── Face rotation ─────────────────────────────────────────────
    let isAnimating = false;

    function getSlice(axis: "x" | "y" | "z", val: number) {
      return cubies.filter((c) => Math.round(c.position[axis] / GAP) === val);
    }

    function rotateFace(axis: "x" | "y" | "z", layer: number, dir: number) {
      if (isAnimating) return;
      isAnimating = true;

      const slice = getSlice(axis, layer);
      const pivot = new THREE.Group();
      cubeGroup.add(pivot);
      slice.forEach((c) => pivot.attach(c));

      const total = (dir * Math.PI) / 2;
      const duration = 280;
      const start = performance.now();

      function step(now: number) {
        const t = Math.min((now - start) / duration, 1);
        const e = t < 0.5 ? 2 * t * t : -1 + (4 - 2 * t) * t;
        pivot.rotation[axis] = total * e;
        renderer.render(scene, camera);
        if (t < 1) {
          requestAnimationFrame(step);
          return;
        }

        pivot.rotation[axis] = total;
        slice.forEach((c) => {
          cubeGroup.attach(c);
          c.position.set(
            Math.round(c.position.x / GAP) * GAP,
            Math.round(c.position.y / GAP) * GAP,
            Math.round(c.position.z / GAP) * GAP,
          );
        });
        cubeGroup.remove(pivot);
        isAnimating = false;
      }
      requestAnimationFrame(step);
    }

    // ── Key map ───────────────────────────────────────────────────
    const keyMap: Record<string, () => void> = {
      u: () => rotateFace("y", 1, -1),
      U: () => rotateFace("y", 1, 1),
      d: () => rotateFace("y", -1, 1),
      D: () => rotateFace("y", -1, -1),
      f: () => rotateFace("z", 1, -1),
      F: () => rotateFace("z", 1, 1),
      b: () => rotateFace("z", -1, 1),
      B: () => rotateFace("z", -1, -1),
      l: () => rotateFace("x", -1, 1),
      L: () => rotateFace("x", -1, -1),
      r: () => rotateFace("x", 1, -1),
      R: () => rotateFace("x", 1, 1),
    };
    // Listen for move events
    const onRubikMove = (e: Event) => {
      const { axis, layer, dir } = (e as CustomEvent).detail;
      rotateFace(axis, layer, dir);
    };
    window.addEventListener("rubik-move", onRubikMove);

    const onKey = (e: KeyboardEvent) => {
      const key = e.shiftKey ? e.key.toUpperCase() : e.key.toLowerCase();
      const map: Record<string, () => void> = {
        u: () => triggerMove("y", 1, -1),
        U: () => triggerMove("y", 1, 1),
        d: () => triggerMove("y", -1, 1),
        D: () => triggerMove("y", -1, -1),
        f: () => triggerMove("z", 1, -1),
        F: () => triggerMove("z", 1, 1),
        b: () => triggerMove("z", -1, 1),
        B: () => triggerMove("z", -1, -1),
        l: () => triggerMove("x", -1, 1),
        L: () => triggerMove("x", -1, -1),
        r: () => triggerMove("x", 1, -1),
        R: () => triggerMove("x", 1, 1),
      };
      map[key]?.();
    };
    window.addEventListener("keydown", onKey);

    // ── Drag orbit (trackball) ────────────────────────────────────
    let dragging = false;
    let autoRotate = true;
    let prev = { x: 0, y: 0 };

    const getPos = (e: MouseEvent | Touch) => ({
      x: (e.clientX / W) * 2 - 1,
      y: -(e.clientY / H) * 2 + 1,
    });

    renderer.domElement.addEventListener("mousedown", (e) => {
      dragging = true;
      autoRotate = false;
      prev = { x: e.clientX, y: e.clientY };
    });
    renderer.domElement.addEventListener(
      "touchstart",
      (e) => {
        dragging = true;
        autoRotate = false;
        prev = { x: e.touches[0].clientX, y: e.touches[0].clientY };
      },
      { passive: true },
    );
    window.addEventListener("mouseup", () => (dragging = false));
    window.addEventListener("touchend", () => (dragging = false));

    window.addEventListener("mousemove", (e) => {
      if (!dragging) return;
      const dx = e.clientX - prev.x;
      const dy = e.clientY - prev.y;
      const speed = 0.01;
      // Rotate around world Y for horizontal drag, world X for vertical
      const qY = new THREE.Quaternion().setFromAxisAngle(
        new THREE.Vector3(0, 1, 0),
        dx * speed,
      );
      const qX = new THREE.Quaternion().setFromAxisAngle(
        new THREE.Vector3(1, 0, 0),
        dy * speed,
      );
      cubeGroup.quaternion.premultiply(qY).premultiply(qX);
      prev = { x: e.clientX, y: e.clientY };
    });

    window.addEventListener(
      "touchmove",
      (e) => {
        if (!dragging) return;
        const dx = e.touches[0].clientX - prev.x;
        const dy = e.touches[0].clientY - prev.y;
        const speed = 0.01;
        const qY = new THREE.Quaternion().setFromAxisAngle(
          new THREE.Vector3(0, 1, 0),
          dx * speed,
        );
        const qX = new THREE.Quaternion().setFromAxisAngle(
          new THREE.Vector3(1, 0, 0),
          dy * speed,
        );
        cubeGroup.quaternion.premultiply(qY).premultiply(qX);
        prev = { x: e.touches[0].clientX, y: e.touches[0].clientY };
      },
      { passive: true },
    );

    // ── Render loop ───────────────────────────────────────────────
    let raf: number;
    const tick = () => {
      raf = requestAnimationFrame(tick);
      if (autoRotate) {
        const q = new THREE.Quaternion().setFromAxisAngle(
          new THREE.Vector3(0, 1, 0),
          0.005,
        );
        cubeGroup.quaternion.premultiply(q);
      }
      renderer.render(scene, camera);
    };
    tick();

    // ── Resize ────────────────────────────────────────────────────
    const onResize = () => {
      const w = mountEl?.clientWidth ?? W;
      const h = mountEl?.clientHeight ?? H;
      camera.aspect = w / h;
      camera.updateProjectionMatrix();
      renderer.setSize(w, h);
    };
    window.addEventListener("resize", onResize);

    return () => {
      cancelAnimationFrame(raf);
      renderer.dispose();
      window.removeEventListener("keydown", onKey);
      window.removeEventListener("rubik-move", onRubikMove);
      window.removeEventListener("resize", onResize);
      if (mountEl?.contains(renderer.domElement))
        mountEl.removeChild(renderer.domElement);
    };
  });
</script>

<div class="flex flex-col items-center w-full">
  <p class="text-xs tracking-[0.2em] uppercase text-accent mb-2">
    Interactive Demo
  </p>
  <h2 class="font-serif text-3xl mb-1">Rubik's Cube</h2>
  <p class="text-muted text-sm mb-6">
    Drag to rotate · Use buttons or keyboard to move faces
  </p>

  <div
    bind:this={mountEl}
    class="w-full rounded-2xl overflow-hidden cursor-grab"
    style="max-width:640px; height:420px; background:#0a0a0c;"
  ></div>

  <div
    class="flex flex-wrap gap-2 justify-center mt-6"
    style="max-width:640px;"
  >
    {#each moves as { label, key }}
      <button
        on:click={() => triggerKey(key)}
        class="px-3 py-1.5 rounded-lg text-sm border border-border bg-card
          text-[#e8e8e8] hover:border-accent transition-all duration-200 font-mono cursor-pointer"
      >
        {label}
      </button>
    {/each}
  </div>

  <div class="flex gap-3 mt-4">
    <button
      on:click={scramble}
      class="w-32 px-6 py-2.5 rounded-lg text-sm font-medium text-white
        transition-all duration-200 hover:opacity-85 hover:-translate-y-px"
      style="background: linear-gradient(135deg, #a78bfa, #60a5fa);"
    >
      Scramble
    </button>
    <button
      on:click={solve}
      class="w-32 px-6 py-2.5 rounded-lg text-sm font-medium text-white
        transition-all duration-200 hover:opacity-85 hover:-translate-y-px"
      style="background: linear-gradient(135deg, #60a5fa, #34d399);"
    >
      Solve
    </button>
  </div>
</div>
