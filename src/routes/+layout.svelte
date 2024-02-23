<script>
  import AHOD from '$lib/components/AHOD.svelte';

  let showAHOD = false;
  function handleExit() {
    showAHOD = false;
  }
</script>

<main>
  {#if showAHOD}
    <AHOD on:exit={handleExit} />
  {:else}
    <button class="ahod" on:click={() => showAHOD = true}>All Hands on Deck!</button>
  {/if}

  <slot />

</main>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Azeret+Mono:wght@300&display=swap');
  @font-face {
    font-family: 'CooperBits Medium';
    font-style: normal;
    font-weight: normal;
    src: local('CooperBits Medium'), url('/fonts/CooperBits.woff') format('woff');
  }
  :global(*) {
    box-sizing: border-box;
  }
  :global(body) {
    background-color: #72998d;
    /*background-image: url(/img/bg/dolphins-sm.png);*/
    background-image: url(/img/bg/bg.jpg);
    background-size: cover;
    background-repeat: no-repeat;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 0;
    font-family: 'Azeret Mono', monospace;
    height: 100vh;
    width: 100vw;
  }
  :global(h1) {
    letter-spacing: 2px;
  }

  :global(header) {
    text-align: center;
  }
  .ahod {
    display: block;
    margin: 20px auto 30px;
    background-color: #000;
    border-radius: 5px;
    padding: 25px 40px 22px;
    font-family: 'CooperBits Medium', serif;
    letter-spacing: 2px;
    color: #fff;
    border: none;
    font-size: 3rem;
    cursor: pointer;
    font-weight: normal;
  }
  .ahod:hover {
    background-color: #333;
  }
</style>