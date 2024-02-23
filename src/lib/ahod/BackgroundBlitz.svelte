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
        // '/ahod/background-blitz/space.gif',
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
          <div class={`video-tile ${revealClass[index] ? 'reveal' : ''}`} style={`background-image: url(${shuffling[index] ? backgrounds[background] : backgrounds[correctBackgroundIndex]});`}>
            <div class="npc"></div>
            <div class="joining">Joining...</div>
          </div>
      {/each}
      <div id="playerTile" style={backgroundStyle} class="video-tile">
        <div id="controls">
          <button on:click={() => changeBackground('left')}>&lsaquo;</button>
          <button on:click={() => changeBackground('right')}>&rsaquo;</button>
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
.video-tile {
    filter: grayscale(100%);
}

.video-tile .npc {
    background-image: url('/ahod/larry.png');
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
.video-tile .joining {
    text-align: center;
    font-size: 2.5rem;
    color: #fff;
    position: absolute;
    top: 40%;
    width: 100%;
}

#playerTile,
.video-tile.reveal,
.video-tile.reveal .npc {
    filter: unset;
}
.video-tile.reveal .joining {
    display: none;
}

#playerTile {
    position: relative;
    background-color: #ccc;
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
    background-color: #ff0030;
    border: 1px solid #000;
    border-radius: 5px;
    box-shadow: inset 1px 1px 0 rgba(255,255,255,0.4), 5px 5px 0 rgba(0,0,0,0.1);
    text-shadow: -1px -1px 0 rgba(0,0,0,0.5);
    color: #fff;
    margin: 0 5px 15px;
    padding: 5px 20px;
    cursor: pointer;
    transition: background-color 0.3s;
    font-weight: normal;
    text-transform: uppercase;
    font-family: 'CooperBits Medium';
    font-size: 2rem;
    display: inline-block;
}

button:hover {
    background-color: #ff0030;
}
</style>