<script lang="ts">
  import { onMount } from "svelte";

  let slides: string[] = [];
  let index = 0;

  function next() {
    if (index < slides.length - 1) index++;
  }

  function prev() {
    if (index > 0) index--;
  }

  onMount(async () => {
    const res = await fetch("/presentation.html");
    const html = await res.text();

    const doc = new DOMParser().parseFromString(html, "text/html");

    // Inject <style> blocks
    doc.querySelectorAll("style").forEach((s) => {
      const el = document.createElement("style");
      el.textContent = s.textContent;
      document.head.appendChild(el);
    });

    // Inject <link> tags (fonts, font-awesome, preconnects)
    doc.querySelectorAll("link").forEach((l) => {
      const clone = document.createElement("link");
      [...l.attributes].forEach((a) => clone.setAttribute(a.name, a.value));
      document.head.appendChild(clone);
    });

    // Extract slides
    slides = [...doc.querySelectorAll(".slide-container")].map((n) =>
      n.outerHTML
    );

    window.addEventListener("keydown", (e) => {
      if (e.key === "ArrowRight") next();
      if (e.key === "ArrowLeft") prev();
    });
  });
</script>

<div class="viewer" on:click={next}>
  {#if slides.length}
    {@html slides[index]}
  {/if}
</div>

<div class="controls">
  <button on:click|stopPropagation={prev} disabled={index === 0}>←</button>
  <button on:click|stopPropagation={next} disabled={index === slides.length - 1}>→</button>
</div>

<style>
  /* Full window */
  .viewer {
    width: 100vw;
    height: 100vh;
    overflow-y: auto; /* new: allow slide to scroll if needed */
    overflow-x: hidden;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  :global(html),
  :global(body) {
    margin: 0;
    padding: 0;
    width: 100%;
    height: 100%;
    overflow: hidden;
  }

  /* Let slides render EXACTLY as defined */
  :global(.slide-container) {
    margin-bottom: 3vw; /* matches spacing from original body flex */
  }

  .controls {
    position: fixed;
    bottom: 2rem;
    right: 2rem;
    display: flex;
    gap: 1rem;
    z-index: 9999;
  }
</style>
