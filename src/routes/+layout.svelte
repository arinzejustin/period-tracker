<script lang="ts">
  import { onMount } from "svelte";
  import { browser } from "$app/environment";

  import Header from "$lib/components/Header.svelte";
  import "../app.css";

  onMount(() => {
    if (browser && "serviceWorker" in navigator) {
      navigator.serviceWorker
        .register("/service-worker.js")
        .then((registration) => {
          console.log("Service Worker registered:", registration);
        })
        .catch((error) => {
          console.log("Service Worker registration failed:", error);
        });
    }
  });
</script>

<div class="relative">
  <div class="fixed top-0 left-0 right-0 z-50">
    <Header />
  </div>
  <slot />
</div>
