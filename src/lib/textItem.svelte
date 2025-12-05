<script lang="ts">
    let { credits = false } = $props();
</script>

<div class="item">
    <div class="video-wrapper">
        <img src={"blank.png"} alt="" />
    </div>
    {#if credits}
        <div class="caption">
            <p>
                made by <a
                    href="https://jasperkatzban.com"
                    target="_blank"
                    rel="noopener noreferrer">jasper</a
                >
            </p>
        </div>
    {/if}
</div>

<style>
    .video-wrapper {
        width: 100%;
        display: flex;
    }

    img {
        width: 100%;
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
        font-size: 1.1rem;
        margin-top: 1rem;
        margin-bottom: 0rem;
    }

    a {
        color: black;
    }

    a:hover {
        color: darkolivegreen;
    }
</style>
