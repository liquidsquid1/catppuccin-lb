<script lang="ts">
    import { listen } from "../../../integration/ws";
    import { scale } from "svelte/transition";
    import type { BlockCountChangeEvent, ClientPlayerDataEvent } from "../../../integration/events";
    import type { PlayerData } from "../../../integration/types";
    import { expoInOut } from "svelte/easing";

    let count: number | undefined;
    let playerData: PlayerData | null = null;

    let lastX = 0;
    let lastZ = 0;

    listen("blockCountChange", (data: BlockCountChangeEvent) => {
        count = data.count;
    });

    listen("clientPlayerData", (event: ClientPlayerDataEvent) => {
      if (playerData) {
        lastX = playerData.position.x;
        lastZ = playerData.position.z;
      }
      playerData = event.playerData;
    });

    function getProgressPercentage(value: number): number {
        return (value % 64) / 64 * 100;
    };

    function roundToDecimal(value: number, decimal: number) {
      return Math.round(value * Math.pow(10, decimal)) / Math.pow(10, decimal)
    }

    function getBPS(
      lastX: number,
      currentX: number,
      lastZ: number,
      currentZ: number,
      tickrate: number
    ): number {
      const deltaX = currentX - lastX;
      const deltaZ = currentZ - lastZ;

      const distanceMoved = Math.sqrt(deltaX * deltaX + deltaZ * deltaZ);
      const blocksPerSecond = distanceMoved * tickrate;

      return blocksPerSecond;
    }
</script>

{#if count !== undefined}
    <div class="notification" in:scale={{ duration: 500, easing: expoInOut }} out:scale={{ duration: 500, easing: expoInOut }}>
        <div class="header">
            <span class="accent">scaffold</span>
        </div>
        <div class="info">
            {count} blocks left
            {#if playerData}
              | {roundToDecimal(getBPS(lastX, playerData.position.x, lastZ, playerData.position.z, 20), 2)}m/s
            {/if}
        </div>
        <div class="progress-bar">
            <div class="progress" style="width: {getProgressPercentage(count)}%"></div>
        </div>
    </div>
{/if}

<style lang="scss">
  @use "../../../colors.scss" as *;

  .notification {
    background-color: rgba($base, 0.8);
    border-radius: 16px;
    padding: 12px;
    min-width: 200px;
    color: rgba($text, 0.7);
    box-shadow: 0px 0px 16px $base;
  }

  .header {
    display: flex;
    align-items: center;
    font-size: 16px;
    font-weight: 900;
    text-shadow: 0px 0px 10px $accent;
  }

  .accent {
    color: $accent;
  }

  .info {
    font-size: 14px;
    margin-top: 5px;
  }

  .progress-bar {
    margin-top: 10px;
    background-color: rgba(255, 255, 255, 0.1);
    border-radius: 5px;
    height: 10px;
    overflow: hidden;
  }

  .progress {
    background-color: $accent;
    height: 100%;
    transition: ease width 0.1s;
  }
</style>
