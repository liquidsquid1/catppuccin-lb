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
                  <TextComponent fontSize={0} allowPreformatting={true} textComponent={score}/>
              </div>
          {/each}
      </div>
  </div>
{/if}

<style lang="scss">
@use "../../../colors.scss" as *;

.scoreboard {
  width: max-content;
  min-width: 200px;
  border-radius: 12px;
  overflow: hidden;
  font-size: 14px;
  border: 2px solid rgba($text, 0.1);
}

.entries {
  background-color: rgba($base, 0.8);
  padding: 10px;
}

.row {
  column-gap: 15px;
}

.header {
  text-align: center;
  background-color: rgba($mantle, 0.8);
  padding: 7px 10px;
  font-size: 14px;
  font-weight: 900;
  color: $accent;
}

</style>