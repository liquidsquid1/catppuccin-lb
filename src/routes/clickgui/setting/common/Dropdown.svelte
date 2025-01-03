<script lang="ts">
    import {createEventDispatcher} from "svelte";
    import {convertToSpacedString, spaceSeperatedNames} from "../../../../theme/theme_config";

    export let name: string | null;
    export let options: string[];
    export let value: string;

    const dispatch = createEventDispatcher();

    let expanded = false;
    let dropdownHead: HTMLElement;

    function windowClickHide(e: MouseEvent) {
        if (!dropdownHead.contains(e.target as Node)) {
            expanded = false;
        }
    }

    function updateValue(v: string) {
        value = v;
        dispatch("change");
    }
</script>

<svelte:window on:click={windowClickHide}/>
<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->
<div class="dropdown" class:expanded on:click={() => (expanded = !expanded)}>
    <div class="head" bind:this={dropdownHead}>
        {#if name !== null}
            <span class="text">{name} &bull; {$spaceSeperatedNames ? convertToSpacedString(value) : value}</span>
        {:else}
            <span class="text">{value}</span>
        {/if}
    </div>

    {#if expanded}
        <div class="options">
            {#each options as o}
                <div
                        class="option"
                        class:active={o === value}
                        on:click={() => updateValue(o)}
                >
                    {$spaceSeperatedNames ? convertToSpacedString(o) : o}
                </div>
            {/each}
        </div>
    {/if}
</div>

<style lang="scss">
  @use "../../../../colors.scss" as *;

  .dropdown {
    position: relative;

    &.expanded {
      .text::after {
        transform: translateY(-50%) rotate(0);
        opacity: 1;
      }

      .head {
        border-radius: 3px 3px 0 0;
      }
    }
  }

  .head {
    background-color: rgba($base, 0.8);
    border: 1px solid rgba(white, 0.1);
    box-shadow: 0px 0px 8px rgba($crust, 0.5);
    padding: 6px 10px;
    cursor: pointer;
    display: flex;
    align-items: center;
    position: relative;
    border-radius: 6px;
    transition: ease border-radius .2s;

    .text {
      font-weight: 500;
      color: $text;
      font-size: 12px;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      margin-right: 20px;
    }

    .text::after {
      content: "";
      display: block;
      position: absolute;
      height: 10px;
      width: 10px;
      right: 10px;
      top: 50%;
      background-image: url("/img/clickgui/icon-settings-expand.svg");
      background-position: center;
      background-repeat: no-repeat;
      transform-origin: 50% 50%;
      transform: translateY(-50%) rotate(-90deg);
      transition: ease opacity 0.2s,
      ease transform 0.4s;
    }
  }

  .options {
    padding: 6px 10px;
    background-color: rgba($crust, 0.8);
    box-shadow: 0px 0px 8px rgba($crust, 0.5);
    border-top: none;
    border-radius: 0 0 3px 3px;
    z-index: 9999;
    width: 100%;
    position: absolute;
    border-bottom-left-radius: 6px;
    border-bottom-right-radius: 6px;

    .option {
      color: $overlay0;
      font-weight: 500;
      font-size: 12px;
      padding: 2px 0;
      cursor: pointer;
      text-align: center;
      transition: ease color 0.2s;

      &:hover {
        color: $text;
      }

      &.active {
        color: $accent;
      }
    }
  }
</style>
