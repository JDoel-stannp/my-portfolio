<script lang="ts">
  interface SocialLink {
    href: string;
    label: string;
  }

  const socials: SocialLink[] = [
    { href: 'https://github.com',   label: 'GitHub' },
    { href: 'https://linkedin.com', label: 'LinkedIn' },
    { href: 'https://twitter.com',  label: 'Twitter' },
  ];

  const year: number = new Date().getFullYear();
</script>

<footer class="border-t border-border mt-auto">
  <div class="max-w-5xl mx-auto px-6 py-8 flex flex-col sm:flex-row items-center justify-between gap-4">

    <p class="text-muted text-sm">
      © {year} James Doel. Crafted with SvelteKit.
    </p>

    <div class="flex items-center gap-6">
      {#each socials as { href, label }}
        <a
          {href}
          target="_blank"
          rel="noopener noreferrer"
          class="text-muted text-sm hover:text-accent transition-colors duration-200"
        >
          {label}
        </a>
      {/each}
    </div>

  </div>
</footer>