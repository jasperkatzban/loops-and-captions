<script lang="ts">
    import Item from "$lib/item.svelte";
    import TextItem from "../lib/textItem.svelte";
    import { itemManifest, allCaptions } from "$lib/itemManifest";
    import { onMount } from "svelte";

    interface manifestItem {
        src: string;
        validCaptions: string[];
    }

    interface videoItem {
        src: string;
        caption: string;
    }

    const MAX_SHUFFLE_ATTEMPTS = 2000;

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

    function shuffleVideoItems(manifest: videoItem[]) {
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

        let availableCaptions = new Set<string>();

        let attempts = 0;

        const attemptCaptionShuffle = () => {
            // reset available captions
            availableCaptions = createAvailableCaptions();

            // pick captions from the list
            itemManifest.forEach((item: manifestItem) => {
                let trialCaption: string;
                let attempts = 0;
                let captionsToTry = $state.snapshot(item.validCaptions);

                do {
                    // if there are no more valid captions for the item left,
                    // this iteration is failed and should restart
                    if (captionsToTry.length == 0) {
                        return false;
                    }

                    let range = captionsToTry.length - 1;
                    let i = Math.round(Math.random() * range);
                    // random caption chosen
                    trialCaption = captionsToTry[i];

                    // ensure we don't try the same option again
                    captionsToTry.splice(i, 1);

                    attempts++;
                } while (!availableCaptions.has(trialCaption));

                let chosenCaption = trialCaption;
                availableCaptions.delete(chosenCaption);

                let index = videoItems.findIndex(
                    (videoItem: videoItem) => videoItem.src == item.src,
                );

                attemptedVideoItems[index].caption = chosenCaption;
            });
            return true;
        };

        do {
            attempts++;
            if (attemptCaptionShuffle() == false) {
                continue;
            }
        } while (availableCaptions.size > 0 && attempts < MAX_SHUFFLE_ATTEMPTS);

        // set the entire state array to ensure new captions are displayed all at once
        videoItems = attemptedVideoItems;
    }

    function createVideoItems(manifest: any) {
        let videoItems = manifest.map((item: manifestItem) => {
            return { src: item.src, caption: item.validCaptions[0] };
        });

        shuffleVideoItems(videoItems);

        return videoItems;
    }

    // await the proper mapping of captions to video items
    function getVideoItems() {
        return Promise.resolve(videoItems);
    }

    // handle positioning the credits tag nicely in the grid
    let grid: HTMLElement | null = $state(null);
    let gridColumnCount = $state(4);
    let numberOfSpacerItems = $derived(
        calculateNumberOfSpacerItems(gridColumnCount),
    );

    onMount(() => {
        window.addEventListener("resize", setNumberOfColumns);

        return () => {
            window.removeEventListener("resize", setNumberOfColumns);
        };
    });

    $effect(() => {
        grid = document.getElementById("content-grid");
    });

    function setNumberOfColumns() {
        if (grid) {
            const gridComputedStyle = window.getComputedStyle(grid);
            // get number of grid columns
            gridColumnCount = gridComputedStyle
                .getPropertyValue("grid-template-columns")
                .split(" ").length;
        }
    }

    // compute number of spacer items based on the number of columns
    function calculateNumberOfSpacerItems(gridColumnCount: number) {
        let n = 0;

        if (gridColumnCount < 3) {
            n = 0;
        } else if (gridColumnCount < 5) {
            n = 1;
        } else {
            n = 0;
        }
        return n;
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
        <div class="grid" id="content-grid">
            {#await getVideoItems() then readyVideoItems}
                {#each readyVideoItems as videoItem, i}
                    <Item
                        src={videoItem.src}
                        caption={videoItem.caption}
                        bind:videoIsLoaded={videoStatus[i]}
                    />
                {/each}
            {/await}
            {#each { length: numberOfSpacerItems }}
                <TextItem />
            {/each}
            <TextItem credits={true} />
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
