<script lang="ts">
    let { clickHandler } = $props();
    import { onMount } from "svelte";
    let buttonImgSrc = $state("refresh-button/0.svg");
    let buttonState = $state("normal");
    let buttonArrowIndex = $state(0);

    onMount(() => {
        const interval = setInterval(() => {
            let i = buttonArrowIndex % 16;
            if (buttonState != "active") {
                buttonImgSrc = "refresh-button/" + i + ".svg";
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

<div class="button-wrapper">
    <button
        aria-label="option"
        onclick={() => {
            clickHandler();
        }}
        onmousedown={() => {
            buttonState = "active";
            buttonImgSrc = "refresh-button/active.svg";
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
