<script lang="ts">
import Subtraction from './Subtraction.svelte'
import Checksum from './Checksum.svelte'

let tab = 0;
</script>

<main>
  <select id="tab" bind:value={tab}>
    <option value={0}>Subtraction</option>
    <option value={1}>Checksums</option>
  </select>
  {#if tab === 0}
    <Subtraction/>
  {:else}
    <Checksum/>
  {/if}
</main>

<style>
</style>
