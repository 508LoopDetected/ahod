<script>
    import { onMount, createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();
    export let timeLeft;

    let backgrounds = [
        '/ahod/background-blitz/office.gif',
        '/ahod/background-blitz/midcen.gif',
        //'/ahod/background-blitz/concert.gif',
        '/ahod/background-blitz/matrix.gif',
        '/ahod/background-blitz/lab.gif',
        '/ahod/background-blitz/headroom.gif',
        '/ahod/background-blitz/falcon.gif',
        '/ahod/background-blitz/closet.gif',
        '/ahod/background-blitz/outside.gif',
        '/ahod/background-blitz/giygas.gif',
        '/ahod/background-blitz/space.gif',
        //'/ahod/background-blitz/graveyard.png'
    ];

    // Logic
    let currentBackgroundIndex = 0;
    let correctBackgroundIndex = 0;
    let interval;
    let totalWins = 0;
    let message = '';
    let revealedBackgrounds = [false, false, false];
    let revealClass = [false, false, false]; // Track reveal class state for each .npc
    let shuffling = [true, true, true];
    let shuffleIntervals = [null, null, null];

    // Audio
    let gameStarted = false;
    let soundNPC;
    let soundWin;
    let soundLose;
    let soundSelect;

    onMount(() => {
        startGame();
    });

    function startGame() {
        gameStarted = true;
        soundNPC = new Audio('/ahod/audio/npc.mp3');
        soundWin = new Audio('/ahod/audio/win.wav');
        soundLose = new Audio('/ahod/audio/lose.wav');
        soundSelect = new Audio('/ahod/audio/select.wav');
        
        correctBackgroundIndex = Math.floor(Math.random() * backgrounds.length);
        revealedBackgrounds = [0, 0, 0];
        revealClass = [false, false, false];
        shuffling = [true, true, true];

        revealRandomBackground();
    }

    function revealRandomBackground() {
        let revealDurations = [900, 2200, 3400]; // Durations for how long each tile shuffles

        revealDurations.forEach((duration, index) => {
            // Clear previous interval if exists
            if (shuffleIntervals[index]) clearInterval(shuffleIntervals[index]);

            shuffleIntervals[index] = setInterval(() => {
                // Randomly change the background while shuffling
                revealedBackgrounds[index] = Math.floor(Math.random() * backgrounds.length);
            }, 100); // Change the background every 100ms

            setTimeout(() => {
                clearInterval(shuffleIntervals[index]);
                revealedBackgrounds[index] = correctBackgroundIndex;
                shuffling[index] = false;
                revealClass[index] = true; // Add the reveal class
                if (soundNPC) soundNPC.play();
            }, duration);
        });
    }

    function checkSelection() {
        if (currentBackgroundIndex === correctBackgroundIndex) {
            if (soundWin) soundWin.play();
            dispatch('win');
        } else {
            if (soundLose) soundLose.play();
            dispatch('lose');
        }
    }

    function changeBackground(direction) {
        if (soundSelect) soundSelect.play();
        if (direction === 'left') {
            currentBackgroundIndex = (currentBackgroundIndex - 1 + backgrounds.length) % backgrounds.length;
        } else {
            currentBackgroundIndex = (currentBackgroundIndex + 1) % backgrounds.length;
        }
    }

    $: backgroundStyle = `background-image: url(${backgrounds[currentBackgroundIndex]});`;

    $: if (timeLeft <= 0) {
        checkSelection();
    }
</script>


{#if gameStarted}
  <div id="video-call-grid">
      {#each revealedBackgrounds as background, index}
          <div class="video-tile" style={`background-image: url(${shuffling[index] ? backgrounds[background] : backgrounds[correctBackgroundIndex]});`}>
              <div class={`npc ${revealClass[index] ? 'reveal' : ''}`}></div>
          </div>
      {/each}
      <div id="playerTile" style={backgroundStyle} class="video-tile">
        <div id="controls">
          <button on:click={() => changeBackground('left')}>Left</button>
          <button on:click={() => changeBackground('right')}>Right</button>
        </div>
      </div>
  </div>
{/if}


<style>
#video-call-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    grid-gap: 15px;
    position: relative;
    overflow: hidden;
    height: 100%;
}

.video-tile {
    background-color: #000;
    background-size: cover;
    background-position: center;
    position: relative;
    overflow: hidden;
}

.video-tile .npc {
    background-image: url('/ahod/background-blitz/larry.png');
    background-size: contain;
    background-position: center bottom;
    background-repeat: no-repeat;
    width: 80%;
    height: 100%;
    position: absolute;
    bottom: -15%;
    left: 0;
    right: 0;
    margin: auto;
    filter: brightness(0);
}
.video-tile .npc.reveal {
    filter: unset;
}

#playerTile {
    position: relative; /* For positioning background images */
    background-color: #ccc; /* Placeholder color */
    background-size: cover;
    background-position: center center;
}

#controls {
    bottom: 0;
    max-width: 300px;
    margin: 0 auto;
    position: absolute;
    left: 0;
    right: 0;
    width: 100%;
}

button {
    padding: 10px 20px;
    border: none;
    background-color: #007bff;
    color: white;
    cursor: pointer;
    transition: background-color 0.3s;
}

button:hover {
    background-color: #0056b3;
}
</style>