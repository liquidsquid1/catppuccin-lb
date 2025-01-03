<script lang="ts">
    export let closeOnInternalClick: boolean;

    let expanded = false;
    let selectElement: HTMLElement;
    let headerElement: HTMLElement;

    function handleWindowClick(e: MouseEvent) {
        if (!selectElement.contains(e.target as Node)) {
            expanded = false;
        }
    }

    function handleSelectClick(e:MouseEvent) {
        if (closeOnInternalClick) {
            expanded = !expanded;
        } else {
            if (!expanded) {
                expanded = true;
            } else {
                expanded = !headerElement.contains(e.target as Node);
            }
        }
    }
</script>

<svelte:window on:click={handleWindowClick}/>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->
<div class="select" class:expanded bind:this={selectElement} on:click={handleSelectClick}>
    <div class="header" bind:this={headerElement}>
        <span class="title">
            <slot name="title"/>
        </span>
        <img src="img/menu/icon-select-arrow.svg" alt="expand">
    </div>
    {#if expanded}
        <div class="options">
            <slot name="options"></slot>
        </div>
    {/if}
</div>

<style lang="scss">
  @use "../../../../../colors.scss" as *;

  .select {
    cursor: pointer;
    min-width: 250px;
    position: relative;

    &.expanded {
      .header {
        border-radius: 12px 12px 0 0;
      }
    }
  }

  .header {
    background: rgba($mantle, 0.5);
    box-shadow: 0px 0px 8px $mantle;
    border-bottom: 1px solid rgba(white, 0.1);
    padding: 20px;
    display: flex;
    column-gap: 20px;
    align-items: center;
    justify-content: space-between;
    border-radius: 12px;
    transition: ease border-radius .2s;

    .title {
      color: $text;
      font-size: 20px;
      font-weight: 900;

    }
  }

  .options {
    position: absolute;
    z-index: 1000;
    width: 100%;
    border-radius: 0 0 12px 12px;
    max-height: 250px;
    overflow: auto;
    background-color: rgba($mantle, 0.9);
    border-left: $accent;
  }
</style>