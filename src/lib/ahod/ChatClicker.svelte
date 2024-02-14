<script>
    import { onMount, createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();
    export let timeLeft;

    let totalClicks = 0;
    let message = '';
    let reply;
    let npcChatScroll;
     let chats = []; // This will hold our messages
      
      // Function to generate the HTML snippet
      function generateHtmlSnippet() {
        return `
          <div class="npcWrap">
              <div class="npc-pic">
                  <img src="/ahod/background-blitz/larry.png" />
              </div>
              <div class="npc-msg">
                  ...
              </div>
          </div>
        `;
      }

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

        const intervalId = setInterval(() => {
          // Instead of directly manipulating the DOM, we're updating the Svelte store
          chats = [...chats, generateHtmlSnippet()];
        }, 2000);

        return () => clearInterval(intervalId);
    }

    function handleClick() {
        if (totalClicks < 25) {
            totalClicks++;
            addWordsToDiatribe();
        } else {
            alert('done son');
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

<div id="messenger">
    <div id="chat">
        {#each chats as chatHtml, index}
            {@html chatHtml}
        {/each}
    </div>
    <div id="replyArea">
        <div id="userReply" bind:this={reply}></div>
        <div id="clickMe" on:click={handleClick}>
            <div>Clicks: {totalClicks}</div>
        </div>
    </div>
</div>

{/if}

<style>
#messenger {
    min-height: 600px;
    max-height: 600px;
    background-color: #777;
    width: 100vw;
    padding: 15px;
    display: grid;
    grid-template-columns: 1fr;
    grid-template-rows: 1fr 1fr;
    grid-column-gap: 0;
    grid-row-gap: 0;
}

#chat {
    padding: 30px;
    overflow: scroll;
    border: 1px solid blue;
    flex-direction: column-reverse;
    display: flex;
}

#replyArea {
    grid-area: 2 / 1;
}

:global(.npcWrap) {
    display: flex;
    justify-content: left;
    margin-bottom: 30px;
    animation: slideIn 1s ease-out;
}
:global(.npc-pic img) {
    width: 64px;
    overflow: hidden;
    height: 64px;
    object-fit: cover;
    object-position: center center;
    border-radius: 50%;
    border: 2px solid #ccc;
    margin-right: 30px;
}
:global(.npc-msg) {
    background-color: #aaa;
    padding: 20px;
}

#clickMe {
    display: block;
    background: red;
    margin: 0 auto;
    width: 100%;
    height: 100px;
    cursor: pointer;
}

#userReply {
    height: 200px;
    border: 2px outset grey;
    width: 100%;
    overflow-y: scroll;
    padding: 10px;
    margin-bottom: 20px;
    margin: 0 auto;
    background: #fff;
    text-align: left;
    flex-direction: column-reverse;
    display: flex;
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

@keyframes slideIn {
    0% { transform: translateX(-300%); }
    100% { transform: translateX(0px); }
}
</style>