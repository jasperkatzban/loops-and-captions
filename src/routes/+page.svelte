<script lang="ts">
    import Item from "$lib/item.svelte";
    import { videoItems } from "$lib/videos";
</script>

<div class="content">
    <div class="grid">
        {#each videoItems as videoItem}
            <Item src={videoItem.src} caption={videoItem.caption} />
        {/each}
    </div>
</div>

<style>
    .grid {
        width: 100%;
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(20rem, 1fr));
        align-items: stretch;
        gap: 2rem;
    }
</style>
