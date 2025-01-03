<script lang="ts">
  import {fade} from "svelte/transition";
  import {createEventDispatcher} from "svelte";
  import ToolTip from "../../common/ToolTip.svelte";

  export let title: string;
  export let icon: string;
  export let parentHovered: boolean;

  const dispatch = createEventDispatcher();
</script>

<!-- svelte-ignore a11y-no-static-element-interactions -->
<!-- svelte-ignore a11y-click-events-have-key-events -->
<div class="child-button" on:click|stopPropagation={() => dispatch("click")} class:parent-hovered={parentHovered}>
  <ToolTip color="black" text="Join Realms server" />

  <div class="icon">
    <img transition:fade="{{ duration: 200 }}" src="img/menu/icon-{icon}.svg" alt={title}>
  </div>

  <div class="title">{title}</div>
</div>

<style lang="scss">
  @use "../../../../colors.scss" as *;

  .child-button {
    position: relative;
    display: flex;
    align-items: center;
    border-radius: 5px;
    background-color: $base;
    transition: ease background-color .2s;
    padding: 15px;

    &.parent-hovered {
      box-shadow: 0px 0px 4px rgba($base, 0.5);

      .title {
        color: $text;
      }
    }
  }

  .title {
    color: $accent;
    font-weight: 600;
    font-size: 16px;
    transition: ease color 0.2s;
    margin-left: 10px;
  }

  .icon { /* necessary because svelte's transition system sucks */
    width: 28px;
    height: 28px;
    position: relative;

    img {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
    }
  }
</style>