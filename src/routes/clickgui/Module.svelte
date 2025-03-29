<script lang="ts">
    import { onMount } from "svelte";
    import {
        getModuleSettings,
        setModuleSettings,
        setModuleEnabled,
        getModule,
        getPrintableKeyName,
    } from "../../integration/rest";
    import type { ConfigurableSetting } from "../../integration/types";
    import GenericSetting from "./setting/common/GenericSetting.svelte";
    import { slide } from "svelte/transition";
    import { quintOut } from "svelte/easing";
    import {
        description as descriptionStore,
        highlightModuleName,
    } from "./clickgui_store";
    import { setItem } from "../../integration/persistent_storage";
    import {
        convertToSpacedString,
        spaceSeperatedNames,
    } from "../../theme/theme_config";
    import { scaleFactor } from "./clickgui_store";
    import { Keyboard } from "lucide-svelte";
    import { listen } from "../../integration/ws";
    import type {
        KeyboardKeyEvent,
        MouseButtonEvent,
    } from "../../integration/events";

    export let name: string;
    export let enabled: boolean;
    export let description: string;
    export let aliases: string[];
    export let keyBind: number | { boundKey: string; action: string };

    let moduleNameElement: HTMLElement;
    let configurable: ConfigurableSetting;
    const path = `clickgui.${name}`;
    let expanded = false;
    let keyBindName = "";
    let hasKeyBind = false;
    let binding = false;
    let isHovered = false;
    const UNKNOWN_KEY = "key.keyboard.unknown";

    async function updateKeyBind() {
        try {
            const moduleDetails = await getModule(name);

            if (
                moduleDetails.keyBind &&
                typeof moduleDetails.keyBind === "object" &&
                "boundKey" in moduleDetails.keyBind
            ) {
                const boundKey = moduleDetails.keyBind.boundKey;

                if (boundKey && boundKey !== UNKNOWN_KEY) {
                    try {
                        const printableKey =
                            await getPrintableKeyName(boundKey);
                        if (printableKey && printableKey.localized) {
                            keyBindName = printableKey.localized;
                            hasKeyBind = true;
                        } else {
                            hasKeyBind = false;
                            keyBindName = "";
                        }
                    } catch (error) {
                        console.error(
                            "Failed to get printable key name:",
                            error,
                        );
                        hasKeyBind = false;
                        keyBindName = "";
                    }
                } else {
                    hasKeyBind = false;
                    keyBindName = "";
                }
            } else {
                hasKeyBind = false;
                keyBindName = "";
            }
        } catch (error) {
            console.error("Failed to get module details:", error);
            hasKeyBind = false;
            keyBindName = "";
        }
    }

    function toggleBinding(event: MouseEvent) {
        event.stopPropagation();

        if (binding) {
            const settingWithBind = configurable.value.find(
                (s) => s.valueType === "KEY" || s.valueType === "BIND",
            );
            if (settingWithBind) {
                if (settingWithBind.valueType === "KEY") {
                    settingWithBind.value = UNKNOWN_KEY;
                } else if (
                    settingWithBind.valueType === "BIND" &&
                    typeof settingWithBind.value === "object" &&
                    "boundKey" in settingWithBind.value
                ) {
                    settingWithBind.value.boundKey = UNKNOWN_KEY;
                }
                updateModuleSettings();
            }
            binding = false;
        } else {
            binding = true;
        }
    }

    listen("keyboardKey", async (e: KeyboardKeyEvent) => {
        if (
            e.screen === undefined ||
            !e.screen.class.startsWith("net.ccbluex.liquidbounce") ||
            !(e.screen.title === "ClickGUI" || e.screen.title === "VS-CLICKGUI")
        ) {
            return;
        }

        if (!binding) {
            return;
        }

        binding = false;

        const settingWithBind = configurable.value.find(
            (s) => s.valueType === "KEY" || s.valueType === "BIND",
        );
        if (settingWithBind) {
            if (e.keyCode !== 256) {
                if (settingWithBind.valueType === "KEY") {
                    settingWithBind.value = e.key;
                } else if (
                    settingWithBind.valueType === "BIND" &&
                    typeof settingWithBind.value === "object" &&
                    "boundKey" in settingWithBind.value
                ) {
                    settingWithBind.value.boundKey = e.key;
                }
            } else {
                if (settingWithBind.valueType === "KEY") {
                    settingWithBind.value = UNKNOWN_KEY;
                } else if (
                    settingWithBind.valueType === "BIND" &&
                    typeof settingWithBind.value === "object" &&
                    "boundKey" in settingWithBind.value
                ) {
                    settingWithBind.value.boundKey = UNKNOWN_KEY;
                }
            }

            await updateModuleSettings();
            await updateKeyBind();
        }
    });

    listen("mouseButton", async (e: MouseButtonEvent) => {
        if (
            e.screen === undefined ||
            !e.screen.class.startsWith("net.ccbluex.liquidbounce") ||
            !(e.screen.title === "ClickGUI" || e.screen.title === "VS-CLICKGUI")
        ) {
            return;
        }

        if (!binding || (e.button === 0 && isHovered)) {
            return;
        }

        binding = false;

        const settingWithBind = configurable.value.find(
            (s) => s.valueType === "KEY" || s.valueType === "BIND",
        );
        if (settingWithBind) {
            if (settingWithBind.valueType === "KEY") {
                settingWithBind.value = e.key;
            } else if (
                settingWithBind.valueType === "BIND" &&
                typeof settingWithBind.value === "object" &&
                "boundKey" in settingWithBind.value
            ) {
                settingWithBind.value.boundKey = e.key;
            }

            await updateModuleSettings();
            await updateKeyBind();
        }
    });

    onMount(async () => {
        configurable = await getModuleSettings(name);

        setTimeout(() => {
            expanded = localStorage.getItem(path) === "true";
        }, 500);

        await updateKeyBind();
    });

    listen("keybindChange", updateKeyBind);

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
        await updateKeyBind();
    }

    async function toggleModule() {
        await setModuleEnabled(name, !enabled);
    }

    function setDescription() {
        if (!moduleNameElement) return;

        const boundingRect = moduleNameElement.getBoundingClientRect();
        const y =
            (boundingRect.top + moduleNameElement.clientHeight / 2) *
            (2 / $scaleFactor);

        let moduleDescription = description;
        if (aliases.length > 0) {
            moduleDescription += ` (aka ${aliases
                .map((name) =>
                    $spaceSeperatedNames ? convertToSpacedString(name) : name,
                )
                .join(", ")})`;
        }

        if (window.innerWidth - boundingRect.right > 300) {
            const x = boundingRect.right * (2 / $scaleFactor);
            descriptionStore.set({
                x,
                y,
                anchor: "right",
                description: moduleDescription,
            });
        } else {
            const x = boundingRect.left * (2 / $scaleFactor);

            descriptionStore.set({
                x,
                y,
                anchor: "left",
                description: moduleDescription,
            });
        }
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
    >
        <div class="module-content">
            <span class="module-name">
                {$spaceSeperatedNames ? convertToSpacedString(name) : name}
            </span>

            <!-- Keybind indicator that's also clickable to set keybinds -->
            {#if binding}
                <div
                    class="keybind-container binding"
                    on:click|stopPropagation={toggleBinding}
                    on:mouseenter={() => (isHovered = true)}
                    on:mouseleave={() => (isHovered = false)}
                >
                    <span class="keybind">Press Any Key</span>
                </div>
            {:else if hasKeyBind}
                <div
                    class="keybind-container"
                    on:click|stopPropagation={toggleBinding}
                    on:mouseenter={() => (isHovered = true)}
                    on:mouseleave={() => (isHovered = false)}
                >
                    <span class="keybind">{keyBindName}</span>
                </div>
            {:else if configurable?.value.some((setting) => setting.valueType === "KEY" || setting.valueType === "BIND")}
                <div
                    class="keybind-container"
                    on:click|stopPropagation={toggleBinding}
                    on:mouseenter={() => (isHovered = true)}
                    on:mouseleave={() => (isHovered = false)}
                >
                    <span class="keybind no-key">
                        <Keyboard size={18} />
                    </span>
                </div>
            {/if}
        </div>
    </div>

    {#if expanded && configurable}
        <div class="settings">
            {#each configurable.value.filter((setting) => !(setting.name === "Bind" && (setting.valueType === "KEY" || setting.valueType === "BIND"))) as setting (setting.name)}
                <GenericSetting
                    {path}
                    bind:setting
                    on:change={updateModuleSettings}
                />
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
            transition:
                ease background 0.2s,
                ease color 0.2s,
                ease text-shadow 0.2s;

            color: $overlay1;
            text-align: center;
            font-size: 12px;
            font-weight: 500;
            position: relative;
            padding: 10px;
            display: flex;
            justify-content: space-between;
            align-items: center;

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
                background: rgba($base, 0.85);
                color: $text;
            }

            &.enabled {
                background: linear-gradient(to right, $flamingo, $mauve);
                text-shadow: 0px 0px 8px $base;
                color: $base;
            }

            .module-content {
                display: flex;
                align-items: center;
                justify-content: space-between;
                width: calc(100% - 25px);
                margin: 0 auto;
            }

            .module-name {
                margin-right: auto;
            }

            .keybind-container {
                display: flex;
                align-items: center;
                justify-content: center;
                background-color: rgba($mantle, 0.8);
                border-radius: 5px;
                margin-right: 10px;
                min-width: 24px;
                height: 20px;
                padding: 0 5px;
                width: auto;
                cursor: pointer;
                transition: background-color 0.2s ease;

                &:hover {
                    background-color: rgba($mantle, 1);
                }

                &.binding {
                    background-color: rgba($accent, 0.3);
                    width: auto;
                }

                .keybind {
                    color: $accent;
                    font-size: 10px;
                    opacity: 0.8;
                    display: flex;
                    align-items: center;
                    justify-content: center;

                    &.no-key {
                        opacity: 0.5;
                    }
                }
            }
        }

        .settings {
            background-color: rgba($base, 0.5);
            border-left: solid 4px $accent;
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
                transition:
                    ease opacity 0.2s,
                    ease transform 0.4s;
            }

            &.expanded .name::after {
                transform: translateY(-50%) rotate(0);
                opacity: 1;
            }
        }
    }
</style>
