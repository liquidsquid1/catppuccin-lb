<script lang="ts">
  export let color: string;
  export let radius: number;
  export let rotateX: number;
  export let rotateZ: number;
</script>

<div class="pointer" style="
  transform-origin: 50% {radius}px;
  transform: translateY(-{radius}px) rotateX({rotateX}deg) rotateZ({rotateZ}deg);">
</div>

<style lang="scss">
@use "../../../../colors.scss" as *;

.pointer {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  position: absolute;
  background: linear-gradient(45deg, rgba($accent, 0.75), rgba($blue, 0.6));
  /* Creating a gradient shadow effect by layering multiple shadows */

}
</style>
