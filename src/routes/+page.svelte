<script lang="ts">
    import Item from "$lib/item.svelte";
    import EmptyItem from "../lib/emptyItem.svelte";
    import { itemManifest, allCaptions } from "$lib/itemManifest";

    // sort items ascending based on how many captions are valid
    itemManifest.sort(
        (a, b) => a.validCaptions.length - b.validCaptions.length,
    );

    let availableCaptions = createAvailableCaptions();

    let videoItems = $state(createVideoItems(itemManifest));
    let shuffledVideoItems = $state(videoItems);

    let allVideosLoaded = $state(false);

    let videoStatus = $state(
        Array.apply(null, Array(shuffledVideoItems.length)).map(() => false),
    );

    $effect(() => {
        allVideosLoaded = !videoStatus.includes(false);
    });

    function createAvailableCaptions() {
        let availableCaptions = new Set();

        allCaptions.forEach((caption) => {
            availableCaptions.add(caption);
        });

        return availableCaptions;
    }

    function createVideoItems(manifest: any) {
        return manifest.map(
            (item: { src: string; validCaptions: string[] }) => {
                return { src: item.src, caption: item.validCaptions[0] };
            },
        );
    }

    function shuffleCaptions() {
        console.log("run");

        // reset available captions
        availableCaptions = createAvailableCaptions();

        // pick captions from the list
        itemManifest.forEach((item) => {
            let chosenCaption: string;

            do {
                chosenCaption = pickCaption(item.validCaptions);
            } while (!availableCaptions.has(chosenCaption));

            availableCaptions.delete(chosenCaption);

            //index of here
            let index = shuffledVideoItems.findIndex(
                (videoItem) => videoItem.src == item.src,
            );
            shuffledVideoItems[index].caption = chosenCaption;
        });
    }

    function pickCaption(validCaptions: string[]) {
        let range = validCaptions.length - 1;
        let i = Math.round(Math.random() * range);

        return validCaptions[i];
    }

    function shuffleItems() {
        return Promise.resolve(fisherYatesShuffle($state.snapshot(videoItems)));
    }

    function fisherYatesShuffle(reference: { src: string; caption: string }[]) {
        let arr = reference;
        for (let i = arr.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [arr[i], arr[j]] = [arr[j], arr[i]];
        }
        return arr;
    }
</script>

<div class="page">
    <div
        class="content"
        style:display={allVideosLoaded ? "block" : "none"}
        onmouseup={() => {
            shuffleCaptions();
        }}
    >
        <div class="grid">
            {#await shuffleItems() then orderedVideoItems}
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
