<script lang="ts">
  import { createEventDispatcher } from "svelte";

  export let value: boolean;
  export let title: string;

  const dispatch = createEventDispatcher();
</script>

<div class="switch-setting">
  <label class="switch">
    <input type="checkbox" bind:checked={value} on:change={() => dispatch("change")} />
    <span class="checkbox"></span>
  </label>
  <div class="title">{title}</div>
</div>

<style lang="scss">
  @use "sass:color";
  @use "../../../../colors.scss" as *;

  .switch-setting {
    display: flex;
    align-items: center;
    justify-content: flex-start;
    margin: 10px 0;
    transition: all 0.3s ease;
  }

  .title {
    color: $text;
    font-size: 22px;
    margin-left: 12px;
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
    width: 32px;
    height: 32px;
    background-color: rgba($crust, 0.8);
    border-radius: 10px;
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
