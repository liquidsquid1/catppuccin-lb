<script lang="ts">
    import {onMount} from "svelte";
    import {
        getModuleSettings,
        setModuleSettings,
        setModuleEnabled,
    } from "../../integration/rest";
    import type {ConfigurableSetting} from "../../integration/types";
    import GenericSetting from "./setting/common/GenericSetting.svelte";
    import {slide} from "svelte/transition";
    import {quintOut} from "svelte/easing";
    import {description as descriptionStore, highlightModuleName} from "./clickgui_store";
    import {setItem} from "../../integration/persistent_storage";
    import {convertToSpacedString, spaceSeperatedNames} from "../../theme/theme_config";
    import {scaleFactor} from "./clickgui_store";

    export let name: string;
    export let enabled: boolean;
    export let description: string;
    export let aliases: string[];

    let moduleNameElement: HTMLElement;
    let configurable: ConfigurableSetting;
    const path = `clickgui.${name}`;
    let expanded = false;

    onMount(async () => {
        configurable = await getModuleSettings(name);

        setTimeout(() => {
            expanded = localStorage.getItem(path) === "true"
        }, 500);
    });

    highlightModuleName.subscribe((m) => {
        if (name !== m) {
            return;
        }

        setTimeout(() => {
            if (!moduleNameElement) {
                return;
            }
            moduleNameElement.scrollIntoView({
                behavior: "smooth",
                block: "center",
            });
        }, 1000);
    });

    async function updateModuleSettings() {
        await setModuleSettings(name, configurable);
        configurable = await getModuleSettings(name);
    }

    async function toggleModule() {
        await setModuleEnabled(name, !enabled);
    }

    function setDescription() {
        const y = (moduleNameElement?.getBoundingClientRect().top ?? 0) + ((moduleNameElement?.clientHeight ?? 0) / 2);
        const x = moduleNameElement?.getBoundingClientRect().right ?? 0;
        let moduleDescription = description;
        if (aliases.length > 0) {
            moduleDescription += ` (aka ${aliases.map(name => $spaceSeperatedNames ? convertToSpacedString(name) : name).join(", ")})`;
        }
        descriptionStore.set({
            x: x * (2 / $scaleFactor),
            y: y * (2 / $scaleFactor),
            description: moduleDescription
        });
    }

    async function toggleExpanded() {
        expanded = !expanded;
        await setItem(path, expanded.toString());
    }
</script>

<!-- svelte-ignore a11y-no-static-element-interactions -->
<div
        class="module"
        class:expanded
        class:has-settings={configurable?.value.length > 2}
        in:slide={{ duration: 500, easing: quintOut }}
        out:slide={{ duration: 500, easing: quintOut }}
>
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <div
            class="name"
            on:contextmenu|preventDefault={toggleExpanded}
            on:click={toggleModule}
            on:mouseenter={setDescription}
            on:mouseleave={() => descriptionStore.set(null)}
            bind:this={moduleNameElement}
            class:enabled
            class:highlight={name === $highlightModuleName}
            class:dim={$highlightModuleName !== null && name !== $highlightModuleName}
    >
        {$spaceSeperatedNames ? convertToSpacedString(name) : name}
    </div>

    {#if expanded && configurable}
        <div class="settings">
            {#each configurable.value as setting (setting.name)}
                <GenericSetting skipAnimationDelay={true} {path} bind:setting on:change={updateModuleSettings}/>
            {/each}
        </div>
    {/if}
</div>

<style lang="scss">
  @use "../../colors.scss" as *;

  .module {
    position: relative;

    .name {
      cursor: pointer;
      transition: ease background-color 0.5s, ease color 0.5s, ease filter 0.5s;
      color: $subtext0;
      text-align: center;
      font-size: 12px;
      font-weight: 500;
      position: relative;
      padding: 8px;

      &.dim {
        filter: opacity(75%) blur(6px) brightness(50%);
      }

      &.highlight::before {
        content: "";
        position: absolute;
        top: 0;
        left: 0;
        width: calc(100% - 4px);
        height: calc(100% - 4px);
        border: solid 2px $accent;
      }

      &:hover {
        background-color: rgba($base, 0.85);
        color: $text;
      }

      &.enabled {
        background-color: $accent;
        box-shadow: 0px 0px 12px $accent;
        color: $base;
        font-weight: 900;

        &::after {
          filter: brightness(0);
          opacity: 0.9;
        }
      }
    }

    .settings {
      background-color: rgba($crust, 0.75);
      padding: 0 11px 0 7px;
    }

    &.has-settings {
      .name::after {
        content: "";
        display: block;
        position: absolute;
        height: 10px;
        width: 10px;
        right: 15px;
        top: 50%;
        background-image: url("/img/clickgui/icon-settings-expand.svg");
        background-position: center;
        background-repeat: no-repeat;
        opacity: 0.5;
        transform-origin: 50% 50%;
        transform: translateY(-50%) rotate(-90deg);
        transition: ease opacity 0.2s,
        ease transform 0.4s;
      }

      &.expanded .name::after {
        transform: translateY(-50%) rotate(0);
        opacity: 1;
      }
    }
  }
</style>
