<div class="button-container">
    <slot />
</div>

<style lang="scss">
    @use "../../../../colors.scss" as *;

    .button-container {
      background-color: rgba($base, 0.5);
      padding: 12px 30px;
      width: max-content;
      border-radius: 16px;
      display: flex;
      column-gap: 20px;
    }
</style>