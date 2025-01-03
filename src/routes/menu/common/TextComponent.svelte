<script lang="ts">
    import type {TextComponent as TTextComponent} from "../../../integration/types";

    export let textComponent: TTextComponent | string;
    export let allowPreformatting = false;
    export let preFormattingMonospace = true
    export let inheritedColor = "#ffffff";
    export let inheritedStrikethrough = false;
    export let inheritedItalic = false;
    export let inheritedUnderlined = false;
    export let inheritedBold = false;
    export let fontSize: number;

    const colors: { [name: string]: string } = {
        black: "#11111b", // Final
        dark_blue: "#1e66f5", // Final
        dark_green: "#40a02b", // Final (from Latte)
        dark_aqua: "#04a5e5", // Final (from Latte)
        dark_red: "#d20f39", // Final (from Latte)
        dark_purple: "#cba6f7", // Final (Mauve)
        gold: "#df8e1d", // Final
        gray: "#a6adc8", // Final
        dark_gray: "#6c7086", // Final
        blue: "#89b4fa", // Final
        green: "#a6e3a1", // Final
        aqua: "#89dceb", // Final
        red: "#f38ba8", // Final
        light_purple: "#b4befe", // Mauve 🎉 (thats a lie, its Lavender)
        yellow: "#f9e2af", // Final
        white: "#cdd6f4" // 🎉🎉🎉🎉 I just felt goofy 🤑🤑
    };

    function translateColor(color: string): string {
        if (!color) {
            return colors.white;
        }
        if (color.startsWith("#")) {
            return color;
        } else {
            return colors[color];
        }
    }

    function convertLegacyCodes(text: string) {
        let obfuscated = false;
        let bold = false;
        let strikethrough = false;
        let underlined = false;
        let italic = false;
        let color = colors.black;

        function reset() {
            obfuscated = false;
            bold = false;
            strikethrough = false;
            underlined = false;
            italic = false;
            color = colors.black;
        }

        const components: TTextComponent[] = [];
        const textParts = (text.startsWith("§") ? text : `§f${text}`).split("§");

        for (const p of textParts) {
            const code = p.charAt(0);
            const t = p.slice(1);

            switch (code) {
                case "k":
                    obfuscated = true;
                    break;
                case "l":
                    bold = true;
                    break;
                case "m":
                    strikethrough = true;
                    break;
                case "n":
                    underlined = true;
                    break;
                case "o":
                    italic = true;
                    break;
                case "r":
                    reset();
                    break;
                default:
                    color = colors[Object.keys(colors)[parseInt(code, 16)]] ?? colors.black;
                    break;
            }

            components.push({
                color,
                bold,
                italic,
                underlined,
                obfuscated,
                strikethrough,
                text: t,
            });
        }

        return {
            extra: components
        };
    }
</script>

<span class="text-component">
    {#if typeof textComponent === "string"}
        <svelte:self {fontSize} {allowPreformatting} {preFormattingMonospace} textComponent={convertLegacyCodes(textComponent)}/>
    {:else if textComponent}
        {#if textComponent.text}
            {#if !textComponent.text.includes("§")}
                <span class="text" class:bold={textComponent.bold !== undefined ? textComponent.bold : inheritedBold}
                      class:italic={textComponent.italic !== undefined ? textComponent.italic : inheritedItalic}
                      class:underlined={textComponent.underlined !== undefined ? textComponent.underlined : inheritedUnderlined}
                      class:strikethrough={textComponent.strikethrough !== undefined ? textComponent.strikethrough : inheritedStrikethrough}
                      class:allow-preformatting={allowPreformatting}
                      style="color: {textComponent.color !== undefined ? translateColor(textComponent.color) : translateColor(inheritedColor)}; font-size: {fontSize}px;">{textComponent.text}</span>
            {:else}
                <svelte:self {allowPreformatting} {preFormattingMonospace} {fontSize}
                             inheritedColor={textComponent.color !== undefined ? textComponent.color : inheritedColor}
                             inheritedBold={textComponent.bold !== undefined ? textComponent.bold : inheritedBold}
                             inheritedItalic={textComponent.italic !== undefined ? textComponent.italic : inheritedItalic}
                             inheritedUnderlined={textComponent.underlined !== undefined ? textComponent.underlined : inheritedUnderlined}
                             inheritedStrikethrough={textComponent.strikethrough !== undefined ? textComponent.strikethrough : inheritedStrikethrough}
                             textComponent={convertLegacyCodes(textComponent.text)}/>
            {/if}
        {/if}
        {#if textComponent.extra}
            {#each textComponent.extra as e}
                <svelte:self {allowPreformatting} {preFormattingMonospace} {fontSize}
                             inheritedColor={textComponent.color !== undefined ? textComponent.color : inheritedColor}
                             inheritedBold={textComponent.bold !== undefined ? textComponent.bold : inheritedBold}
                             inheritedItalic={textComponent.italic !== undefined ? textComponent.italic : inheritedItalic}
                             inheritedUnderlined={textComponent.underlined !== undefined ? textComponent.underlined : inheritedUnderlined}
                             inheritedStrikethrough={textComponent.strikethrough !== undefined ? textComponent.strikethrough : inheritedStrikethrough}
                             textComponent={e}/>
            {/each}
        {/if}
    {/if}
</span>

<style>
    .text-component {
        font-size: 0;
    }

    .text {
        display: inline;

        &.allow-preformatting {
            font-family: "Inter";
            white-space: pre;
        }

        &.bold {
            font-weight: 500;
        }

        &.italic {
            font-style: italic;
        }

        &.underlined {
            text-decoration: underline;
        }

        &.strikethrough {
            text-decoration: line-through;
        }
    }
</style>