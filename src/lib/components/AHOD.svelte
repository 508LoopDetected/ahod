<script>
    import { onMount, onDestroy, createEventDispatcher } from 'svelte';
    import BackgroundBlitz from '$lib/ahod/BackgroundBlitz.svelte';
    import UploadPanic from '$lib/ahod/UploadPanic.svelte';
    import ChatClicker from '$lib/ahod/ChatClicker.svelte';

    const dispatch = createEventDispatcher();
    const MAX_LOSSES = 3;
    $: playerHealth = Array(3).fill(false).fill(true, totalLosses);

    let currentGameIndex = 0;
    let inGame = false;
    let showStartScreen = true;
    let showEndScreen = false;
    let totalWins = 0;
    let totalLosses = 0;
    let soundMenu;
    let soundHuzzah;
    let soundLevel;
    let soundWinner;
    let soundLose;
    let timeLeft = 5; // e.g. 5 sec per microgame, but maybe not always
    let totalTime;
    let timerInterval;
    let showIntroScreen = false;
    let lastGameResult = null; // null indicates no previous game, 'win' or 'lose' will be set otherwise
    let introClass = '';
    let currentInstruction = '';
    let currentInstructionDetails = '';    
    let shuffledGamePairs = [];
    let gamePairs = [
        { game: BackgroundBlitz, instruction: 'Conform!', details: 'Match your <strong>BACKDROP</strong> before the meeting starts!', timeLimit: 4.5 },
        { game: UploadPanic, instruction: 'Upload!', details: 'Deliver your <strong>FILES</strong> before the deadline!', timeLimit: 4.5 },
        { game: ChatClicker, instruction: 'Respond!', details: 'Send a <strong>REBUTTAL</strong> before they gaslight you!', timeLimit: 4.5 }
    ];

    $: timeLeftPercentage = (timeLeft / totalTime) * 100;

    onMount(() => {
        soundMenu = new Audio('ahod/audio/AHOD Menu [Shh].ogg');
        soundMenu.loop = true;
        soundMenu.preload = 'auto';
        if (soundMenu) soundMenu.play();
        // buffer loop because browser-based sound design is shite
        soundMenu.addEventListener('timeupdate', function(){
            let buffer = .22;
            if (this.currentTime > this.duration - buffer){
                this.currentTime = 0;
                this.play();
            }
        });
    });

    function startTimer() {
        inGame = true;
        clearInterval(timerInterval);
        totalTime = shuffledGamePairs[currentGameIndex].timeLimit; // Set total time to the current game's limit
        timeLeft = totalTime; // Reset timeLeft to the total time at the start of the game

        timerInterval = setInterval(() => {
            if (timeLeft > 0) {
                timeLeft -= 0.1;
            } else {
                clearInterval(timerInterval);
                if (inGame) {
                    recordLoss();
                }
            }
        }, 100);
    }

    function startGameSequence() {
        showStartScreen = false;
        showEndScreen = false;
        totalWins = 0;
        totalLosses = 0;
        shuffledGamePairs = createShuffledGameLoop(gamePairs, 10);
        currentGameIndex = 0;
        lastGameResult = null;
        if (soundWinner) {
            soundWinner.pause();
            soundWinner.currentTime = 0;
        }
        if (soundMenu) {
            soundMenu.pause();
            soundMenu.currentTime = 0;
        }
        soundLevel = new Audio('ahod/audio/AHOD Long.ogg');
        soundLevel.loop = true;
        soundLevel.preload = 'auto';
        nextGame();
    }

    function shuffleGames(array) {
        let shuffledArray = [...array];
        for (let i = shuffledArray.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [shuffledArray[i], shuffledArray[j]] = [shuffledArray[j], shuffledArray[i]];
        }
        return shuffledArray;
    }

    function createShuffledGameLoop(pairs, loopLength) {
        let extendedPairs = [];
        while (extendedPairs.length < loopLength) {
            extendedPairs = extendedPairs.concat(shuffleGames([...pairs]));
        }
        return extendedPairs.slice(0, loopLength);
    }

    function nextGame() {
        if (totalLosses >= MAX_LOSSES || currentGameIndex >= shuffledGamePairs.length) {
            endGameSequence();
            return; // Exit early if the game should end
        }
        showIntroScreen = true;
        currentInstruction = shuffledGamePairs[currentGameIndex].instruction;
        currentInstructionDetails = shuffledGamePairs[currentGameIndex].details;
        if (currentGameIndex == 0) {
            soundHuzzah = new Audio('ahod/audio/AHOD Next.ogg');
            if (soundHuzzah) soundHuzzah.play();
        }
        introClass = lastGameResult === 'lose' ? 'loser' : (lastGameResult === 'win' ? 'winner' : '');
        setTimeout(() => {
            // just for the lady's win/lose state
            introClass = '';
        }, 1000);
        setTimeout(() => {
            showIntroScreen = false;
            console.log("Intro screen should be hidden now");
            setTimeout(() => {
                console.log("Actually starting game now");
                startTimer(); // Now this also handles setting `inGame = true`.
            }, 0); // Slight delay to ensure intro screen has time to hide
            if (soundLevel) soundLevel.play();
        }, 2000);
    }

    function endGame() {
        inGame = false;
        currentGameIndex++;
        nextGame();
    }

    function recordWin() {
        lastGameResult = 'win';
        totalWins++;
        endGame();
    }

    function recordLoss() {
        lastGameResult = 'lose';
        console.log("Inside recordLoss, inGame:", inGame, "totalLosses:", totalLosses);
        if (totalLosses < MAX_LOSSES) {
            totalLosses++;
            if (totalLosses >= MAX_LOSSES) {
                endGameSequence();
            } else {
                endGame();
            }
        }
    }

    function endGameSequence() {
        if (soundLevel) {
            soundLevel.pause();
            soundLevel.currentTime = 0;
        }
        if (totalLosses == 3) {
            soundLose = new Audio('ahod/audio/AHOD Lose.ogg');
            if (soundLose) soundLose.play();
        } else {
            soundWinner = new Audio('ahod/audio/AHOD Boss.ogg');
            soundWinner.preload = 'auto';
            soundHuzzah = new Audio('ahod/audio/AHOD Next 2.ogg');
            if (soundHuzzah) soundHuzzah.play();
            setTimeout(() => {
                if (soundWinner) soundWinner.play();
            }, 2000);            
        }
        showEndScreen = true;
    }

    function resetToStartMenu() {
        if (soundWinner) {
            soundWinner.pause();
            soundWinner.currentTime = 0;
        }
        if (soundMenu) {
            soundMenu.currentTime = 0;
            soundMenu.play();
        }
        // buffer loop because browser-based sound design is shite
        soundMenu.addEventListener('timeupdate', function(){
            let buffer = .22;
            if (this.currentTime > this.duration - buffer){
                this.currentTime = 0;
                this.play();
            }
        });
        showStartScreen = true;
        showEndScreen = false;
        totalWins = 0;
        totalLosses = 0;
        currentGameIndex = 0;
    }

    function exitGame() {
        dispatch('exit');
    }

    onDestroy(() => {
        clearInterval(timerInterval);
        if (soundMenu) {
            soundMenu.pause();
            soundMenu.currentTime = 0;
        }
        if (soundWinner) {
            soundWinner.pause();
            soundWinner.currentTime = 0;
        }
    });

</script>

<div id="ahod">
    {#if showIntroScreen}
                <div id="intro-screen" class={introClass}>
                    {#if lastGameResult}
                        <div class="result-message"></div>
                    {/if}
                    <div class="instbg"></div>
                    <div class="insttxt">NEXT UP:<br/><span>{currentInstruction}</span></div>
                    <div class="lady"></div>
                </div>
            {/if}
{#if showStartScreen}
    <div class="start-screen">
        <button on:click={startGameSequence}>Start</button> <button on:click={exitGame}>Exit</button>
    </div>
{:else if showEndScreen}
    <div id="end-screen" class={totalLosses === 3 ? 'loser' : 'winner'}>
        <div class="options">
            <!-- Wins: {totalWins}
            Losses: {totalLosses} -->
            <h3>YOU {totalLosses === 3 ? 'ARE DISGRACED' : 'STILL HAVE A JOB'}</h3>
            <button on:click={startGameSequence}>Retry</button><br/>
            <button on:click={resetToStartMenu}>Menu</button>
        </div>
        <div class="hero"></div>
        <div class="filter"></div>
    </div>
{:else}
    <div class="game-screen">
        <div class="timer">
            <div class="timer-bar-container">
                <div class="timer-bar" style="height: {timeLeftPercentage}%;">
                    <!-- <span>{timeLeft.toFixed(1)}</span> -->
                </div>
            </div>
        </div>
        <div class="microgame">
            {#if !showIntroScreen && shuffledGamePairs && currentGameIndex < shuffledGamePairs.length}
                <svelte:component this={shuffledGamePairs[currentGameIndex].game}
                    on:win={recordWin} 
                    on:lose={recordLoss}
                    timeLeft={timeLeft} />
            {/if}
        </div>
        <div class="hud">
            <!-- <div class="wins-losses">
                Wins: {totalWins}<br/>
                Losses: {totalLosses}
            </div> -->
            <div class="player-health">
                {#each playerHealth as heart}
                    <span class="heart"><img src="/ahod/{heart ? 'pending' : 'late'}.png" /></span>
                {/each}
            </div>
            <div class="details">
                <div class="details-text">{@html currentInstructionDetails}</div>
                <div class="lady"></div>
            </div>
        </div>
    </div>
{/if}
</div>

<style>
    #ahod {
        width: 1000px;
        height: 600px;
        border: 3px solid #000;
        border-radius: 10px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
        padding: 0;
        text-align: center;
        margin: 0 auto 50px;
        background-color: #474745;
        position: relative;
        overflow: hidden;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .start-screen {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        display: flex;
        align-items: center;
        flex-direction: column;
        justify-content: center;
        background-image: url('/ahod/start2.png');
        background-size: cover;
        background-position: center center;
        background-repeat: no-repeat;
        background-color: #000;
    }

    #intro-screen {
        clip-path: polygon(0 25%, 100% 0, 100% 90%, 0 75%);
        animation: filterInOut 2s ease-in-out;
        display: flex;
        background-image:url('/img/bg/battle4.gif');
        background-size: cover;
        background-repeat: no-repeat;
        width: calc(100% - 30px);
        height: calc(100% - 30px);
        border: 3px solid #171715;
        border-radius: 10px;
        align-items: center;
        justify-content: center;
        flex-direction: column;
        overflow: hidden;
        opacity: 0;
        z-index: 9;
    }
    #intro-screen .lady {
        position: absolute;
        bottom: 0;
        right: 5%;
        width: 512px;
        height: 512px;
        background-image:url('/ahod/lady.png');
        background-size: contain;
        background-position: bottom center;
        background-repeat: no-repeat;
        animation: slideInOut 2s ease-in-out;
        transform: translateX(-300%);
        z-index: 3;
    }
    #intro-screen.winner .lady {
        background-image:url('/ahod/lady-win.png');
    }
    #intro-screen.loser .lady {
        background-image:url('/ahod/lady-lose.png');
    }

    #intro-screen .instbg {
        clip-path: polygon(0 25%, 100% 0, 100% 70%, 0 75%);
        animation: instructionInOut 2s ease-in-out;
        display: flex;
        align-items: flex-start;
        justify-content: center;
        flex-direction: column;
        width: 100%;
        height: 100%;
        position: relative;
        mix-blend-mode: overlay;
        background-color: crimson;
    }
    #intro-screen .insttxt {
        transform: rotate(-10deg);
        position: absolute;
        z-index: 1;
        color: #fff;
        font-size: 3rem;
        letter-spacing: 1px;
        line-height: 5rem;
        font-style: italic;
        z-index: 3;
        top: 30%;
        left: 5%;
        animation: instructionTextIn 2s ease-in-out;
        font-family: 'CooperBits', serif;
        text-shadow: 12px 7px 0 rgba(0,0,0,0.7);
    }
    #intro-screen .insttxt span {
        font-size: 6rem;
        letter-spacing: 2px;
    }

    #intro-screen .result-message {
        position: absolute;
        bottom: 8%;
        z-index: 5;
        left: 15%;
        width: 320px;
        height: 230px;
        transform: rotate(-10deg);
        animation: blinkInOut 2s linear;
        opacity: 0;
        background-image: none;
        background-size: cover;
        background-repeat: no-repeat;
        background-position: bottom center;
    }
    #intro-screen.loser .result-message {
        background-image: url('/ahod/lady-status.png');
        background-position: top center;
        height: 160px;
    }
    #intro-screen.winner .result-message {
        background-image: url('/ahod/lady-status.png');
        background-position: bottom center;
        height: 200px;
    }

    .game-screen {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        display: grid;
        grid-template-columns: 0.5fr 6fr 2fr;
        grid-template-rows: 1fr;
        grid-column-gap: 0;
        grid-row-gap: 0; 
    }

    .microgame {
        margin: 15px;
        display: block;
        grid-area: 1 / 2 / 2 / 3;
        background-color: #171715;
        border: 3px solid #171715;
        border-radius: 10px;
        overflow: hidden;
    }

    .hud {
        border: 3px solid #171715;
        border-radius: 10px;
        padding: 20px 20px 0;
        background-color: #575755;
        list-style: none;
        margin: 15px 15px 15px 0;
        grid-area: 1 / 3 / 2 / 4;
        overflow: hidden;
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        background-image: url(/img/bg/bg.jpg);
        background-size: cover;
        background-repeat: no-repeat;
        background-position: center center;
    }

    button {
        display: block;
        margin: 20px auto 0;
        border-radius: 0px;
        padding: 14px 40px 12px;
        font-family: 'CooperBits', serif;
        text-transform: none;
        color: #fff;
        letter-spacing: 2px;
        border: 2px solid #474745;
        border-radius: 5px;
        font-size: 1.3rem;
        cursor: pointer;
        transition: background 1s, transform 0.2s;
        background-color: #000;
        text-shadow: 1px 2px 0 #777775;
    }
    button:hover {
        transform: scale(1.1);
    }

    .timer {
        grid-area: 1 / 1 / 2 / 2;
        margin: 15px 0 15px 15px;
        font-size: 2rem;
        font-weight: bold;
        display: block;
        z-index: 3;
        font-weight: bold;
        color: #000;
        background-color: #f4f4f4;
        border-radius: 10px;
        border: 3px solid #171715;
        overflow: hidden;
    }
    .timer-bar-container {
        width: 100%;
        height: 100%;
        background-color: #171715;
        position: relative;
        background-image: url('/ahod/timerbg.png');
        background-size: cover;
        background-position: bottom center;
        background-repeat: no-repeat;
    }
    .timer-bar {
        width: 100%;
        height: 100%;
        position: absolute;
        bottom: 0;
        left: 0;
        background-color: crimson;
        mix-blend-mode: overlay;
        display: flex;
        justify-content: center;
        align-items: flex-end;
        /*background-image: url('/ahod/timer.png');
        background-size: cover;
        background-position: bottom center;
        background-repeat: no-repeat;*/
    }

    .heart {
        display: block;
        margin: 20px 5px 0;
    }
    .heart img {
        max-width: 50px;
    }
    .details .details-text {
        background: #fff;
        border: 1px solid #000;
        border-radius: 10px;
        padding: 15px 15px 30px;
        margin: 20px auto -30px;
        box-shadow: 5px 5px 0px 0px rgba(0,0,0,0.3);
    }
    .details .lady {
        width: 80%;
        height: 180px;
        margin: 0 auto;
        background-image:url('/ahod/lady.png');
        background-size: cover;
        background-position: top center;
        background-repeat: no-repeat;
    }

    #end-screen {
        width: 100%;
        height: 100%;
        position: relative;
        background-color: #171715;
        background-image: url('/ahod/office.png');
        background-size: cover;
        background-position: center center;
        background-repeat: no-repeat;
    }
    #end-screen.loser {
        background-image: url('/ahod/office-bw.png');
    }

    #end-screen .filter {
        clip-path: polygon(0 25%, 100% 0, 100% 90%, 0 75%);
        width: 100%;
        height: 100%;
        position: absolute;
        top: 0;
        left: 0;
        z-index: 0;
        mix-blend-mode: multiply;
        animation: filterIn 0.5s ease-in-out;
        background-size: cover;
        background-repeat: no-repeat;
    }
    #end-screen.winner .filter {
        background-color: teal;
        background-image:url('/img/bg/battle3.gif');
        mix-blend-mode: overlay;
        clip-path: none;
    }
    #end-screen.loser .filter {
        background-color: crimson;
        background-image:url('/img/bg/battle2.gif');
        mix-blend-mode: multiply;
    }
    #end-screen .hero {
        position: absolute;
        bottom: 0;
        right: 6%;
        width: 512px;
        height: 512px;
        background-size: contain;
        background-position: bottom center;
        animation: slideIn 1s ease-in-out;
        transform: translateX(0px);
        z-index: 1;
    }
    #end-screen.winner .hero {
        background-image:url('/ahod/winrar.png');
    }
    #end-screen.loser .hero {
        background-image:url('/ahod/loser.png');
    }
    #end-screen .options {
        position: absolute;
        top: 27%;
        left: 7%;
        width: 400px;
        background-size: contain;
        background-position: bottom center;
        animation: slideIn 1.5s ease-in-out;
        transform: translateX(0px);
        z-index: 1;
    }
    #end-screen .options button {
        margin-top: 0;
        padding: 10px 30px 7px;
        font-size: 1.7rem;
    }
    #end-screen .options h3 {
        font-family: 'CooperBits', serif;
        color: #fff;
        font-size: 3.5rem;
        font-style: italic;
        margin: 0 0 20px;
        line-height: 3.5rem;
        text-shadow: 3px 4px 0 #000, -4px -2px 0 rgba(0,0,0,0.5);
        font-weight: normal;
    }

    @keyframes blinkInOut {
        0% { opacity: 0; }
        4% { opacity: 0; }
        5% { opacity: 1; }
        6% { opacity: 0; }
        9% { opacity: 0; }
        10% { opacity: 1; }
        15% { opacity: 1; }
        16% { opacity: 0; }
        18% { opacity: 0; }
        19% { opacity: 1; }
        90% { opacity: 1; }
        95% { opacity: 1; }
        96% { opacity: 0; }
        98% { opacity: 0; }
        99% { opacity: 1; }
        100% { opacity: 0; }
    }
    @keyframes slideInOut {
        0% { transform: translateX(-300%);  }
        20% { transform: translateX(0px); }
        80% { transform: translateX(0px); }
        100% { transform: translateX(300%);  }
    }
    @keyframes slideIn {
        0% { transform: translateX(-300%); }
        100% { transform: translateX(0px); }
    }
    @keyframes instructionInOut {
        0% { clip-path: polygon(0 75%, 100% 0, 100% 0%, 0 75%); }
        20% { clip-path: polygon(0 75%, 100% 0, 100% 0%, 0 75%); }
        30% { clip-path: polygon(0 25%, 100% 0, 100% 60%, 0 75%); }
        85% { clip-path: polygon(0 25%, 100% 0, 100% 60%, 0 75%); }
        100% { clip-path: polygon(0 75%, 100% 0, 100% 0%, 0 75%); }
    }
    @keyframes filterInOut {
        0% { clip-path: polygon(0 75%, 100% 0, 100% 0%, 0 75%); opacity: 0; }
        10% { clip-path: polygon(0 0%, 100% 0, 100% 100%, 0 100%);  opacity: 1; }
        90% { clip-path: polygon(0 0%, 100% 0, 100% 100%, 0 100%); opacity: 1; }
        100% { clip-path: polygon(0 0%, 100% 100%, 100% 100%, 0 0); opacity: 0; }
    }
    @keyframes filterIn {
        0% { clip-path: polygon(0 75%, 100% 0, 100% 0%, 0 75%); }
        100% { clip-path: polygon(0 25%, 100% 0, 100% 90%, 0 75%); }
    }
    @keyframes instructionTextIn {
        0%, 30% { transform: translateX(500%) rotate(-10deg); }
        40% { transform: translateX(0px) rotate(-10deg); }
        90% { transform: translateX(0px) rotate(-10deg); }
        100% { transform: translateX(-500%) rotate(-10deg); }
    }
</style>