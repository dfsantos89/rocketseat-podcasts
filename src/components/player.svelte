<style lang="scss">
.container {
    padding: 3rem 4rem;
    width: 26.5rem;
    height: 100vh;

    background: var(--purple-500);
    color: var(--white);

    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: space-between;

    header {
        display: flex;
        align-items: center;
        gap: 1rem;
    }

    strong {
        font-family: Lexend, sans-serif;
        font-weight: 600;
    }

    footer {
        align-self: stretch;

        &.empty {
            opacity: 0.5;
        }
    }
}

.current-episode {
    text-align: center;

    img {
        width: 18.5rem;
        height: 18.5rem;
        border-radius: 1.5rem;
        object-fit: cover;
    }

    strong {
        display: block;
        margin-top: 2rem;
        font: 600 1.25rem Lexend, sans-serif;
        line-height: 1.75rem;
    }

    span {
        display: block;
        margin-top: 1rem;
        opacity: 0.6;
        line-height: 1.5rem;
    }
}

.empty-player {
    width: 100%;
    height: 20rem;
    border: 1.5px dashed var(--purple-300);
    border-radius: 1.5rem;
    background: linear-gradient(143.8deg, rgba(145, 100, 250, 0.8) 0%, rgba(0, 0, 0, 0) 100%);

    padding: 4rem;
    text-align: center;

    display: flex;
    align-items: center;
    justify-content: center;
}

.progress {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    font-size: 0.875rem;

    span {
        display: inline-block;
        min-width: 4rem;
        text-align: center;
    }

    .slider {
        flex: 1;

        .empty-slider {
            width: 100%;
            height: 4px;
            background: var(--purple-300);
            border-radius: 2px;
        }
    }
}

.buttons {
    display: flex;
    align-items: center;
    justify-content: center;
    margin-top: 2.5rem;
    gap: 1.5rem;

    button {
        background: transparent;
        border: 0;
        font-size: 0;

        transition: filter 0.2s;

        &:disabled {
            cursor: default;
        }

        &:hover:not(:disabled) {
            filter: brightness(0.8);
        }

        &.active {
            filter: invert(0.35) sepia(1) saturate(3) hue-rotate(100deg);
        }

        &.play {
            width: 4rem;
            height: 4rem;
            border-radius: 1rem;
            background: var(--purple-400);

            &:hover:not(:disabled)  {
                filter: brightness(0.95);
            }
        }
    }
}
</style>

<script lang="ts">
    import { usePlayer } from '../contexts/player.svelte';
    import { convertDurationToTimeString } from '../utils/convertDurationToTimeString';

    type AudioFile = {
        url: string;
        duration: number;
    };

    type Episode = {
        title: string;
        thumbnail: string;
        members: string;
        file: AudioFile;
        duration: string;
    };

    let audio;
    let progress = 0;
    let ended;

    const { playlist, current, isPlaying, isLooping, isShuffling, next, previous, hasNext, hasPrevious, clear } = usePlayer();

    $: if (ended) hasNext() ? next() : clear();

    $: episode = $playlist[$current] as Episode;
    
    $: audio && ($isPlaying ? audio.play() : audio.pause());
</script>

<div class="container">
    <header>
        <img src="/playing.svg" alt="Tocando agora">
        <strong>Tocando agora</strong>
    </header>

    {#if episode }
        <div class="current-episode">
            <img width="{592}" height="{592}" src="{episode.thumbnail}" alt="{episode.title}"/>
            <strong>{episode.title}</strong>
            <span>{episode.members}</span>
        </div>
    {:else}
        <div class="empty-player">
            <strong>Selecione um podcast para ouvir</strong>
        </div>
    {/if}

    <footer class:empty="{!episode}">
        <div class="progress">
            <span>{convertDurationToTimeString(progress)}</span>
            <div class="slider">
                {#if episode}
                    <input type="range" bind:value="{progress}" max="{episode.file.duration}"/>
                {:else}
                    <div class="empty-slider"/>
                {/if}
            </div>
            <span>{episode?.duration ?? '00:00'}</span>
        </div>

        {#if episode }
            <audio 
                src="{episode.file.url}" 
                autoplay 
                loop="{$isLooping}" 
                bind:this="{audio}" 
                bind:currentTime="{progress}" 
                bind:ended="{ended}"
                on:play="{$isPlaying = true}" 
                on:pause="{$isPlaying = false}"
            >
                <track kind="captions">
            </audio>
        {/if}

        <div class="buttons">
            <button type="button" disabled="{!episode}" on:click="{() => $isShuffling = !$isShuffling}" class:active="{$isShuffling}">
                <img src="/shuffle.svg" alt="Embaralhar">
            </button>
            <button type="button" disabled="{!episode || !hasPrevious()}" on:click="{previous}">
                <img src="/play-previous.svg" alt="Tocar anterior">
            </button>
            <button 
                type="button" 
                class="play" 
                disabled="{!episode}" 
                on:click="{() => $isPlaying = !$isPlaying}" 
            >
                {#if $isPlaying }
                    <img src="/pause.svg" alt="Pausar">
                {:else}
                    <img src="/play.svg" alt="Tocar">
                {/if}
            </button>
            <button type="button" disabled="{!episode || !hasNext()}" on:click="{next}">
                <img src="/play-next.svg" alt="Tocar próxima">
            </button>
            <button type="button" disabled="{!episode}" class:active="{$isLooping}" on:click="{() => $isLooping = !$isLooping}">
                <img src="/repeat.svg" alt="Repetir">
            </button>
        </div>
    </footer>
</div>