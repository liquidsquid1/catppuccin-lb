<script lang="ts">
    import type {Browser} from "../../integration/types.js";
    import {onMount} from "svelte";
    import {
        browserForceReload,
        browserGoBack,
        browserGoForward,
        browserNavigate,
        browserReload,
        getBrowser
    } from "../../integration/rest.js";
    import {listen} from "../../integration/ws";
    import type {BrowserUrlChangeEvent} from "../../integration/events";

    let browser: Browser;

    async function loadBrowser() {
        browser = await getBrowser();
    }

    onMount(async () => {
        await new Promise(resolve => setTimeout(resolve, 250));
        await loadBrowser();
    });

    async function onKeyPress(event: KeyboardEvent) {
        if (event.key === "Enter") {
            await browserNavigate(browser.url);
        }
    }

    async function handleGo() {
        await browserNavigate(browser.url);
    }

    async function handleBack() {
        await browserGoBack();
    }

    async function handleForward() {
        await browserGoForward();
    }

    async function handleReload() {
        await browserReload();
    }

    async function handleForceReload() {
        await browserForceReload();
    }

    listen("browserUrlChange", (e: BrowserUrlChangeEvent) => {
        browser.url = e.url;
    });
</script>

<style lang="scss">

    @use "../../colors.scss" as *;

    .browser-controls {
        display: flex;
        justify-content: space-between;
        align-items: center;
        position: fixed;
        bottom: 10px;
        left: 10px;
        right: 10px;
        background-color: $base;
        padding: 10px;
        border-radius: 10px;
        box-shadow: 0 4px 8px rgba($crust, 0.4);
        border: 2px solid $accent;
    }

    .address-bar {
        flex-grow: 1;
        margin: 0 10px;
        background: $surface0;
    }

    input {
        width: 100%;
        padding: 10px;
        border-radius: 5px;
        outline: none;
        font-size: 14px;
    }

    button {
        background-color: $accent;
        color: $text;
        border: none;
        padding: 10px 15px;
        margin-left: 5px;
        border-radius: 5px;
        cursor: pointer;
        font-size: 14px;
    }

    button:disabled {
        background-color: $overlay0;
        cursor: not-allowed;
    }

    button:focus {
        outline: none;
    }
</style>

{#if browser}
    <div class="browser-controls">
        <button on:click={handleBack}>&larr;</button>
        <button on:click={handleForward}>&rarr;</button>
        <button on:click={handleReload}>&#x21bb;</button>
        <div class="address-bar">
            <input id="url" bind:value={browser.url} on:keypress={onKeyPress} placeholder="Enter URL" />
        </div>
        <button on:click={handleGo}>Go</button>
        <button on:click={handleForceReload}>Force Reload</button>
    </div>
{/if}
