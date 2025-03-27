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
    <div class="glow"></div>
    <div class="scoreboard">
        {#if scoreboard.header}
            <div class="header">
                <TextComponent fontSize={14} allowPreformatting={true} textComponent={scoreboard.header}/>
            </div>
        {/if}
        <div class="entries">
            {#each scoreboard.entries as {name, score}}
                <div class="row">
                    <TextComponent fontSize={14} allowPreformatting={true} textComponent={name}/>
                </div>
            {/each}
        </div>
    </div>
{/if}

<style lang="scss">
  @use "../../../colors.scss" as *;

  .glow {
    position: absolute;
    inset: -8px;
    background: linear-gradient(to right, $flamingo, $mauve, $blue);
    border-radius: 1rem;
    filter: blur(12px) opacity(10%);
  }

  .scoreboard {
    width: max-content;
    border-radius: 12px;
    overflow: hidden;
    font-size: 14px;
    min-width: 200px;
  }

  .entries {
    background-color: rgba($base, 0.9);
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
  }
</style>
