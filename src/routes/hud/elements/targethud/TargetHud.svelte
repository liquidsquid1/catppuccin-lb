<script lang="ts">
    import { listen } from "../../../../integration/ws.js";
    import type { PlayerData } from "../../../../integration/types";
    import { REST_BASE } from "../../../../integration/host";
    import { scale } from "svelte/transition";
    import HealthProgress from "./HealthProgress.svelte";
    import type { TargetChangeEvent } from "../../../../integration/events";
    import { Howl } from 'howler';
    import { cubicOut } from "svelte/easing";

    let target: PlayerData | null = null;
    let visible = true;
    let engagePlayed = false;
    let hideTimeout: number;

    const engageSound = new Howl({
        src: ['audio/targethud/engage.wav'],
        preload: true
    });

    const disengageSound = new Howl({
        src: ['audio/targethud/disengage.wav'],
        preload: true
    });

    function startHideTimeout() {
        hideTimeout = setTimeout(() => {
            if (visible) {
                visible = false;
                disengageSound.play();
            }
        }, 500);
    }

    listen("targetChange", (data: TargetChangeEvent) => {
        target = data.target;
        visible = true;

        if (visible && !engagePlayed) {
            engageSound.play();
            engagePlayed = true;
        }

        clearTimeout(hideTimeout);
        startHideTimeout();
    });

    $: if (!visible) {
        engagePlayed = false;
    }
</script>

{#if visible && target != null}
    <div class="targethud" transition:scale={{ duration: 500, easing: cubicOut }}>
        <div class="main-wrapper">
            <div class="avatar">
                <img src="{REST_BASE}/api/v1/client/resource/skin?uuid={target.uuid}" alt="avatar" />
            </div>
            <div class="name">{target.username}</div>
            <div class="health-stats">
                <div class="stat">
                    <div class="value">{Math.floor(target.actualHealth + target.absorption)}</div>
                    <img class="icon" src="img/hud/targethud/icon-health.svg" alt="health" />
                </div>
            </div>
        </div>
        <HealthProgress maxHealth={target.maxHealth + target.absorption} health={target.actualHealth + target.absorption} />
    </div>
{/if}

<style lang="scss">
    @use "../../../../colors.scss" as *;

    .targethud {
        background: rgba($base, 0.9);
        box-shadow: 0px 0px 16px $base;
        border-radius: 12px;
        overflow: hidden;
    }

    .main-wrapper {
        display: grid;
        grid-template-areas:
            "a b d"
            "a c d";
        column-gap: 10px;
        padding: 10px 12px;
    }

    .name {
        grid-area: b;
        color: $text;
        font-weight: 900;
        text-transform: lowercase;
        align-self: flex-end;
    }

    .health-stats {
        grid-area: c;
        display: flex;
        column-gap: 10px;

        .stat {
            .value {
                color: $subtext0;
                font-size: 14px;
                min-width: 18px;
                display: inline-block;
            }
        }
    }

    .avatar {
        grid-area: a;
        height: 50px;
        width: 50px;
        position: relative;
        image-rendering: pixelated;
        background-image: url("/img/steve.png");
        background-repeat: no-repeat;
        background-size: cover;
        border-radius: 5px;
        overflow: hidden;

        img {
            position: absolute;
            scale: 6.25;
            left: 118px;
            top: 118px;
        }
    }
</style>
