<script lang="ts">
    import {listen} from "../../../../integration/ws";
    import type {KeyEvent} from "../../../../integration/events";
    import type {MinecraftKeybind} from "../../../../integration/types";

    export let gridArea: string;
    export let key: MinecraftKeybind | undefined;

    let active = false;

    listen("key", (e: KeyEvent) => {
        if (e.key !== key?.key.translationKey) {
            return;
        }

        active = e.action === 1 || e.action === 2;
    });
</script>

<div class="key" style="grid-area: {gridArea};" class:active>
    {key?.key.localized ?? "???"}
</div>

<style lang="scss">
  @use "../../../../colors.scss" as *;

  .key {
    height: 50px;
    background: rgba($base, .8);
    border: 2px solid rgba($text, 0.1);
    color: $text;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 12px;
    font-size: 16px;
    font-weight: 900;
    text-transform: lowercase;
    transition: ease background .2s, ease color .2s, ease box-shadow .2s;
    position: relative;
    box-shadow: 0px 0px 12px $base;
    text-align: center;

    &.active {
      background: rgba($accent, .8);
      box-shadow: 0px 0px 12px $accent;
      color: $base;
    }
  }
</style>
