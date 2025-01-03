<script lang="ts">
    import {onMount} from "svelte";
    import type {Module as TModule} from "../../integration/types";
    import {listen} from "../../integration/ws";
    import Module from "./Module.svelte";
    import type {ModuleToggleEvent} from "../../integration/events";
    import {fly} from "svelte/transition";
    import {quintOut} from "svelte/easing";
    import {gridSize, highlightModuleName, maxPanelZIndex, showGrid, snappingEnabled} from "./clickgui_store";
    import {setItem} from "../../integration/persistent_storage";
    import {scaleFactor} from "./clickgui_store";

    export let category: string;
    export let modules: TModule[];
    export let panelIndex: number;

    let panelElement: HTMLElement;
    let modulesElement: HTMLElement;

    let renderedModules: TModule[] = [];

    let moving = false;
    let offsetX = 0;
    let offsetY = 0;
    const panelConfig = loadPanelConfig();

    let ignoreGrid = false;

    interface PanelConfig {
        top: number;
        left: number;
        expanded: boolean;
        scrollTop: number;
        zIndex: number;
    }

    function clamp(number: number, min: number, max: number) {
        return Math.max(min, Math.min(number, max));
    }

    function loadPanelConfig(): PanelConfig {
        const localStorageItem = localStorage.getItem(
            `clickgui.panel.${category}`,
        );

        if (!localStorageItem) {
            return {
                top: panelIndex * 50 + 20,
                left: 20,
                expanded: false,
                scrollTop: 0,
                zIndex: 0
            };
        } else {
            const config: PanelConfig = JSON.parse(localStorageItem);

            // Migration
            if (!config.zIndex) {
                config.zIndex = 0;
            }

            if (config.zIndex > $maxPanelZIndex) {
                $maxPanelZIndex = config.zIndex;
            }

            if (config.expanded) {
                renderedModules = modules;
            }

            return config;
        }
    }

    async function savePanelConfig() {
        await setItem(
            `clickgui.panel.${category}`,
            JSON.stringify(panelConfig),
        );
    }

    function fixPosition() {
        panelConfig.left = clamp(panelConfig.left, 0, document.documentElement.clientWidth * (2 / $scaleFactor) - panelElement.offsetWidth);
        panelConfig.top = clamp(panelConfig.top, 0, document.documentElement.clientHeight * (2 / $scaleFactor) - panelElement.offsetHeight);
    }

    function onMouseDown(e: MouseEvent) {
        moving = true;

        offsetX = e.clientX - panelConfig.left;
        offsetY = e.clientY - panelConfig.top;
        panelConfig.zIndex = ++$maxPanelZIndex;
        $showGrid = $snappingEnabled;
    }

    function onMouseMove(e: MouseEvent) {
        if (moving) {
            const newLeft = (e.clientX - offsetX) * (2 / $scaleFactor);
            const newTop = (e.clientY - offsetY) * (2 / $scaleFactor);

            panelConfig.left = snapToGrid(newLeft);
            panelConfig.top = snapToGrid(newTop);

            fixPosition();
            savePanelConfig();
        }
    }

    function onMouseUp() {
        moving = false;
        $showGrid = false;
    }

    function toggleExpanded() {
        if (panelConfig.expanded) {
            renderedModules = [];
        } else {
            renderedModules = modules;
        }

        panelConfig.expanded = !panelConfig.expanded;

        setTimeout(() => {
            fixPosition();
            savePanelConfig();
        }, 500);
    }

    function handleKeydown(e: KeyboardEvent) {
        if (e.key === "Shift") {
            ignoreGrid = true;
        }
    }
    function handleKeyup(e: KeyboardEvent) {
        if (e.key === "Shift") {
            ignoreGrid = false;
        }
    }
    function snapToGrid(value: number): number {
        if (ignoreGrid || !$snappingEnabled) return value;
        return Math.round(value / $gridSize) * $gridSize;
    }

    function handleModulesScroll() {
        panelConfig.scrollTop = modulesElement.scrollTop;
        savePanelConfig();
    }

    highlightModuleName.subscribe((m) => {
        const highlightModule = modules.find(
            (m) => m.name === m.name,
        );
        if (highlightModule) {
            panelConfig.zIndex = ++$maxPanelZIndex;
            panelConfig.expanded = true;
            renderedModules = modules;
            savePanelConfig();
        }
    });

    listen("moduleToggle", (e: ModuleToggleEvent) => {
        const moduleName = e.moduleName;
        const moduleEnabled = e.enabled;

        const mod = modules.find((m) => m.name === moduleName);
        if (!mod) return;

        mod.enabled = moduleEnabled;
        modules = modules;
        if (panelConfig.expanded) {
            renderedModules = modules;
        }
    });

    onMount(() => {
        setTimeout(() => {
            if (!modulesElement) {
                return;
            }

            modulesElement.scrollTo({
                top: panelConfig.scrollTop,
                behavior: "smooth"
            })
        }, 500);
    });
</script>

<svelte:window on:mouseup={onMouseUp} on:mousemove={onMouseMove} on:keydown={handleKeydown} on:keyup={handleKeyup}/>

<div
        class="panel"
        style="left: {panelConfig.left}px; top: {panelConfig.top}px; z-index: {panelConfig.zIndex};"
        bind:this={panelElement}
        in:fly|global={{y: -30, duration: 200, easing: quintOut}}
        out:fly|global={{y: -30, duration: 200, easing: quintOut}}
>
    <!-- svelte-ignore a11y-no-static-element-interactions -->
    <div
            class="title"
            on:mousedown={onMouseDown}
            on:contextmenu|preventDefault={toggleExpanded}
    >
        <img
                class="icon"
                src="img/clickgui/icon-{category.toLowerCase()}.svg"
                alt="icon"
        />
        <span class="category">{category}</span>

        <!-- svelte-ignore a11y_consider_explicit_label -->
        <button class="expand-toggle" on:click={toggleExpanded}>
            <div class="icon" class:expanded={panelConfig.expanded}></div>
        </button>
    </div>

    <div class="modules" on:scroll={handleModulesScroll} bind:this={modulesElement}>
        {#each renderedModules as {name, enabled, description, aliases} (name)}
            <Module {name} {enabled} {description} {aliases}/>
        {/each}
    </div>
</div>

<style lang="scss">
  @use "../../colors.scss" as *;

  .panel {
    border-radius: 16px;
    width: 200px;
    position: absolute;
    overflow: hidden;
    will-change: transform;
    transition: none;
    user-select: none;
    border: 1px solid rgba(white, 0.1);
  }

  .title {
    display: grid;
    grid-template-columns: max-content 1fr max-content;
    align-items: center;
    column-gap: 12px;
    background-color: $base;
    border-bottom: solid 2px $accent;
    padding: 10px 15px;
    cursor: grab;

    .category {
      font-size: 14px;
      color: $text;
      font-weight: 900;
    }
  }

  .modules {
    max-height: 600px;
    overflow-y: auto;
    overflow-x: hidden;
    background-color: rgba($base, 0.8);
  }

  .modules::-webkit-scrollbar {
    width: 0;
  }

  .expand-toggle {
    background-color: transparent;
    border: none;
    cursor: pointer;

    .icon {
      height: 12px;
      width: 12px;
      position: relative;

      &::before {
        content: "";
        position: absolute;
        background-color: white;
        transition: transform 0.4s ease-out;
        top: 0;
        left: 50%;
        width: 2px;
        height: 100%;
        margin-left: -1px;
      }

      &::after {
        content: "";
        position: absolute;
        background-color: white;
        transition: transform 0.4s ease-out;
        top: 50%;
        left: 0;
        width: 100%;
        height: 2px;
        margin-top: -1px;
      }

      &.expanded {
        &::before {
          transform: rotate(90deg);
        }

        &::after {
          transform: rotate(180deg);
        }
      }
    }
  }
</style>