<script lang="ts">
  import { onMount } from "svelte";

  interface ContactLink {
    emoji: string;
    label: string;
    value: string;
    href: string;
  }

  const FORMSPREE_URL = "https://formspree.io/f/myknpapl";

  const contactLinks: ContactLink[] = [
    {
      emoji: "📧",
      label: "Email",
      value: "jd.stannp@gmail.com",
      href: "mailto:hello@jamesdoel.com",
    },
    {
      emoji: "🐙",
      label: "GitHub",
      value: "github.com/JDoel-stannp",
      href: "https://github.com/JDoel-stannp",
    },
  ];

  let name: string = "";
  let email: string = "";
  let message: string = "";
  let sent: boolean = false;
  let sending: boolean = false;
  let error: string = "";
  let visible: boolean = false;

  onMount(() => setTimeout(() => (visible = true), 100));

  async function handleSubmit(): Promise<void> {
    sending = true;
    error = "";
    try {
      const res = await fetch(FORMSPREE_URL, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Accept: "application/json",
        },
        body: JSON.stringify({ name, email, message }),
      });
      if (res.ok) {
        sent = true;
        name = "";
        email = "";
        message = "";
        setTimeout(() => (sent = false), 4000);
      } else {
        error = "Something went wrong. Please try again.";
      }
    } catch {
      error = "Network error. Please check your connection and try again.";
    } finally {
      sending = false;
    }
  }
</script>

<svelte:head>
  <title>Contact — James Doel</title>
</svelte:head>

<section
  class="max-w-5xl mx-auto px-6 pt-32 pb-24
  transition-all duration-700 {visible
    ? 'opacity-100 translate-y-0'
    : 'opacity-0 translate-y-6'}"
>
  <div style="display:flex; gap:4rem; align-items:flex-start;">
    <!-- Left -->
    <div style="flex:1.2; min-width:320px;">
      <p class="text-xs tracking-[0.2em] uppercase text-accent mb-3">
        Say Hello
      </p>
      <h1 class="font-serif text-[clamp(2.5rem,6vw,4rem)] mb-4">
        Get In Touch
      </h1>
      <div
        class="w-12 h-0.5 bg-gradient-to-r from-accent to-accent2 rounded mb-10"
      ></div>

      <p class="text-[#aaaaaa] text-[1.05rem] leading-relaxed mb-10">
        Whether you have a project in mind, want to collaborate, or just want to
        say hi — my inbox is always open. I'll get back to you as soon as I can.
      </p>

      <div class="flex flex-col gap-6">
        {#each contactLinks as { emoji, label, value, href }}
          <a
            {href}
            target="_blank"
            rel="noopener noreferrer"
            class="flex items-center gap-4 group"
          >
            <div
              class="w-11 h-11 rounded-xl flex items-center justify-center text-xl flex-shrink-0
              border border-border bg-card transition-all duration-200
              group-hover:border-accent group-hover:shadow-[0_0_16px_rgba(167,139,250,0.3)]"
            >
              {emoji}
            </div>
            <div>
              <p class="text-xs text-muted uppercase tracking-widest mb-0.5">
                {label}
              </p>
              <p
                class="text-sm text-white group-hover:text-accent transition-colors duration-200"
              >
                {value}
              </p>
            </div>
          </a>
        {/each}
      </div>
    </div>

    <!-- Right: form -->
    <div style="flex:1.4;" class="rounded-2xl p-8 bg-card border border-border">
      {#if sent}
        <div
          class="flex flex-col items-center justify-center gap-4 py-12 text-center"
        >
          <div
            class="w-16 h-16 rounded-full flex items-center justify-center text-3xl"
            style="background:rgba(34,197,94,0.1); border:1px solid rgba(34,197,94,0.3);"
          >
            ✓
          </div>
          <h3 class="font-serif text-2xl">Message Sent!</h3>
          <p class="text-muted text-sm">
            Thanks for reaching out. I'll get back to you soon.
          </p>
        </div>
      {:else}
        <form
          on:submit|preventDefault={handleSubmit}
          class="flex flex-col gap-6"
        >
          <div style="display:grid; grid-template-columns:1fr 1fr; gap:1rem;">
            <div class="flex flex-col gap-2">
              <label
                for="name"
                class="text-xs uppercase tracking-widest text-muted"
              >
                Name
              </label>
              <input
                id="name"
                type="text"
                bind:value={name}
                placeholder="Your name"
                required
                class="px-4 py-3 rounded-lg text-sm text-white placeholder:text-[#444]
                  bg-bg border border-border outline-none transition-all duration-200
                  focus:border-accent focus:shadow-[0_0_0_3px_rgba(167,139,250,0.15)]"
              />
            </div>
            <div class="flex flex-col gap-2">
              <label
                for="email"
                class="text-xs uppercase tracking-widest text-muted"
              >
                Email
              </label>
              <input
                id="email"
                type="email"
                bind:value={email}
                placeholder="you@example.com"
                required
                class="px-4 py-3 rounded-lg text-sm text-white placeholder:text-[#444]
                  bg-bg border border-border outline-none transition-all duration-200
                  focus:border-accent focus:shadow-[0_0_0_3px_rgba(167,139,250,0.15)]"
              />
            </div>
          </div>

          <div class="flex flex-col gap-2">
            <label
              for="message"
              class="text-xs uppercase tracking-widest text-muted"
            >
              Message
            </label>
            <textarea
              id="message"
              bind:value={message}
              rows={6}
              placeholder="What's on your mind?"
              required
              class="px-4 py-3 rounded-lg text-sm text-white placeholder:text-[#444]
                bg-bg border border-border outline-none resize-none transition-all duration-200
                focus:border-accent focus:shadow-[0_0_0_3px_rgba(167,139,250,0.15)]"
            ></textarea>
          </div>

          {#if error}
            <p class="text-sm text-red-400">{error}</p>
          {/if}

          <button
            type="submit"
            disabled={sending}
            class="w-full py-3.5 rounded-lg text-sm font-medium text-white
              transition-all duration-200 hover:opacity-85 hover:-translate-y-px
              disabled:opacity-50 disabled:cursor-not-allowed"
            style="background:linear-gradient(135deg, #a78bfa, #60a5fa);"
          >
            {sending ? "Sending..." : "Send Message"}
          </button>
        </form>
      {/if}
    </div>
  </div>
</section>
