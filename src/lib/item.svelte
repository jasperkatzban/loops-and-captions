<script lang="ts">
    import { fade } from "svelte/transition";
    import { inview } from "svelte-inview";
    import type { Options } from "svelte-inview";

    const randomizeRootMargin = () => {
        const pixels = 10 + Math.floor(Math.random() * 40);

        return pixels + "px";
    };

    const options: Options = {
        rootMargin: randomizeRootMargin(),
    };

    let videoRef: HTMLMediaElement;

    let { src, caption, videoIsLoaded = $bindable(false) } = $props();

    let readyState = $state(0);

    $effect(() => {
        // only set the loaded flag once video is loaded the first time
        //  to avoid it being unset when the video restarts
        if (readyState >= HTMLMediaElement.HAVE_ENOUGH_DATA) {
            videoIsLoaded = true;
        }
    });
</script>

<div class="item">
    <div
        class="video-wrapper"
        use:inview={options}
        oninview_enter={() => videoRef.play()}
        oninview_leave={() => videoRef.pause()}
    >
        <video
            {src}
            ontouchmove={(e) => {
                e.preventDefault();
            }}
            disablePictureInPicture={true}
            bind:readyState
            bind:this={videoRef}
            loop
            muted
        >
            <track kind="captions" />
        </video>
    </div>
    <div class="caption">
        {#key caption}
            <p in:fade>{caption}</p>
        {/key}
    </div>
</div>

<style>
    .video-wrapper {
        width: 100%;
        display: flex;
    }

    video {
        width: 100%;
        filter: sepia(20%);
        filter: saturate(60%);
        filter: contrast(80%);
        border-radius: 0.2rem;
    }

    .caption {
        display: flex;
        height: 1.8rem;
    }

    .caption p {
        width: 100%;
        text-align: center;
        font-family: "Lexend", sans-serif;
        font-optical-sizing: auto;
        font-weight: 400;
        font-style: normal;
        font-size: 1.1rem;
        margin-top: 1rem;
        margin-bottom: 0rem;
    }
</style>
