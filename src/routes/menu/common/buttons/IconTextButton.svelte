<script lang="ts">
    import {createEventDispatcher} from "svelte";

    export let title: string;
    export let icon: string;
    export let disabled = false;

    const dispatch = createEventDispatcher();
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->
<button class="icon-text-button" on:click={() => dispatch("click")} {disabled}>
    <div class="icon">
        <img src="img/menu/{icon}" alt={title}>
    </div>
    <div class="title">{title}</div>
</button>

<style lang="scss">
    @use "../../../../colors.scss" as *;

    .icon-text-button {
      display: flex;
      border: none;

      border-radius: 16px;
      border: 1px solid rgba(white, 0.1);
      background: rgba($base, 0.75);
      align-items: center;
      overflow: hidden;
      transition: ease opacity .2s, transform .3s ease;

      &:not([disabled]):hover {
        &:hover {
          cursor: pointer;
          transform: translateY(-4px);
        }
      }

      &[disabled] {
        opacity: .6;
      }
    }

    .icon {
      height: 58px;
      width: 58px;
      background: rgba($crust, 0.75);
      display: flex;
      align-items: center;
      justify-content: center;
    }

    .title {
      font-size: 20px;
      font-weight: 500;
      color: $accent;
      padding: 0 30px;
    }
</style>