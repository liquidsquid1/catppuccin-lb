<script>
    import { Cat } from "lucide-svelte";
</script>

<style lang="scss">
    @use "../../../colors.scss" as *;

    @keyframes gradientAnimation {
        0% { background-position: 0% 50%; }
        50% { background-position: 100% 50%; }
        100% { background-position: 0% 50%; }
    }

    @keyframes pulse {
        0% { opacity: 0.6; filter: blur(12px) opacity(25%); }
        50% { opacity: 0.8; filter: blur(14px) opacity(50%); }
        100% { opacity: 0.6; filter: blur(12px) opacity(25%); }
    }

    .watermark-wrapper {
        position: relative;
        display: inline-block;
    }

    .glow {
        position: absolute;
        inset: -8px;
        background: linear-gradient(to right, $flamingo, $mauve, $blue, $lavender, $mauve);
        background-size: 300% 100%;
        border-radius: 1.0rem;
        opacity: 0.7;
        animation: 
            gradientAnimation 12s ease infinite,
            pulse 8s ease-in-out infinite;
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
        position: relative;
        z-index: 1;
    }

    .watermark-content span {
        font-size: 1.25rem;
        font-weight: bold;
        background: linear-gradient(to right, $flamingo, $mauve, $blue, $mauve);
        background-size: 300% 100%;
        background-clip: text;
        color: transparent;
        animation: gradientAnimation 8s ease infinite;
    }
</style>

<div class="watermark-wrapper">
    <div class="glow"></div>

    <div class="watermark-content">
        <Cat width="16" height="16" color="#cba6f7" />
        <span>catppuccin</span>
    </div>
</div>
