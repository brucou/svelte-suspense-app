<script>
 import { key } from './util.js';
 import { getContext, onMount } from 'svelte';
 export let id = '';
 export let height = 800;
 let load = false;
 const { registerObserver } = getContext(key);
 onMount(() => {
     const el = document.querySelector('#lazy-'+id);
     registerObserver(el, () => {
         load = true;
     })
 });
 /* registerObserver() */
</script>

<div id="lazy-{id}">
    {#if load}
    <slot></slot>
    {:else}
    <div style="height: {height}px;">
    </div>
    {/if}
</div>
