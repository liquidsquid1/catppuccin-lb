<script lang="ts">
  import { createEventDispatcher } from "svelte";

  export let value: boolean;
  export let name: string;

  const dispatch = createEventDispatcher();
</script>

<div class="switch-container">
  <label class="switch">
    <input type="checkbox" bind:checked={value} on:change={() => dispatch("change")} />
    <span class="checkbox"></span>
  </label>
  <div class="title">{name}</div>
</div>

<style lang="scss">
  @use "sass:color";
  @use "../../../../colors.scss" as *;

  .switch-container {
    display: flex;
    align-items: center;
    justify-content: flex-start;
    margin: none;
    transition: all 0.3s ease;
  }

  .title {
    color: $text;
    font-size: 11px;
    margin-left: 6px;
    font-weight: 500;
    transition: color 0.2s ease-in-out;
  }

  .switch {
    position: relative;
    display: flex;
    align-items: center;
    cursor: pointer;
  }

  .checkbox {
    display: block;
    position: relative;
    width: 16px;
    height: 16px;
    background-color: rgba($crust, 0.8);
    border-radius: 4px;
    border: 2px solid rgba(white, 0.1);
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
    transition: background-color 0.3s ease, box-shadow 0.3s ease, border 0.3s ease;
    will-change: background-color, box-shadow;
  }

  input {
    display: none;
  }

  input:checked + .checkbox {
    background-color: $accent;
    box-shadow: 0 0 8px 2px $accent;
  }

  .switch:hover .checkbox {
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
  }

  .switch:focus-within .checkbox {
    outline: 2px solid $accent;
  }
</style>
