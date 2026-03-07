<script lang="ts">
  import '../app.css';
  import Navbar from '$lib/components/NavBar.svelte';
  import Footer from '$lib/components/Footer.svelte';
</script>

<div class="min-h-screen bg-bg flex flex-col">
  <Navbar />
  <main class="flex-1">
    <slot />
  </main>
  <Footer />
</div>