<script lang="ts">
    import { onMount } from "svelte";
    import * as imgSrcs from "$lib/assets/cursor/";
    type imgSrcKey = keyof typeof imgSrcs;

    // preload all source images for cursor
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

    let cursorImgSrc = $state(imgSrcs["normal0"]);
    let cursorState = $state("normal");

    let c = 0;

    onMount(() => {
        document.addEventListener("mouseover", (event) => {
            const target = event.target;
            // Check if the hovered element is a link or refresh button
            if (
                target &&
                (target.tagName === "A" ||
                    target.id === "refresh-button-visible-element")
            ) {
                cursorState = "hover";
            } else {
                cursorState = "normal";
            }
        });

        const interval = setInterval(() => {
            if (cursorState == "normal") {
                let i = c % 20;
                let index = "normal" + i;
                cursorImgSrc = imgSrcs[index as imgSrcKey];
                c++;
            } else {
                let i = c % 10;
                let index = "hover" + i;
                cursorImgSrc = imgSrcs[index as imgSrcKey];
                c++;
            }
        }, 55);

        return () => {
            clearInterval(interval);
        };
    });

    const useMousePosition = () => {
        let x = $state<number>(0);
        let y = $state<number>(0);

        const updateMousePosition = (event: MouseEvent) => {
            x = event.clientX;
            y = event.clientY;
        };

        $effect(() => {
            window.addEventListener("mousemove", updateMousePosition);

            return () => {
                window.removeEventListener("mousemove", updateMousePosition);
            };
        });

        return {
            get x() {
                return x;
            },
            get y() {
                return y;
            },
        };
    };

    let position = useMousePosition();
</script>

{#await preload() then _}
    <svg class="w-full h-full">
        <image
            href={cursorImgSrc}
            x={position.x - 18}
            y={position.y - 18}
            height={36}
        />
    </svg>
{/await}

<style>
    :global(body) {
        cursor: none;
    }

    .w-full {
        width: 100vw;
    }

    .h-full {
        height: 100vh;
    }

    svg {
        position: fixed;
        top: 0;
        left: 0;
        pointer-events: none;
        mix-blend-mode: exclusion;
    }
</style>
