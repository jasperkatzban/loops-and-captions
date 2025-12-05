<script lang="ts">
    import Item from "$lib/item.svelte";
    import EmptyItem from "../lib/emptyItem.svelte";
    import { itemManifest, allCaptions } from "$lib/itemManifest";

    interface manifestItem {
        src: string;
        validCaptions: string[];
    }

    // sort items ascending based on how many captions are valid
    itemManifest.sort(
        (a, b) => a.validCaptions.length - b.validCaptions.length,
    );

    // create stateful list of video items based on item Manifest
    let videoItems = $state(createVideoItems(itemManifest));

    let allVideosLoaded = $state(false);

    // set loading status to stateful variable
    let videoStatus = $state(
        Array.apply(null, Array(videoItems.length)).map(() => false),
    );

    // check if videos are loaded
    $effect(() => {
        allVideosLoaded = !videoStatus.includes(false);
    });

    function shuffleManifest(manifest: manifestItem[]) {
        for (let i = manifest.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [manifest[i], manifest[j]] = [manifest[j], manifest[i]];
        }
        return manifest;
    }

    function createAvailableCaptions() {
        let availableCaptions = new Set<string>();

        allCaptions.forEach((caption) => {
            availableCaptions.add(caption);
        });
        return availableCaptions;
    }

    function shuffleCaptions() {
        // temporary buffer to store shuffle attempts
        const attemptedVideoItems = videoItems;

        let availableCaptions: Set<string>;

        do {
            // reset available captions
            availableCaptions = createAvailableCaptions();

            // pick captions from the list
            itemManifest.forEach((item) => {
                let chosenCaption: string;
                let attempts = 0;
                let trialCaptions = item.validCaptions;

                do {
                    let range = trialCaptions.length - 1;
                    let i = Math.round(Math.random() * range);
                    // random caption chosen
                    chosenCaption = trialCaptions[i];

                    attempts++;
                } while (
                    !availableCaptions.has(chosenCaption) &&
                    attempts < 10
                );

                availableCaptions.delete(chosenCaption);

                let index = videoItems.findIndex(
                    (videoItem) => videoItem.src == item.src,
                );

                attemptedVideoItems[index].caption = chosenCaption;
            });
        } while (availableCaptions.size > 0);

        videoItems = attemptedVideoItems;
        console.log(availableCaptions);
    }

    function createVideoItems(manifest: any) {
        let shuffledManifest = shuffleManifest(manifest);

        return shuffledManifest.map((item: manifestItem) => {
            return { src: item.src, caption: item.validCaptions[0] };
        });
    }

    // await the proper mapping of captions to video items
    function getVideoItems() {
        return Promise.resolve(videoItems);
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
            {#await getVideoItems() then readyVideoItems}
                {#each readyVideoItems as videoItem, i}
                    <Item
                        src={videoItem.src}
                        caption={videoItem.caption}
                        bind:videoIsLoaded={videoStatus[i]}
                    />
                {/each}
            {/await}
            <EmptyItem />
            <EmptyItem caption="made by jasper" />
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
