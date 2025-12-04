<script lang="ts">
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

<div class="item" style:display={videoIsLoaded ? "block" : "none"}>
    <div class="video-wrapper">
        <video
            {src}
            ontouchmove={(e) => {
                e.preventDefault();
            }}
            disablePictureInPicture={true}
            bind:readyState
            autoplay
            loop
            muted
        >
            <track kind="captions" />
        </video>
    </div>
    <div class="caption">
        <p>{caption}</p>
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
        border-radius: 0.15rem;
    }

    .caption {
        display: flex;
    }

    .caption p {
        width: 100%;
        text-align: center;
        font-family: "Lexend", sans-serif;
        font-optical-sizing: auto;
        font-weight: 400;
        font-style: normal;
        margin-block: 0.5rem;
    }
</style>
