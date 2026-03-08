<script lang="ts">
  import { onMount } from "svelte";
  import * as THREE from "three";

  let mountEl: HTMLDivElement;

  onMount(async () => {
    await new Promise((r) => setTimeout(r, 50));

    const W = mountEl.clientWidth || 600;
    const H = mountEl.clientHeight || 500;

    // ── Renderer ──────────────────────────────────────────────────
    const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(W, H);
    renderer.toneMapping = THREE.ACESFilmicToneMapping;
    renderer.toneMappingExposure = 1.5;
    mountEl.appendChild(renderer.domElement);

    // ── Scene & Camera ────────────────────────────────────────────
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(45, W / H, 0.1, 100);
    camera.position.set(0, 1, 5);
    camera.lookAt(0, 0, 0);

    // ── Lights ────────────────────────────────────────────────────
    scene.add(new THREE.AmbientLight(0xffffff, 0.3));

    const lights: [number, number, number, number][] = [
      [4, 4, 4, 0xffffff],
      [-4, 2, -4, 0xa78bfa],
      [0, -4, 2, 0x60a5fa],
      [2, 6, -2, 0xffffff],
    ];
    lights.forEach(([x, y, z, col]) => {
      const l = new THREE.PointLight(col, 2, 20);
      l.position.set(x, y, z);
      scene.add(l);
    });

    // ── Diamond geometry ──────────────────────────────────────────
    // Build a custom diamond shape using ConeGeometry top + inverted bottom
    const group = new THREE.Group();
    scene.add(group);

    const mat = new THREE.MeshPhysicalMaterial({
      color: 0xffffff,
      metalness: 0,
      roughness: 0,
      transmission: 0.95, // glass-like transmission
      thickness: 1.5,
      ior: 2.42, // diamond IOR
      reflectivity: 1,
      transparent: true,
      opacity: 0.9,
      envMapIntensity: 2,
      clearcoat: 1,
      clearcoatRoughness: 0,
      attenuationColor: new THREE.Color(0xaaddff),
      attenuationDistance: 2,
    });

    // Crown (top) — wide flat cone
    const crown = new THREE.Mesh(new THREE.ConeGeometry(1.2, 0.7, 8, 1), mat);
    crown.position.y = 0.2;
    group.add(crown);

    // Table (flat top face) — disc
    const table = new THREE.Mesh(
      new THREE.CylinderGeometry(0.75, 0.75, 0.02, 8),
      mat,
    );
    table.position.y = 0.56;
    group.add(table);

    // Pavilion (bottom) — pointed cone
    const pavilion = new THREE.Mesh(
      new THREE.ConeGeometry(1.2, 1.6, 8, 1),
      mat,
    );
    pavilion.rotation.z = Math.PI;
    pavilion.position.y = -0.6;
    group.add(pavilion);

    // Girdle (thin band at widest point)
    const girdle = new THREE.Mesh(
      new THREE.CylinderGeometry(1.21, 1.21, 0.08, 8),
      new THREE.MeshPhysicalMaterial({
        color: 0xffffff,
        metalness: 0.1,
        roughness: 0,
        transparent: true,
        opacity: 0.6,
      }),
    );
    girdle.position.y = 0.18;
    group.add(girdle);

    // ── Floating sparkles ─────────────────────────────────────────
    const sparkleCount = 120;
    const sparkPositions = new Float32Array(sparkleCount * 3);
    const sparkSpeeds = new Float32Array(sparkleCount);

    for (let i = 0; i < sparkleCount; i++) {
      const i3 = i * 3;
      const r = 1.5 + Math.random() * 2;
      const theta = Math.random() * Math.PI * 2;
      const phi = Math.random() * Math.PI;
      sparkPositions[i3] = r * Math.sin(phi) * Math.cos(theta);
      sparkPositions[i3 + 1] = r * Math.cos(phi);
      sparkPositions[i3 + 2] = r * Math.sin(phi) * Math.sin(theta);
      sparkSpeeds[i] = 0.3 + Math.random() * 0.7;
    }

    const sparkGeo = new THREE.BufferGeometry();
    sparkGeo.setAttribute(
      "position",
      new THREE.BufferAttribute(sparkPositions, 3),
    );
    const sparkMat = new THREE.PointsMaterial({
      color: 0xffffff,
      size: 0.06,
      transparent: true,
      opacity: 0.8,
      sizeAttenuation: true,
    });
    const sparkles = new THREE.Points(sparkGeo, sparkMat);
    scene.add(sparkles);

    // ── Mouse drag ────────────────────────────────────────────────
    let dragging = false;
    let prev = { x: 0, y: 0 };

    renderer.domElement.addEventListener("mousedown", (e) => {
      dragging = true;
      prev = { x: e.clientX, y: e.clientY };
    });
    window.addEventListener("mouseup", () => (dragging = false));
    window.addEventListener("mousemove", (e) => {
      if (!dragging) return;
      const dx = e.clientX - prev.x;
      const dy = e.clientY - prev.y;
      const qY = new THREE.Quaternion().setFromAxisAngle(
        new THREE.Vector3(0, 1, 0),
        dx * 0.01,
      );
      const qX = new THREE.Quaternion().setFromAxisAngle(
        new THREE.Vector3(1, 0, 0),
        dy * 0.01,
      );
      group.quaternion.premultiply(qY).premultiply(qX);
      prev = { x: e.clientX, y: e.clientY };
    });

    // ── Render loop ───────────────────────────────────────────────
    let raf: number;
    let time = 0;

    const tick = () => {
      raf = requestAnimationFrame(tick);
      time += 0.01;

      // Auto rotate
      if (!dragging) {
        const q = new THREE.Quaternion().setFromAxisAngle(
          new THREE.Vector3(0, 1, 0),
          0.006,
        );
        group.quaternion.premultiply(q);
      }

      // Bob up and down
      group.position.y = Math.sin(time * 0.8) * 0.1;

      // Sparkle twinkle
      sparkMat.opacity = 0.5 + Math.sin(time * 3) * 0.3;
      sparkles.rotation.y += 0.003;
      sparkles.rotation.x += 0.001;

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
      window.removeEventListener("mouseup", () => (dragging = false));
      window.removeEventListener("mousemove", () => {});
      window.removeEventListener("resize", onResize);
      if (mountEl?.contains(renderer.domElement))
        mountEl.removeChild(renderer.domElement);
    };
  });
</script>

<div class="flex flex-col items-center w-full">
  <p class="text-xs tracking-[0.2em] uppercase text-accent mb-2">Three.js</p>
  <h2 class="font-serif text-3xl mb-1">Diamond</h2>
  <p class="text-muted text-sm mb-6">
    Drag to rotate · Watch the light refract
  </p>

  <div
    bind:this={mountEl}
    class="w-full rounded-2xl overflow-hidden cursor-grab"
    style="height:500px; background: radial-gradient(ellipse at center, #1a1030 0%, #0a0a0c 100%);"
  ></div>
</div>
