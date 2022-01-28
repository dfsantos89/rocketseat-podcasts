<svelte:head>
    <title>{episode.title} | Podcasts</title>
</svelte:head>

<style lang="scss">
    .episode {
        padding: 3rem 2rem;
        margin: 0 auto;

        // Coloca a barra de rolagem no conteúdo do post
        height: calc(100vh - 6.5rem);
        overflow-y: auto;

        // Centraliza o conteúdo do post
        > * {
            max-width: 45rem;
            margin: 0 auto;
        }

        .thumbnail-container {
            position: relative;

            img {
                border-radius: 1px;
                object-fit: cover;
            }

            button {
                width: 3rem;
                height: 3rem;
                border-radius: 0.75rem;
                border: 0;
                position: absolute;
                z-index: 5;
                font-size: 0;

                transition: filter 0.2s;

                &:hover {
                    filter: brightness(0.9);
                }
            }

            a button {
                left: 0;
                top: 50%;
                background: var(--purple-500);
                transform: translate(-50%, -50%);
            }

            > button {
                right:0;
                top: 50%;
                background: var(--green-500);
                transform: translate(50%, -50%);
            }
        }

        header {
            padding-bottom: 1rem;
            border-bottom: 1px solid var(--gray-100);

            h1 {
                margin-top: 2rem;
                margin-bottom: 1.5rem;
            }

            span {
                display: inline-block;
                font-size: 0.875rem;

                & + span {
                    margin-left: 1rem;
                    padding-left: 1rem;
                    position: relative;

                    &::before {
                        content: '';
                        width: 4px;
                        height: 4px;
                        border-radius: 2px;
                        background: #DDD;
                        position: absolute;
                        left: 0;
                        top: 50%;
                        transform: translate(-50%, -50%);
                    }
                }
            }
        }

        .description {
            margin-top: 2rem;
            line-height: 1.675rem;
            color: var(--gray-800);

            :global(p) {
                margin: 1.5rem 0;
            }
        }
    }
</style>

<script lang="ts">
    import { usePlayer } from '../../contexts/player.svelte';

    type Episode = {
        title: string;
        thumbnail: string;
        members: string;
        publishedAt: string;
        duration: string;
        description: string;
    };

    export let episode :Episode;
    
    const { play } = usePlayer();
</script>

<script context="module" lang="ts">
    import { format, parseISO } from 'date-fns';
    import ptBR from 'date-fns/locale/pt-BR';

    import { api } from '../../services/api';
    import { convertDurationToTimeString } from '../../utils/convertDurationToTimeString';

    export async function load ({ params }) {
        const { data } = await api.get(`/episodes/${params.slug}`);

        const episode = {
            title: data.title,
            thumbnail: data.thumbnail,
            members: data.members,
            publishedAt: format(parseISO(data.published_at), 'd MMM yy', {
                locale: ptBR
            }),
            duration: convertDurationToTimeString(Number(data.file.duration)),
            description: data.description,
            file: {
                url: data.file.url,
                duration: data.file.duration
            }
        };

        return {
            props: { episode }
        };
    }
</script>

<div class="episode">
    <div class="thumbnail-container">
        <a href="/">
            <button type="button">
                <img src="/arrow-left.svg" alt="Voltar">
            </button>
        </a>
        <img width="{700}" height="{160}" src="{episode.thumbnail}" alt="{episode.title}">
        <button type="button" on:click="{() => play([episode], 0)}">
            <img src="/play.svg" alt="Trocar episódio">
        </button>
    </div>

    <header>
        <h1>{episode.title}</h1>
        <span>{episode.members}</span>
        <span>{episode.publishedAt}</span>
        <span>{episode.duration}</span>
    </header>

    <div class="description">
        {@html episode.description}
    </div>
</div>