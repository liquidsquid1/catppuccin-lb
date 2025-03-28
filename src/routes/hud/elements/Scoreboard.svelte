<script lang="ts">
    import {listen} from "../../../integration/ws";
    import type {PlayerData, Scoreboard} from "../../../integration/types";
    import TextComponent from "../../menu/common/TextComponent.svelte";
    import type {ClientPlayerDataEvent} from "../../../integration/events";

    let scoreboard: Scoreboard | null = null;

    listen("clientPlayerData", (e: ClientPlayerDataEvent) => {
        const playerData: PlayerData = e.playerData;
        scoreboard = playerData.scoreboard;
    });
</script>

{#if scoreboard}
    <div class="scoreboard-wrapper">
        <div class="glow"></div>
        <div class="scoreboard">
            {#if scoreboard.header}
                <div class="header">
                    <TextComponent fontSize={14} allowPreformatting={true} textComponent={scoreboard.header} />
                </div>
            {/if}
            <div class="entries">
                {#each scoreboard.entries as { name, score }}
                    <div class="row">
                        <TextComponent fontSize={14} allowPreformatting={true} textComponent={name} />
                    </div>
                {/each}
            </div>
        </div>
    </div>
{/if}

<style lang="scss">
  @use "../../../colors.scss" as *;

  .scoreboard-wrapper {
    position: relative;
    display: inline-block;

    .glow {
      position: absolute;
      inset: -8px;
      background: linear-gradient(to right, $flamingo, $mauve, $blue);
      border-radius: 12px;
      filter: blur(16px) opacity(50%);
      opacity: 0.7;
      z-index: -1 !important;
    }
  }

  .scoreboard {
    z-index: 1 !important;
    width: max-content;
    border-radius: 12px;
    overflow: hidden;
    font-size: 14px;
    min-width: 200px;
    border: 1px solid rgba($text, 0.3);
  }

  .entries {
    background: rgba($base, 0.8);
    padding: 10px;
  }

  .row {
    display: flex;
    column-gap: 15px;
    justify-content: space-between;
  }

  .header {
    text-align: center;
    background-color: $crust;
    padding: 7px 10px;
    font-weight: bold;
  }
</style>
