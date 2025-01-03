<script lang="ts">
    import {listen} from "../../../../integration/ws";
    import type {PointerInfoEvent} from "../../../../integration/events";
    import type {Pointer} from "../../../../integration/types";
    import PointerView from "./PointerView.svelte";
    let pointers: Pointer[] | undefined;
    listen("pointerInfo", (e: PointerInfoEvent) => {
        pointers = e.pointers;
    });
</script>

<div class="container">
    {#if pointers}
        {#each pointers as pointer}
            <PointerView {...pointer}/>
        {/each}
    {/if}
</div>

<style lang="scss">

  @use "../../../../colors.scss" as *;

  .container {
    position: relative;
    filter: drop-shadow(0 0 8px rgba($accent, 0.8))
          drop-shadow(0 0 16px rgba($accent, 0.5))
          drop-shadow(0 0 24px rgba($blue, 0.8));
  }
</style>