<!-- senk ju approved arraylist -->

<script lang="ts">
    import { onMount, tick } from "svelte";
    import type { Module } from "../../../integration/types";
    import { getModules } from "../../../integration/rest";
    import { listen } from "../../../integration/ws";
    import { getTextWidth } from "../../../integration/text_measurement";
    import { convertToSpacedString, spaceSeperatedNames } from "../../../theme/theme_config";
    import { expoOut } from "svelte/easing";
    import { scale } from "svelte/transition";

    let enabledModules: Module[] = [];

    async function updateEnabledModules() {
        const modules = await getModules();
        const visibleModules = modules.filter(m => m.enabled && !m.hidden);

        const modulesWithWidths = visibleModules.map(module => {
                let formattedName = $spaceSeperatedNames ? convertToSpacedString(module.name) : module.name;
                let fullName = module.tag == null ? formattedName : `${formattedName} ${module.tag}`;
                let categoryIconWidth = getTextWidth(" ", "500 14px Inter") + 16;

                return {
                    ...module,
                    width: getTextWidth(fullName, "500 14px Inter") + categoryIconWidth
                };
            }
        );

        modulesWithWidths.sort((a, b) => b.width - a.width);

        enabledModules = modulesWithWidths;
        await tick();
    }

    spaceSeperatedNames.subscribe(async () => {
        await updateEnabledModules();
    });

    onMount(async () => {
        await updateEnabledModules();
    });

    listen("moduleToggle", async () => {
        await updateEnabledModules();
    });

    listen("refreshArrayList", async () => {
        await updateEnabledModules();
    });

    function getIconPath(category: string): string {
        return `img/clickgui/icon-${category.toLowerCase()}.svg`;
    }
</script>

<div class="arraylist">
    {#each enabledModules as {name, tag, category} (name)}
        <div class="module" transition:scale={{ duration: 500, easing: expoOut }}>
            {$spaceSeperatedNames ? convertToSpacedString(name) : name}
            {#if tag}
                <span class="tag"> {tag}</span>
            {/if}
            <span class="category">
                <img src={getIconPath(category)} alt={category}>
            </span>
        </div>
    {/each}
</div>

<style lang="scss">
    @use "../../../colors.scss" as *;
  
    .arraylist {
      display: flex;
      flex-direction: column;
      gap: 2px;
    }
  
    .module {
      background: rgba($base, 0.8);
      box-shadow: 0px 0px 6px rgba($base, 0.8);
      border-radius: 8px;
      border-top-right-radius: 0px;
      border-bottom-right-radius: 0px;
      border-right: 4px solid $accent;
      color: $text;
      font-size: 14px;
      padding-inline: 12px;
      padding-block: 4px;
      width: max-content;
      font-weight: 500;
      margin-left: auto;
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 4px;
    }
  
    .tag {
      color: $overlay1;
      text-shadow: none;
    }

    .category {
      display: flex;
      align-items: center;
    }

    .category img {
      width: 16px;
      height: 16px;
      filter: brightness(75%);
    }
</style>
