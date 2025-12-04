<script lang="ts">
    import Item from "$lib/item.svelte";
    import EmptyItem from "../lib/emptyItem.svelte";
    import { videoItems } from "$lib/videos";

    let allVideosLoaded = $state(false);

    let videoStatus = $state(
        Array.apply(null, Array(videoItems.length)).map(() => false),
    );

    $effect(() => {
        allVideosLoaded = !videoStatus.includes(false);
    });

    function shuffle() {
        return Promise.resolve(fisherYatesShuffle(videoItems));
    }

    function fisherYatesShuffle(arr: { src: string; caption: string }[]) {
        for (let i = arr.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [arr[i], arr[j]] = [arr[j], arr[i]];
        }
        return arr;
    }
</script>

<div class="page">
    <div class="content" style:display={allVideosLoaded ? "block" : "none"}>
        <div class="grid">
            {#await shuffle() then shuffledVideoItems}
                {#each shuffledVideoItems as videoItem, i}
                    <Item
                        src={videoItem.src}
                        caption={videoItem.caption}
                        bind:videoIsLoaded={videoStatus[i]}
                    />
                {/each}
                <EmptyItem />
                <EmptyItem caption="made by jasper" />
            {/await}
        </div>
    </div>
    <div class="loader" style:display={allVideosLoaded ? "none" : "flex"}>
        <p>loading...</p>
    </div>
</div>

<style>
    .page {
        height: 100vh;
    }

    .content {
        padding: 5rem;
    }

    .grid {
        width: 100%;
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(13rem, 1fr));
        align-items: stretch;
        gap: 6rem;
        margin-bottom: -2rem;
    }

    .loader {
        display: flex;
        justify-content: center;
        align-items: center;
        width: 100%;
        height: 100%;
    }

    .loader p {
        text-align: center;
        font-family: "Lexend", sans-serif;
        font-optical-sizing: auto;
        font-weight: 400;
        font-style: normal;
        font-size: 1.1rem;
        margin-block: 0.5rem;
    }
</style>
