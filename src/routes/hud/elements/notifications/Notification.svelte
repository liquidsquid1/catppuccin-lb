<script lang="ts">
  export let title: string;
  export let message: string;
  export let severity: string;
</script>

<div class="notification">
  <div class="icon {severity.toString().toLowerCase()}"></div>
  <div class="title">{title}</div>
  <div class="message">{message}</div>
</div>

<style lang="scss">

@use "../../../../colors.scss" as *;

.notification {
  display: grid;
  grid-template-areas:
          "a b"
          "a c";
  grid-template-columns: max-content 1fr;
  column-gap: 10px;
  background: rgba($base, 0.8);
  border-radius: 16px;
  width: 300px;
  overflow: hidden;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid rgba(white, 0.1);
}

.icon {
  height: 40px;
  width: 40px;
  background-position: center;
  background-repeat: no-repeat;
  border-radius: 20px;
  grid-area: a;
  transition: background-color 0.2s;
  position: relative;
  background-image: url("/img/hud/notification/icon-toggle.svg");
  background-color: $accent;

  &.success {
    background-image: url("/img/hud/notification/icon-success.svg");
  }

  &.error {
    background-image: url("/img/hud/notification/icon-error.svg");
  }

  &.info {
    background-image: url("/img/hud/notification/icon-info.svg");
  }

  &.disabled,
  &.enabled {
    &::after {
      content: "";
      position: absolute;
      height: 10px;
      width: 10px;
      border-radius: 5px;
      top: 50%;
      transform: translate(-50%, -50%);
      background-color: none;
      transition: all 0.2s ease-out;
    }
  }

  &.enabled::after {
    left: 62%;
  }

  &.disabled::after {
    left: 38%;
  }
}

.title {
  grid-area: b;
  font-size: 14px;
  color: $text;
  font-weight: 900;
}

.message {
  grid-area: c;
  font-size: 12px;
  color: rgba($subtext0, 0.7);
}
</style>
