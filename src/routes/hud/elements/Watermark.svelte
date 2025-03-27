<script>
    import { Cat } from "lucide-svelte";
</script>

<style lang="scss">
    @use "../../../colors.scss" as *;

    .watermark-wrapper {
        position: relative;
        display: inline-block;
    }

    .glow {
        position: absolute;
        inset: -8px;
        background: linear-gradient(to right, $flamingo, $mauve, $blue);
        border-radius: 1.0rem;
        filter: blur(12px) opacity(25%);
        opacity: 0.7;
    }

    .watermark-content {
        background-color: rgba($base, 0.9);
        backdrop-filter: blur(5px);
        border-radius: 1.0rem;
        border: 1px solid rgba($text, 0.3);
        padding: 0.5rem 0.75rem;
        box-shadow: 0 2px 10px rgba(0, 0, 0, 0.5);
        display: flex;
        align-items: center;
        gap: 0.5rem;
    }

    .watermark-content span {
        font-size: 1.25rem;
        font-weight: bold;
        background: linear-gradient(to right, $flamingo, $mauve, $blue);
        background-clip: text;
        color: transparent;
    }
</style>

<div class="watermark-wrapper">
    <div class="glow"></div>

    <div class="watermark-content">
        <Cat width="16" height="16" color="#cba6f7" />
        <span>catppuccin</span>
    </div>
</div>