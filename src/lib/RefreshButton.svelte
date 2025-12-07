<script lang="ts">
    import { onMount } from "svelte";
    import * as imgSrcs from "$lib/assets/refresh-button/";
    type imgSrcKey = keyof typeof imgSrcs;

    // preload all source images for button
    function preload() {
        let imgPreloads = Object.keys(imgSrcs).map((key) => {
            return new Promise((resolve) => {
                let src = imgSrcs[key as imgSrcKey];
                let img = new Image();
                img.onload = resolve;
                img.src = src;
            });
        });
        return Promise.all(imgPreloads);
    }
    let { clickHandler } = $props();

    let buttonImgSrc = $state(imgSrcs["hover0"]);
    let buttonState = $state("normal");
    let buttonArrowIndex = $state(0);

    onMount(() => {
        const interval = setInterval(() => {
            let i = buttonArrowIndex % 16;
            if (buttonState != "active") {
                let index = "hover" + i;
                buttonImgSrc = imgSrcs[index as imgSrcKey];
                if (buttonState == "hovered") {
                    buttonArrowIndex++;
                } else if (buttonState == "normal" && i != 0) {
                    buttonArrowIndex++;
                } else {
                    buttonArrowIndex = 0;
                }
            }
        }, 55);

        return () => {
            clearInterval(interval);
        };
    });
</script>

{#await preload() then _}
    <div class="button-wrapper">
        <button
            aria-label="option"
            onclick={() => {
                clickHandler();
            }}
            onmousedown={() => {
                buttonState = "active";
                buttonImgSrc = imgSrcs["active"];
                buttonArrowIndex = 0;
            }}
            onmouseup={() => {
                buttonState = "normal";
            }}
            onmouseenter={() => {
                buttonState = "hovered";
                buttonArrowIndex = 0;
            }}
            onmouseleave={() => {
                buttonState = "normal";
            }}
        >
            <img
                id="refresh-button-visible-element"
                src={buttonImgSrc}
                alt="two arrows in a cross indicating a shuffling action"
            />
        </button>
    </div>
{/await}

<style>
    .button-wrapper {
        position: fixed;
        bottom: 1.7rem;
        left: 50%;
        margin: 0;
    }

    button {
        margin-left: -1.5rem;
        padding: 0;
        border-style: none;
        background: none;
        cursor: none;
    }

    button img {
        width: 3rem;
    }
</style>
