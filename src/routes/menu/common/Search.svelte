<script lang="ts">
    import {createEventDispatcher} from "svelte";

    let value = "";

    const dispatch = createEventDispatcher<{
        search: { query: string }
    }>();
</script>

<input class="search" type="text" placeholder="Search..." bind:value={value}
       on:input={() => dispatch("search", {query: value})} spellcheck="false">

<style lang="scss">
  @use "../../../colors.scss" as *;

  .search {
    background-color: rgba($base, 0.8);
    color: $text;
    font-family: "Inter", sans-serif;
    font-size: 20px;
    border-radius: 12px;
    border: none;
    border-bottom: solid 4px $accent;
    background-image: url("/img/menu/icon-search.svg");
    background-repeat: no-repeat;
    background-position: 20px center;
    padding: 20px 20px 20px 60px;
    flex: 1;
  }
</style>