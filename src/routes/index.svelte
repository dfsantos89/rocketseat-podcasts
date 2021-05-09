<svelte:head>
    <title>Home | Podcasts</title>
</svelte:head>

<style lang="scss">
    .homepage {
        padding: 0 4rem;
        height: calc(100vh - 6.5rem);
        overflow-y: scroll;

        h2 {
            margin-top: 3rem;
            margin-bottom: 1.5rem;
        }
    }

    .latest-episodes {
        ul {
            list-style: none;
            display: grid;

            grid-template-columns: repeat(2, 1fr);
            gap: 1.5rem;

            @media (max-width: 1480px) {
                grid-template-columns: 1fr;
            }

            li {
                background: var(--white);
                border: 1px solid var(--gray-100);
                padding: 1.25rem;
                border-radius: 1.5rem;
                position: relative;

                display: flex;
                align-items: center;

                img {
                    width: 6rem;
                    height: 6rem;
                    border-radius: 1rem;
                    object-fit: cover;
                }

                .episode-details {
                    flex: 1;
                    margin-left: 1rem;

                    // Calcula o tamanho máximo do episode details
                    // para que a imagem fique com um width de 6rem
                    // max-width: calc(100% - 7rem);

                    a {
                        display: block;
                        color: var(--gray-800);
                        font-family: Lexend, sans-serif;
                        font-weight: 600;
                        text-decoration: none;
                        line-height: 1.4rem;

                        &:hover {
                            text-decoration: underline;
                        }
                    }

                    p {
                        font-size: 0.875rem;
                        margin-top: 0.5rem;
                        max-width: 70%;
                        white-space: nowrap;
                        overflow: hidden;
                        text-overflow: ellipsis;
                    }

                    span {
                        display: inline-block;
                        margin-top: 0.5rem;
                        font-size: 0.875rem;

                        &:last-child {
                            margin-left: 0.5rem;
                            padding-left: 0.5rem;
                            position: relative;
                        }

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

                button {
                    position: absolute;
                    right: 2rem;
                    bottom: 2rem;
                    
                    width: 2.5rem;
                    height: 2.5rem;
                    background: var(--white);
                    border: 1px solid var(--gray-100);
                    border-radius: 0.675rem;
                    font-size: 0;

                    transition: filter 0.2s;

                    img {
                        width: 1.5rem;
                        height: 1.5rem;
                    }

                    &:hover {
                        filter: brightness(0.95);
                    }
                }
            }
        }
    }

    .all-episodes {
        padding-bottom: 2rem;

        table {
            width: 100%;

            th, td {
                padding: 0.75rem 1rem;
                border-bottom: 1px solid var(--gray-100);
            }

            th {
                color: var(--gray-200);
                text-transform: uppercase;
                font: 500 0.75rem Lexend, sans-serif;
                text-align: left;
            }

            td {
                font-size: 0.875rem;

                &:first-child {
                    width: 4.5rem;
                }

                &:nth-child(4) {
                    width: 6.25rem;
                }

                img {
                    width: 2.5rem;
                    height: 2.5rem;
                    border-radius: 0.5rem;
                    object-fit: cover;
                }

                button {
                    width: 2rem;
                    height: 2rem;
                    background: var(--white);
                    border: 1px solid var(--gray-100);
                    border-radius: 0.5rem;
                    font-size: 0;

                    transition: filter 0.2s;

                    img {
                        width: 1.25rem;
                        height: 1.25rem;
                    }

                    &:hover {
                        filter: brightness(0.95);
                    }
                }

                a {
                    color: var(--gray-800);
                    font-family: Lexend, sans-serif;
                    font-weight: 600;
                    text-decoration: none;
                    line-height: 1.4rem;
                    font-size: 1rem;

                    &:hover {
                        text-decoration: underline;
                    }
                }
            }

            // Responsive suporte pra table
            td, th {
                @media (max-width: 1480px) {
                    &:nth-child(4) {
                        display: none;
                    }
                }

                @media (max-width: 1340px) {
                    &:nth-child(3) {
                        display: none;
                    }
                }

                @media (max-width: 1180px) {
                    &:nth-child(5) {
                        display: none;
                    }
                }
            }
        }
    }
</style>

<script lang="ts">
    import { usePlayer } from '../contexts/player.svelte';

    type Episode = {
        id: string;
        title: string;
        members: string;
        thumbnail: string;
        publishedAt: string;
        duration: string;
    };

    export let episodes :Episode[];

    const { play } = usePlayer();
</script>

<script context="module" lang="ts">
    import { format, parseISO } from 'date-fns';
    import ptBR from 'date-fns/locale/pt-BR';

    import { api } from "../services/api";
    import { convertDurationToTimeString } from '../utils/convertDurationToTimeString';
    
    export async function load() {
        const { data } = await api.get('episodes', {
            params: {
                _limit: 12,
                _sort: 'published_at',
                _order: 'desc'
            }
        });

        const episodes = data.map(episode => ({
            id: episode.id,
            title: episode.title,
            members: episode.members,
            thumbnail: episode.thumbnail,
            duration: convertDurationToTimeString(episode.file.duration),
            publishedAt: format(parseISO(episode.published_at), 'd MMM yy', {
                locale: ptBR
            }),
            file: {
                duration: episode.file.duration,
                url: episode.file.url
            }
        }));

        return {
            props: {
                episodes
            }
        };
    }
</script>

<div class="homepage">
    <section class="latest-episodes">
        <h2>Últimos lançamentos</h2>
    
        <ul>
            {#each episodes.slice(0, 2) as episode, i (episode.id)}
                <li>
                    <img src="{episode.thumbnail}" width="{192}" height="{192}" alt="{episode.title}"/>

                    <div class="episode-details">
                        <a href="/episodes/{episode.id}">{episode.title}</a>
                        <p>{episode.members}</p>
                        <span>{episode.publishedAt}</span>
                        <span>{episode.duration}</span>
                    </div>

                    <button type="button" on:click="{() => play(episodes, i)}">
                        <img src="/play-green.svg" alt="Trocar episódio">
                    </button>
                </li>
            {/each}
        </ul>
    </section>
    <section class="all-episodes">
        <h2>Todos episódios</h2>
    
        <table>
            <thead>
                <tr>
                    <th></th>
                    <th>Podcast</th>
                    <th>Integrantes</th>
                    <th>Data</th>
                    <th>Duração</th>
                    <th></th>
                </tr>
            </thead>
            <tbody>
                {#each episodes.slice(2) as episode, i (episode.id)}
                    <tr>
                        <td>
                            <img src="{episode.thumbnail}" width="{120}" height="{120}" alt="{episode.title}"/>
                        </td>
                        <td>
                            <a href="/episodes/{episode.id}">{episode.title}</a>
                        </td>
                        <td>{episode.members}</td>
                        <td>{episode.publishedAt}</td>
                        <td>{episode.duration}</td>
                        <td>
                            <button type="button">
                                <img src="/play-green.svg" alt="Tocar episódio" on:click="{() => play(episodes, i + 2)}">
                            </button>
                        </td>
                    </tr>
                {/each}
            </tbody>
        </table>
    </section>
</div>
