<style lang="scss">
    header {
        background: var(--white);
        height: 6.5rem;
    
        display: flex;
        align-items: center;
    
        padding: 2rem 4rem;
    
        border-bottom: 1px solid var(--gray-100);
    
        p {
            margin-left: 2rem;
            padding: 0.25rem 0 0.25rem 2rem;
            border-left: 1px solid var(--gray-100);
        }
    
        span {
            margin-left: auto;
            text-transform: capitalize;
        }
    }
</style>

<script lang="ts">
import format from 'date-fns/format';
import ptBR from 'date-fns/locale/pt-BR';

const currentData = format(new Date(), 'EEEEEE, d MMMM', {
    locale: ptBR
});
</script>

<header>
    <img src="/logo.svg" alt="Podcasts">

    <p>O melhor para você ouvir, sempre</p>

    <span>{currentData}</span>
</header>