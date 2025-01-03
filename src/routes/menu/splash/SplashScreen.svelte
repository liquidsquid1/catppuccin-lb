<script lang="ts">
  import { expoInOut } from "svelte/easing";


  import { scale } from "svelte/transition";

</script>
<div class="wrapper" in:scale={{ duration: 200, easing: expoInOut }} out:scale={{ duration: 200, easing: expoInOut }}>
  <div class="loader-wrapper" in:scale={{ duration: 1000, easing: expoInOut }} out:scale={{ duration: 1000, easing: expoInOut }}>
    <div class="lds-ring">
        <div></div>
        <div></div>
        <div></div>
        <div></div>
    </div>
</div>
</div>

<style lang="scss">

  @use "../././../../colors.scss" as *;

  .wrapper {
    width: 100vw;
    height: 100vh;
    margin: 0;
    background: 
    repeating-conic-gradient($crust 0% 25%, $base 0% 50%) 
      50% / 50px 50px;
  }

  @import "../../../colors";
  .loader-wrapper {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translateY(-50%, -50%) scale(150%);
    width: 32px;
    height: 32px;
  }
  .lds-ring,
  .lds-ring div {
    box-sizing: border-box;
    position: relative;
    color: white;
  }
  .lds-ring {
    display: inline-block;
    width: 32px;
    height: 32px;
  }
  .lds-ring div {
    box-sizing: border-box;
    display: block;
    position: absolute;
    width: 25px;
    height: 25px;
    margin: 4px;
    border: 4px solid currentColor;
    border-radius: 50%;
    animation: lds-ring 1.2s cubic-bezier(0.5, 0, 0.5, 1) infinite;
    border-color: currentColor transparent transparent transparent;
  }
  .lds-ring div:nth-child(1) {
    animation-delay: -0.45s;
  }
  .lds-ring div:nth-child(2) {
    animation-delay: -0.3s;
  }
  .lds-ring div:nth-child(3) {
    animation-delay: -0.15s;
  }
  @keyframes lds-ring {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }

</style>
