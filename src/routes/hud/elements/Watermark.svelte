<script lang="ts">
    import { getClientInfo, getSession } from "../../../integration/rest";
    import type { ClientInfo, Session } from "../../../integration/types";
    import { onMount } from "svelte";
    import { listen } from "../../../integration/ws";
    import {getModules} from "../../../integration/rest";

    async function canShowUsername() {
        const modules = await getModules();
        return modules.some(module => module.name === "NameProtect" && !module.enabled)
    }

    let clientInfo: ClientInfo | null = null;
    let session: Session | null = null;
    let time: string;

    let showUsername = false;

    async function updateClientInfo() {
        clientInfo = await getClientInfo();
    }

    async function updateSession() {
        session = await getSession();
    }

    function updateTime() {
        const now = new Date();
        const hours = now.getHours();
        const minutes = now.getMinutes();
        time = `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}`;
    }

    onMount(async () => {
        updateClientInfo();
        updateTime();
        updateSession();
        showUsername = await canShowUsername();
        setInterval(async () => {
            updateClientInfo();
            updateTime();
            updateSession();
            showUsername = await canShowUsername();
        }, 1000);
    });

    listen("session", async () => {
        await updateSession();
    });
</script>

<div class="watermark">
    <div class="watermark-content">
        <div class="logo">
            <img src="img/clickgui/icon-client.svg" alt="icon"/>
        </div>
        {#if session && showUsername}
            <div class="separator"></div>
            <div class="info">
                {session.username}
            </div>
        {/if}
        <div class="separator"></div>
        <div class="info">
            {time}
        </div>
        <div class="separator"></div>
        {#if clientInfo}
            <div class="info">
                {clientInfo.fps}fps
            </div>
        {/if}
    </div>
</div>

<style lang="scss">
  @import "../../../colors.scss";

  @keyframes pulsing {
    0% {
        filter: opacity(100%) drop-shadow(0px 0px 8px white);
    }
    50% {
        filter: opacity(75%) drop-shadow(0px 0px 2px rgba(white, 0.5));
    }
    100% {
        filter: opacity(100%) drop-shadow(0px 0px 8px white);
    }
  }

  .watermark {
    padding: 10px;
    background: rgba($base, 0.8);
    border-radius: 24px;
    color: rgba($text, 0.7);
    font-size: 14px;
    display: flex;
    align-items: center;
    min-width: 150px;
    box-shadow: 0px 0px 16px $base;
  }

  .watermark-content {
    display: flex;
    align-items: center;
    padding: 0 8px;
  }

  .logo {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 28px;
    height: 28px;
    padding: 4px;
    border-radius: 50%;
    // background: $accent;

    img {
        animation: 2.5s ease pulsing infinite;
    }
  }

  .separator {
    width: 2px;
    height: 16px;
    background: rgba($text, 0.25);
    margin: 0 15px;
  }

  .info {
    margin: 0 6px;
  }

  img {
    width: 16px;
    height: 16px;
  }
</style>