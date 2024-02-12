<script>
    import { onMount, createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();
    export let timeLeft;

    let totalClicks = 0;
    let message = '';
    let reply;

    let gameStarted = false;
    let soundWin;
    let soundLose;

    const diatribes = [
        "I'd just like to interject for a moment. What you're referring to as Linux, is in fact, GNU/Linux, or as I've recently taken to calling it, GNU plus Linux. Linux is not an operating system unto itself, but rather another free component of a fully functioning GNU system made useful by the GNU corelibs, shell utilities and vital system components comprising a full OS as defined by POSIX. Many computer users run a modified version...",
        "Per our last chat, I'm circling back to synergize our core competencies and streamline our paradigm shift towards a more holistic approach. Leveraging our bandwidth, let's deep dive into the data-driven metrics and touch base offline for a granular analysis. We need to pivot strategically to elevate our game-changing innovation while maintaining alignment with blue-sky thinking. Let's unpack this further in our touchpoint to optimize our bandwidth and actualize our roadmap to success. Stay agile."
    ];

    let currentDiatribe = '';
    let wordsAdded = 0;
    const wordsPerClick = 3;

    onMount(() => {
        startGame();
    });

    function startGame() {
        gameStarted = true;
        soundWin = new Audio('/ahod/audio/win.wav');
        soundLose = new Audio('/ahod/audio/lose.wav');
        currentDiatribe = diatribes[Math.floor(Math.random() * diatribes.length)];
        totalClicks = 0;
        message = '';
    }

    function handleClick() {
        if (totalClicks < 25) {
            totalClicks++;
            addWordsToDiatribe();
        }
    }

    function addWordsToDiatribe() {
        const words = currentDiatribe.split(/(\s+)/).filter(word => word.trim().length > 0);
        wordsAdded = totalClicks * wordsPerClick;
        reply.innerText = words.slice(0, wordsAdded).join(' ');
    }

    function checkResult() {
        if (totalClicks >= 25) {
            if (soundWin) soundWin.play();
            dispatch('win');
        } else {
            if (soundLose) soundLose.play();
            dispatch('lose');
        }
    }

    $: if (timeLeft <= 0) {
        checkResult();
    }
</script>

{#if gameStarted}
    <div id="userReply" bind:this={reply}></div>
    <div id="clickMe" on:click={handleClick}>
        <div>Clicks: {totalClicks}</div>
    </div>
{/if}

<style>
#clickMe {
    display: block;
    background: red;
    margin: 0 auto 30px;
    width: 300px;
    height: 100px;
    cursor: pointer;
}

#browser {
    display: grid;
    grid-template-columns: 1fr 1fr;
    grid-gap: 10px;
    margin-bottom: 20px;
    position: relative;
    overflow: hidden;
}

#file-container {
    display: grid;
    grid-template-columns: 1fr 1fr;
    grid-gap: 10px;
    margin-bottom: 20px;
    overflow: hidden;
    min-height: 600px;
}

.file {
    padding: 5px;
    width: 100px;
    height: 100px;
    border: 1px solid #ddd;
    background-color: #f0f0f0;
    cursor: pointer;
        position: absolute;
        transition: top 0.5s, left 0.5s;
}
.file.uploaded {
    opacity: 0.5;
}

#drop-zone {
    margin-top: 20px;
    padding: 10px;
    border: 2px dashed #007bff;
    text-align: center;
    position: absolute;
    right: 0;
    width: 50%;
    height: 600px;
}

#userReply {
    height: 200px;
    border: 2px outset grey;
    width: 400px;
    overflow-y: scroll;
    padding: 10px;
    margin-bottom: 20px;
    margin: 0 auto;
    background: #fff;
    text-align: left;
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