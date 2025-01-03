<script lang="ts">
    import {createEventDispatcher} from "svelte";
    import CircleLoader from "../CircleLoader.svelte";

    export let title: string;
    export let disabled = false;
    export let secondary = false;
    export let inset = false;
    export let listenForEnter = false;
    export let loading = false;

    const dispatch = createEventDispatcher();

    function handleKeyDown(e: KeyboardEvent) {
        if (!listenForEnter) {
            return;
        }
        if (e.key === "Enter") {
            dispatch("click");
        }
    }
</script>

<svelte:window on:keydown={handleKeyDown}/>
<button class="button-setting" class:inset type="button" on:click={() => dispatch("click")} {disabled} class:secondary>
  {#if loading}
      <CircleLoader/>
  {/if}
  {title}
</button>

<style lang="scss">
  @use "sass:color";
  @use "../../../../colors.scss" as *;

  .button-setting {
    position: relative;
    color: $text;
    font-family: "Inter", sans-serif;
    font-weight: 900;
    padding: 20px;
    border: none;
    border-radius: 12px;
    font-size: 20px;
    transition: ease background-color .2s, ease opacity .2s, border-bottom .2s ease;
    background: rgba($mantle, .6);
    box-shadow: 0px 0px 16px rgba($crust, .5);

    &.inset {
      margin: 0 30px;
    }

    &:not([disabled]):hover {
      background-color: color.adjust(color.adjust($accent, $saturation: -30%), $lightness: -10%);
      cursor: pointer;

      &.secondary {
        background-color: color.adjust(color.adjust($base, $saturation: -30%), $lightness: -10%);
      }
    }

    &[disabled] {
      opacity: .6;
    }
  }
</style>