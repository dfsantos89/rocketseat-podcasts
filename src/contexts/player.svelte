<script context="module" lang="ts">
    import { getContext } from 'svelte';

    type PlayerContextData = {
        playlist;
        current;
        isPlaying;
        isLooping;
        isShuffling;
        play(episodes : any[], i: number) : void;
        next(): void;
        previous(): void;
        hasNext(): boolean;
        hasPrevious() :boolean;
        clear(): void;
    };

    const key = {};

    export function usePlayer(): PlayerContextData {
        return getContext(key);
    }
</script>

<script lang="ts">
    import { setContext } from 'svelte';
    import { writable } from 'svelte/store';

    const current = writable(0);
    const playlist = writable([]);
    const isPlaying = writable(false);
    const isLooping = writable(false);
    const isShuffling = writable(false);

    $: hasNext = $current + 1 < $playlist.length;
    $: hasPrevious = $current > 0;

    setContext<PlayerContextData>(key, {
        playlist,
        current,
        isPlaying,
        isLooping,
        isShuffling,
        play: (episodes, i: number) => {
            $playlist = episodes;
            $current = i;
            $isPlaying = true;
        },
        next: () => {
            if ($isShuffling) {
                const next = ~~(Math.random() * $playlist.length);
                $current = next;
            } else if (hasNext) {
                $current += 1;
            }
        },
        previous: () => {
            if (hasPrevious) {
                $current -= 1;
            }
        },
        hasNext: () => hasNext,
        hasPrevious: () => hasPrevious,
        clear: () => {
            $playlist = [];
            $current = 0;
        }
    });

</script>

<slot></slot>