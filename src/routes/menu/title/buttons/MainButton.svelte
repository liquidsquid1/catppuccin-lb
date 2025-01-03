<script lang="ts">
  import {fade, fly} from "svelte/transition";
  import {createEventDispatcher} from "svelte";

  export let title: string;
  export let icon: string;
  export let index: number;

  let hovered = false;

  const dispatch = createEventDispatcher();
</script>

<!-- svelte-ignore a11y-no-static-element-interactions -->
<!-- svelte-ignore a11y-click-events-have-key-events -->
<div class="main-button" on:mouseenter={() => hovered = true} on:mouseleave={() => hovered = false} on:click={() => hovered = false}
   on:click={() => dispatch("click")}>
  <div class="icon">
    <img src="img/menu/icon-{icon}.svg" alt={icon}>
  </div>

  <div class="title">{title}</div>

  <div class="wrapped-content">
      <slot parentHovered={hovered}/>
  </div>
</div>

<style lang="scss">
@use "../../../../colors.scss" as *;

.main-button {
  background-color: rgba($base, 0.9);
  width: 590px;
  padding: 25px 35px;
  display: grid;
  grid-template-columns: max-content 1fr max-content;
  align-items: center;
  cursor: pointer;
  border-radius: 24px;
  column-gap: 25px;

  background: linear-gradient(to left, rgba($base, .68) 50%, $accent 50%);
  background-size: 200% 10%;
  background-position: right bottom;
  will-change: background-position;
  transition: background-position 2s linear, transform .1s ease;

  &:hover {
    background-position: left bottom;
    transform: scale(1.1) translateY(-0.25em) rotateZ(-2deg);
  }
}

.icon {
  background-color: $mantle;
  border: 2px solid $accent;
  width: 90px;
  height: 90px;
  border-radius: 32px;
  transition: ease background-color 0.2s;
  position: relative;

  img {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
  }
}

.title {
  font-size: 26px;
  color: $accent;
  filter: drop-shadow(4px 4px 16px $mantle);
  font-weight: 600;
}
</style>