<script lang="ts">
    import favicon from "$lib/assets/cursor.svg";

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
    <image href={favicon} x={position.x - 20} y={position.y - 20} height={40} />
</svg>

<style>
    :global(body) {
        cursor: none;
    }

    .w-full {
        width: 100vw;
    }

    .h-full {
        height: 100vw;
    }

    svg {
        position: fixed;
        top: 0;
        left: 0;
        pointer-events: none;
    }
</style>
