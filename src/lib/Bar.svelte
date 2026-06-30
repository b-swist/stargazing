<script lang="ts">
  import { onMount } from "svelte";
  import { fly as transition } from "svelte/transition";

  let {
    url = "https://duckduckgo.com/?q=%s",
    placeholder = "search...",
    alwaysHideUnfocused = false,
    alwaysShow = false,
  } = $props();

  let visible = $state(true);
  let query = $state("");
  let searchbar: HTMLInputElement | undefined = $state();

  type KeyHandler = Record<string, (e?: KeyboardEvent) => void>;

  const inputKeyHandler: KeyHandler = {
    Enter: search,
    Escape: () => {
      if (isSelectionActive()) clearSelection();

      if (alwaysShow) return;
      else searchbar!.blur();
    },
  };

  const windowKeyHandler: KeyHandler = {
    "/": () => {
      visible = true;
      setTimeout(() => searchbar!.focus(), transition.length);
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
    searchbar!.selectionStart !== searchbar!.selectionEnd;

  function clearSelection(): void {
    const end = searchbar!.selectionEnd;
    searchbar!.setSelectionRange(end, end);
  }

  onMount(() => searchbar!.focus());
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
      bind:this={searchbar}
      onfocus={() => searchbar!.select()}
      onfocusout={() => {
        if (alwaysShow) return;

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
