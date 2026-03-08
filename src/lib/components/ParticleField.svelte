<script lang="ts">
  import { onMount } from "svelte";
  import * as THREE from "three";

  let mountEl: HTMLDivElement;

  onMount(async () => {
    await new Promise((r) => setTimeout(r, 50));

    const W = mountEl.clientWidth || 800;
    const H = mountEl.clientHeight || 500;

    // ── Renderer ──────────────────────────────────────────────────
    const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(W, H);
    mountEl.appendChild(renderer.domElement);

    // ── Scene & Camera ────────────────────────────────────────────
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(60, W / H, 0.1, 200);
    camera.position.set(0, 0, 50);

    // ── Mouse tracking ────────────────────────────────────────────
    const mouse = new THREE.Vector2(0, 0);
    const target = new THREE.Vector2(0, 0);

    window.addEventListener("mousemove", (e) => {
      const rect = mountEl.getBoundingClientRect();
      mouse.x = ((e.clientX - rect.left) / W) * 2 - 1;
      mouse.y = -((e.clientY - rect.top) / H) * 2 + 1;
    });

    // ── Particles ─────────────────────────────────────────────────
    const COUNT = 3000;
    const positions = new Float32Array(COUNT * 3);
    const velocities = new Float32Array(COUNT * 3);
    const origins = new Float32Array(COUNT * 3);
    const colors = new Float32Array(COUNT * 3);

    const c1 = new THREE.Color(0xa78bfa); // purple
    const c2 = new THREE.Color(0x60a5fa); // blue
    const c3 = new THREE.Color(0xffffff); // white

    for (let i = 0; i < COUNT; i++) {
      const i3 = i * 3;
      const x = (Math.random() - 0.5) * 100;
      const y = (Math.random() - 0.5) * 60;
      const z = (Math.random() - 0.5) * 40;

      positions[i3] = origins[i3] = x;
      positions[i3 + 1] = origins[i3 + 1] = y;
      positions[i3 + 2] = origins[i3 + 2] = z;

      velocities[i3] = (Math.random() - 0.5) * 0.02;
      velocities[i3 + 1] = (Math.random() - 0.5) * 0.02;
      velocities[i3 + 2] = (Math.random() - 0.5) * 0.01;

      // Mix colours
      const t = Math.random();
      const col =
        t < 0.5
          ? c1.clone().lerp(c2, t * 2)
          : c2.clone().lerp(c3, (t - 0.5) * 2);
      colors[i3] = col.r;
      colors[i3 + 1] = col.g;
      colors[i3 + 2] = col.b;
    }

    const geo = new THREE.BufferGeometry();
    geo.setAttribute("position", new THREE.BufferAttribute(positions, 3));
    geo.setAttribute("color", new THREE.BufferAttribute(colors, 3));

    const mat = new THREE.PointsMaterial({
      size: 0.25,
      vertexColors: true,
      transparent: true,
      opacity: 0.85,
      sizeAttenuation: true,
    });

    const particles = new THREE.Points(geo, mat);
    scene.add(particles);

    // ── Flow field (simplex-like using sin/cos) ───────────────────
    let time = 0;

    function flowAngle(
      x: number,
      y: number,
      z: number,
      t: number,
    ): THREE.Vector3 {
      const angle =
        Math.sin(x * 0.05 + t) * Math.cos(y * 0.05 + t) * Math.PI * 2;
      const vx = Math.cos(angle) * 0.04;
      const vy = Math.sin(angle) * 0.04;
      const vz = Math.sin(x * 0.03 + y * 0.03 + t) * 0.02;
      return new THREE.Vector3(vx, vy, vz);
    }

    // ── Mouse repulsion ───────────────────────────────────────────
    const mouseWorld = new THREE.Vector3();

    // ── Render loop ───────────────────────────────────────────────
    let raf: number;

    const tick = () => {
      raf = requestAnimationFrame(tick);
      time += 0.005;

      // Smooth mouse follow
      target.lerp(mouse, 0.05);
      mouseWorld.set(target.x * 40, target.y * 25, 10);

      const pos = geo.attributes.position.array as Float32Array;

      for (let i = 0; i < COUNT; i++) {
        const i3 = i * 3;
        let x = pos[i3],
          y = pos[i3 + 1],
          z = pos[i3 + 2];

        // Flow field
        const flow = flowAngle(x, y, z, time);
        velocities[i3] += flow.x;
        velocities[i3 + 1] += flow.y;
        velocities[i3 + 2] += flow.z;

        // Mouse repulsion
        const dx = x - mouseWorld.x;
        const dy = y - mouseWorld.y;
        const dist = Math.sqrt(dx * dx + dy * dy);
        if (dist < 12) {
          const force = ((12 - dist) / 12) * 0.3;
          velocities[i3] += (dx / dist) * force;
          velocities[i3 + 1] += (dy / dist) * force;
        }

        // Damping
        velocities[i3] *= 0.92;
        velocities[i3 + 1] *= 0.92;
        velocities[i3 + 2] *= 0.92;

        // Spring back to origin
        velocities[i3] += (origins[i3] - x) * 0.003;
        velocities[i3 + 1] += (origins[i3 + 1] - y) * 0.003;
        velocities[i3 + 2] += (origins[i3 + 2] - z) * 0.003;

        pos[i3] += velocities[i3];
        pos[i3 + 1] += velocities[i3 + 1];
        pos[i3 + 2] += velocities[i3 + 2];
      }

      geo.attributes.position.needsUpdate = true;
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
      window.removeEventListener("resize", onResize);
      if (mountEl?.contains(renderer.domElement))
        mountEl.removeChild(renderer.domElement);
    };
  });
</script>

<div class="flex flex-col items-center w-full">
  <p class="text-xs tracking-[0.2em] uppercase text-accent mb-2">Three.js</p>
  <h2 class="font-serif text-3xl mb-1">Particle Field</h2>
  <p class="text-muted text-sm mb-6">
    Move your mouse over the field to interact
  </p>

  <div
    bind:this={mountEl}
    class="w-full rounded-2xl overflow-hidden"
    style="height:500px; background:#0a0a0c; cursor:crosshair;"
  ></div>
</div>
