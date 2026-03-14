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
    renderer.toneMappingExposure = 1.6;
    mountEl.appendChild(renderer.domElement);

    // ── Scene & Camera ────────────────────────────────────────────
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(42, W / H, 0.1, 100);
    camera.position.set(0, 0, 9);
    camera.lookAt(0, 0, 0);

    // ── Lights ────────────────────────────────────────────────────
    scene.add(new THREE.AmbientLight(0xffffff, 0.5));
    const lights: [number, number, number, number, number][] = [
      [0xffffff, 5, 8, 6, 20],
      [0xa78bfa, -6, 2, -4, 16],
      [0x60a5fa, 0, -6, 4, 12],
      [0xf472b6, 6, -2, 0, 10],
    ];
    const dynLights = lights.map(([col, x, y, z, i]) => {
      const l = new THREE.PointLight(col, i, 50);
      l.position.set(x, y, z);
      scene.add(l);
      return l;
    });

    // ── DNA config ────────────────────────────────────────────────
    const TURNS = 4; // number of full helix turns
    const STEPS = 120; // points per strand
    const RADIUS = 1.6; // helix radius
    const HEIGHT = 7.0; // total height
    const PAIR_EVERY = 8; // base pair every N steps
    const group = new THREE.Group();
    scene.add(group);

    // Strand colours
    const colA = new THREE.Color(0xa78bfa); // purple
    const colB = new THREE.Color(0x60a5fa); // blue
    const pairColours = [
      new THREE.Color(0xf472b6), // pink
      new THREE.Color(0x34d399), // green
      new THREE.Color(0xffd700), // gold
      new THREE.Color(0xfb923c), // orange
    ];

    // ── Sphere material factory ───────────────────────────────────
    const makeMat = (col: THREE.Color, emissive = 0.15) =>
      new THREE.MeshPhongMaterial({
        color: col,
        emissive: col,
        emissiveIntensity: emissive,
        specular: new THREE.Color(0xffffff),
        shininess: 300,
      });

    // ── Build strands ─────────────────────────────────────────────
    const strandAPoints: THREE.Vector3[] = [];
    const strandBPoints: THREE.Vector3[] = [];

    for (let i = 0; i < STEPS; i++) {
      const t = i / (STEPS - 1);
      const angle = t * TURNS * Math.PI * 2;
      const y = (t - 0.5) * HEIGHT;

      strandAPoints.push(
        new THREE.Vector3(
          RADIUS * Math.cos(angle),
          y,
          RADIUS * Math.sin(angle),
        ),
      );
      strandBPoints.push(
        new THREE.Vector3(
          RADIUS * Math.cos(angle + Math.PI),
          y,
          RADIUS * Math.sin(angle + Math.PI),
        ),
      );
    }

    // Draw strand tube using small spheres
    const nodeGeoA = new THREE.SphereGeometry(0.1, 10, 10);
    const nodeGeoB = new THREE.SphereGeometry(0.1, 10, 10);
    const matA = makeMat(colA);
    const matB = makeMat(colB);

    strandAPoints.forEach((p) => {
      const m = new THREE.Mesh(nodeGeoA, matA);
      m.position.copy(p);
      group.add(m);
    });
    strandBPoints.forEach((p) => {
      const m = new THREE.Mesh(nodeGeoB, matB);
      m.position.copy(p);
      group.add(m);
    });

    // ── Strand backbone tubes ─────────────────────────────────────
    function makeTube(points: THREE.Vector3[], col: THREE.Color) {
      const curve = new THREE.CatmullRomCurve3(points);
      const geo = new THREE.TubeGeometry(curve, STEPS * 3, 0.045, 8, false);
      const mat = new THREE.MeshPhongMaterial({
        color: col,
        emissive: col,
        emissiveIntensity: 0.08,
        specular: new THREE.Color(0xffffff),
        shininess: 200,
        transparent: true,
        opacity: 0.85,
      });
      return new THREE.Mesh(geo, mat);
    }

    group.add(makeTube(strandAPoints, colA));
    group.add(makeTube(strandBPoints, colB));

    // ── Base pairs (rungs) ────────────────────────────────────────
    let pairIdx = 0;
    for (let i = 0; i < STEPS; i += PAIR_EVERY) {
      const pA = strandAPoints[i];
      const pB = strandBPoints[i];
      const col = pairColours[pairIdx % pairColours.length];
      pairIdx++;

      // Rung cylinder
      const dir = new THREE.Vector3().subVectors(pB, pA);
      const len = dir.length();
      const mid = new THREE.Vector3().addVectors(pA, pB).multiplyScalar(0.5);

      const rungGeo = new THREE.CylinderGeometry(0.035, 0.035, len, 8);
      const rungMat = new THREE.MeshPhongMaterial({
        color: col,
        emissive: col,
        emissiveIntensity: 0.2,
        specular: 0xffffff,
        shininess: 200,
        transparent: true,
        opacity: 0.9,
      });
      const rung = new THREE.Mesh(rungGeo, rungMat);
      rung.position.copy(mid);
      rung.quaternion.setFromUnitVectors(
        new THREE.Vector3(0, 1, 0),
        dir.normalize(),
      );
      group.add(rung);

      // End caps (nucleotide spheres)
      const capGeo = new THREE.SphereGeometry(0.16, 12, 12);
      const capMat = makeMat(col, 0.3);
      const capA = new THREE.Mesh(capGeo, capMat);
      const capB = new THREE.Mesh(capGeo, capMat);
      capA.position.copy(pA);
      capB.position.copy(pB);
      group.add(capA, capB);

      // Midpoint sphere (hydrogen bond)
      const bondGeo = new THREE.SphereGeometry(0.09, 8, 8);
      const bondMat = makeMat(new THREE.Color(0xffffff), 0.4);
      const bond = new THREE.Mesh(bondGeo, bondMat);
      bond.position.copy(mid);
      group.add(bond);
    }

    // ── Floating sparkles ─────────────────────────────────────────
    const sparkCount = 180;
    const sparkPos = new Float32Array(sparkCount * 3);
    const sparkCol = new Float32Array(sparkCount * 3);
    const hues = [0.7, 0.6, 0.85, 0.14, 0.5];
    for (let i = 0; i < sparkCount; i++) {
      const r = 3 + Math.random() * 3;
      const theta = Math.random() * Math.PI * 2;
      const phi = Math.random() * Math.PI;
      sparkPos[i * 3] = r * Math.sin(phi) * Math.cos(theta);
      sparkPos[i * 3 + 1] = r * Math.cos(phi);
      sparkPos[i * 3 + 2] = r * Math.sin(phi) * Math.sin(theta);
      const c = new THREE.Color().setHSL(hues[i % hues.length], 0.9, 0.8);
      sparkCol[i * 3] = c.r;
      sparkCol[i * 3 + 1] = c.g;
      sparkCol[i * 3 + 2] = c.b;
    }
    const sparkGeo = new THREE.BufferGeometry();
    sparkGeo.setAttribute("position", new THREE.BufferAttribute(sparkPos, 3));
    sparkGeo.setAttribute("color", new THREE.BufferAttribute(sparkCol, 3));
    const sparkMat = new THREE.PointsMaterial({
      size: 0.06,
      transparent: true,
      opacity: 0.8,
      vertexColors: true,
      sizeAttenuation: true,
      blending: THREE.AdditiveBlending,
      depthWrite: false,
    });
    scene.add(new THREE.Points(sparkGeo, sparkMat));

    // ── Drag orbit ────────────────────────────────────────────────
    let dragging = false;
    let prev = { x: 0, y: 0 };
    renderer.domElement.addEventListener("mousedown", (e) => {
      dragging = true;
      prev = { x: e.clientX, y: e.clientY };
    });
    renderer.domElement.addEventListener(
      "touchstart",
      (e) => {
        dragging = true;
        prev = { x: e.touches[0].clientX, y: e.touches[0].clientY };
      },
      { passive: true },
    );
    window.addEventListener("mouseup", () => {
      dragging = false;
    });
    window.addEventListener("touchend", () => {
      dragging = false;
    });
    window.addEventListener("mousemove", (e) => {
      if (!dragging) return;
      const dx = e.clientX - prev.x,
        dy = e.clientY - prev.y;
      group.quaternion
        .premultiply(
          new THREE.Quaternion().setFromAxisAngle(
            new THREE.Vector3(0, 1, 0),
            dx * 0.01,
          ),
        )
        .premultiply(
          new THREE.Quaternion().setFromAxisAngle(
            new THREE.Vector3(1, 0, 0),
            dy * 0.01,
          ),
        );
      prev = { x: e.clientX, y: e.clientY };
    });

    // ── Render loop ───────────────────────────────────────────────
    let raf: number;
    let time = 0;

    const tick = () => {
      raf = requestAnimationFrame(tick);
      time += 0.007;

      if (!dragging) {
        group.quaternion.premultiply(
          new THREE.Quaternion().setFromAxisAngle(
            new THREE.Vector3(0, 1, 0),
            0.004,
          ),
        );
      }

      // Slow vertical drift
      group.position.y = Math.sin(time * 0.4) * 0.2;

      // Cycle light colours
      dynLights.forEach((l, i) => {
        l.color.setHSL((time * 0.06 + i / dynLights.length) % 1, 0.9, 0.6);
        l.intensity = 12 + Math.sin(time * 1.5 + i) * 5;
      });

      sparkMat.opacity = 0.4 + Math.sin(time * 3) * 0.35;

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
  <h2 class="font-serif text-3xl mb-1">DNA Helix</h2>
  <p class="text-muted text-sm mb-6">Drag to rotate · Double helix structure</p>
  <div
    bind:this={mountEl}
    class="w-full rounded-2xl overflow-hidden cursor-grab"
    style="height:500px; background: radial-gradient(ellipse at 50% 30%, #0d0820 0%, #0a0a0c 70%);"
  ></div>
</div>
