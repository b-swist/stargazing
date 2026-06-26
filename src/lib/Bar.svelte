<script lang="ts">
  import { onMount } from "svelte";
  import { fly as transition } from "svelte/transition";

  let {
    url = "https://duckduckgo.com/?q=%s",
    placeholder = "search...",
    alwaysHideUnfocused = true,
  } = $props();

  let visible = $state(true);
  let query = $state("");
  let searchbox: HTMLInputElement | undefined = $state();

  type KeyHandler = Record<string, (e?: KeyboardEvent) => void>;

  const inputKeyHandler: KeyHandler = {
    Enter: search,
    Escape: () => {
      if (isSelectionActive()) clearSelection();
      else searchbox!.blur();
    },
  };

  const windowKeyHandler: KeyHandler = {
    "/": () => {
      visible = true;
      setTimeout(() => searchbox!.focus(), transition.length);
    },
  };

  function handleInput(event: KeyboardEvent, keyHandler: KeyHandler): void {
    if (keyHandler[event.key]) {
      event.preventDefault();
      keyHandler[event.key]();
    }
  }

  function search(): void {
    if (!query) return;

    const encoded = encodeURIComponent(query);
    window.location.href = url.replace("%s", encoded);
  }

  const isInputActive = () =>
    document.querySelector("input") === document.activeElement;

  const isSelectionActive = () =>
    searchbox!.selectionStart !== searchbox!.selectionEnd;

  function clearSelection(): void {
    const end = searchbox!.selectionEnd;
    searchbox!.setSelectionRange(end, end);
  }

  onMount(() => searchbox!.focus());
</script>

<svelte:window
  on:keydown={(e) => {
    if (isInputActive()) return;
    handleInput(e, windowKeyHandler);
  }}
/>

<div>
  {#if visible}
    <input
      type="search"
      {placeholder}
      bind:value={query}
      onkeydown={(e) => handleInput(e, inputKeyHandler)}
      bind:this={searchbox}
      onfocus={() => searchbox!.select()}
      onfocusout={() => {
        if (alwaysHideUnfocused || !query) visible = false;
      }}
      transition:transition={{ y: -75, duration: 750 }}
    />
  {/if}
</div>

<style>
  input {
    background-color: MidnightBlue;
    color: Linen;
    border: none;
    font-size: 16pt;
    width: 50vw;
    padding: 0.6em 1em;
    border-radius: 2em;
    &:focus {
      outline: none;
    }
  }
</style>
