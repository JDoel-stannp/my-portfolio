<script lang="ts">
  import { page } from '$app/stores';

  interface NavLink {
    href: string;
    label: string;
  }

  const links: NavLink[] = [
    { href: '/',          label: 'Home' },
    { href: '/about',     label: 'About' },
    { href: '/svelte',    label: 'Svelte Dev' },
    { href: '/animation', label: 'Animation' },
    { href: '/music',     label: 'Music' },
    { href: '/contact',   label: 'Contact' },
  ];

  let menuOpen: boolean = false;

  function toggleMenu(): void {
    menuOpen = !menuOpen;
  }

  function closeMenu(): void {
    menuOpen = false;
  }
</script>

<nav class="fixed top-0 left-0 right-0 z-50 bg-bg/80 backdrop-blur-md border-b border-border">
  <div class="max-w-5xl mx-auto px-6 flex items-center justify-between h-16">

    <!-- Logo -->
    <a href="/" class="font-serif text-2xl font-bold tracking-tight">
      JD<span class="text-accent">.</span>
    </a>

    <!-- Desktop links -->
    <ul class="hidden md:flex items-center gap-8 list-none">
      {#each links as { href, label }}
        <li>
          <a
            {href}
            class="text-sm transition-colors duration-200 relative pb-0.5
              after:absolute after:bottom-0 after:left-0 after:h-px after:bg-accent
              after:transition-all after:duration-200
              {$page.url.pathname === href
                ? 'text-white after:w-full'
                : 'text-muted hover:text-white after:w-0 hover:after:w-full'}"
          >
            {label}
          </a>
        </li>
      {/each}
    </ul>

    <!-- Mobile burger -->
    <button
      class="md:hidden flex flex-col gap-1.5 p-1 cursor-pointer bg-transparent border-none"
      on:click={toggleMenu}
      aria-label="Toggle menu"
    >
      <span class="block w-6 h-0.5 bg-white rounded transition-all duration-300
        {menuOpen ? 'translate-y-2 rotate-45' : ''}"></span>
      <span class="block w-6 h-0.5 bg-white rounded transition-all duration-300
        {menuOpen ? 'opacity-0' : ''}"></span>
      <span class="block w-6 h-0.5 bg-white rounded transition-all duration-300
        {menuOpen ? '-translate-y-2 -rotate-45' : ''}"></span>
    </button>

  </div>

  <!-- Mobile menu -->
  <div class="md:hidden overflow-hidden transition-all duration-300
    {menuOpen ? 'max-h-96 border-b border-border' : 'max-h-0'}">
    <ul class="flex flex-col items-center gap-6 py-8 list-none">
      {#each links as { href, label }}
        <li>
          <a
            {href}
            on:click={closeMenu}
            class="text-sm transition-colors duration-200
              {$page.url.pathname === href ? 'text-white' : 'text-muted hover:text-white'}"
          >
            {label}
          </a>
        </li>
      {/each}
    </ul>
  </div>
</nav>