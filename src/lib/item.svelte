<script lang="ts">
    let { src, caption } = $props();
</script>

<div class="item">
    <div class="video-wrapper">
        <video
            {src}
            ontouchmove={(e) => {
                e.preventDefault();
            }}
            disablePictureInPicture="true"
            autoplay
            loop
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
    }

    .caption p {
        width: 100%;
        text-align: center;
        font-family: "Lexend", sans-serif;
        font-optical-sizing: auto;
        font-weight: 400;
        font-style: normal;
    }
</style>
