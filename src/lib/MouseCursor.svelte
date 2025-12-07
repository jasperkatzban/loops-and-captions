<script lang="ts">
    import { onMount } from "svelte";

    let cursorImgSrc = $state("cursor/0.svg");
    let cursorState = $state("normal");
    $inspect(cursorState);

    let c = 0;

    onMount(() => {
        document.addEventListener("mouseover", (event) => {
            const target = event.target;
            // Check if the hovered element is a link or refresh button
            if (
                target.tagName === "A" ||
                target.id === "refresh-button-visible-element"
            ) {
                cursorState = "hover";
            } else {
                cursorState = "normal";
            }
        });

        const interval = setInterval(() => {
            if (cursorState == "normal") {
                let i = c % 19;
                cursorImgSrc = "cursor/" + i + ".svg";
                c++;
            } else {
                cursorImgSrc = "cursor/hover.svg";
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

<svg class="w-full h-full">
    <image
        href={cursorImgSrc}
        x={position.x - 18}
        y={position.y - 18}
        height={36}
    />
</svg>

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
