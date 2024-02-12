<script>
    import { onMount, onDestroy } from 'svelte';

    let timer = 5.00;
    let interval;
    let message = '';
    let showTimer = true;
    let totalWins = 0;
    let gameStarted = false;

    let soundWin;
    let soundLose;

    function startGame() {
        soundWin = new Audio('/path/to/win-sound.mp3');
        soundLose = new Audio('/path/to/lose-sound.mp3');
        gameStarted = true;
        resetGame();
    }

    function resetGame() {
        timer = 5.00;
        message = '';
        showTimer = true;
        startTimer();
    }

    function startTimer() {
        clearInterval(interval);
        interval = setInterval(() => {
            if (timer > 0) {
                timer -= 0.1;
            } else {
                clearInterval(interval);
                endGame(false); // Automatically lose if timer runs out
            }
        }, 100);
    }

    function endGame(won) {
        clearInterval(interval);
        showTimer = false;
        if (won) {
            soundWin.play();
            message = 'Win!';
            totalWins++;
        } else {
            soundLose.play();
            message = 'Lose!';
        }
    }

    onDestroy(() => {
        clearInterval(interval);
    });
</script>

<div id="game-container">
    
    {#if !gameStarted}
        <button on:click={startGame}>Start Game</button>
    {:else}
        {#if showTimer}
            <span class="timer">Time: {timer.toFixed(1)}</span><br/>
            <button on:click={() => endGame(true)}>Win</button>
            <button on:click={() => endGame(false)}>Lose</button>
        {:else}
            <div id="message">{message}</div>
            <div>Total Wins: {totalWins}</div>
            <button on:click={startGame}>Play Again</button>
        {/if}
    {/if}
</div>


<style>
    #game-container {
        width: 1000px;
        border: 1px solid #ddd;
        box-shadow: 0 4px 8px rgba(0,0,0,0.1);
        padding: 20px;
        text-align: center;
        margin: 0 auto;
        background-color: #f0f0f0;
    }

    .timer {
        font-size: 2rem;
        margin-bottom: 20px;
    }

    #message {
        margin: 10px auto;
        font-size: 1.5rem;
    }

    button {
        margin: 5px;
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

    /* Additional styles as needed */
</style>
