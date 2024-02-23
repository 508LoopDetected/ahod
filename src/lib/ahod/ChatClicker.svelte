<script>
    import { onMount, createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();
    export let timeLeft;

    $: progressPercentage = (totalClicks / 25) * 100;
    let totalClicks = 0;
    let reply;
    let npcChatScroll;
    let chats = [];
    let currentMessageIndex = 0;
    const totalMessages = 3;
    let messageId = 0;
    let timeoutIds = []; // Store timeout IDs for potential cancellation
    let gameCompleted = false;

    function addMessage(initialDelay = 0) {
        if (gameCompleted) return; // Stop if game is already completed

        const scheduleId = setTimeout(() => {
            if (gameCompleted) return; // Check again due to async delay

            const newMessage = { id: messageId, who: "npc", content: "...", animation: "slideIn 0.15s ease-out" };
            chats = [...chats, newMessage];
            
            let messageIndex = messageId % customMessages.length;
            const contentTimeoutId = setTimeout(() => {
                if (gameCompleted) return; // Final check before updating content
                
                updateMessageContent(newMessage.id, customMessages[messageIndex]);
            }, 700);

            messageId++;
            timeoutIds.push(contentTimeoutId); // Store for potential cancellation

            if (messageId < totalMessages) {
                setTimeout(() => addMessage(), 1700);
            }
        }, initialDelay);

        timeoutIds.push(scheduleId); // Store initial timeout ID
    }

    function updateMessageContent(id, content) {
        chats = chats.map(chat => chat.id === id ? { ...chat, content } : chat);
    }

    let gameStarted = false;
    let soundWin;
    let soundLose;
    let soundOhNo;

    const customMessages = [
        "Hey champ, I'm gonna need that TPS report.",
        "Are you even online?", 
        "Listen here, hotshot, you've got about 2 seconds before I report you to upper managment."
    ];
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
        soundOhNo = new Audio('/ahod/audio/ohno.mp3');
        currentDiatribe = diatribes[Math.floor(Math.random() * diatribes.length)];
        totalClicks = 0;
        addMessage();
    }

    function handleClick() {
        if (totalClicks < 25) {
            totalClicks++;
            addWordsToDiatribe();
        } else {
            // Ensure this block only executes once
            if (!gameCompleted) {
                gameCompleted = true; // Set the flag
                timeoutIds.forEach(id => clearTimeout(id)); // Cancel all pending timeouts
                timeoutIds = []; // Clear the list of timeout IDs
                addDiatribeMessage(currentDiatribe); // Add the final diatribe message
            }
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

    function addDiatribeMessage(diatribeText) {
        chats = [...chats, { id: messageId++, who: "player", content: diatribeText, animation: "slideIn 0.5s ease-out" }];
        if (soundOhNo) soundOhNo.play();
        reply.innerText = ''; // Clear the userReply content
    }

    $: if (timeLeft <= 0) {
        checkResult();
    }
</script>

{#if gameStarted}
<div id="messenger">
    <div id="chat">
        {#each chats as { id, who, content, animation }}
            <div class="npcWrap {who}" style={animation}>
                <div class="npc-pic">
                    <img src="/ahod/{who == 'player' ? 'hobbes' : 'larry'}.png" />
                </div>
                <div class="npc-msg">{content}</div>
            </div>
        {/each}
    </div>
    <div id="replyArea">
        <div id="userReply" bind:this={reply}></div>
        <div id="clickMe" on:click={handleClick}>
           SEND
        </div>
    </div>
    <div id="progressBarContainer">
        <div id="progressBarFill" style="width: {progressPercentage}%;"></div>
    </div>
</div>
{/if}

<style>
#messenger {
    width: 100%;
    height: 100%;
    /*width: 800px;
    height: 600px;*/
    background-color: #777;
    padding: 15px;
    display: flex;
    flex-direction: column;
}

#chat {
    background-color: #ccc;
    padding: 30px;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    justify-content: flex-end;
    height: 100%;
    border: 1px solid #000;
    border-radius: 5px;
    box-shadow: inset 1px 1px 0 rgba(255,255,255,0.4), 5px 5px 0 rgba(0,0,0,0.1);
}

:global(.npcWrap) {
    display: flex;
    justify-content: left;
    margin-bottom: 30px;
    animation: slideIn 0.15s ease-out;
}
:global(.npc-pic img) {
    width: 64px;
    overflow: hidden;
    height: 64px;
    object-fit: cover;
    object-position: center center;
    border-radius: 50%;
    border: 2px solid #fff;
    background-color: #000;
    margin-right: 30px;
    box-shadow: inset 1px 1px 0 rgba(255,255,255,0.4), 5px 5px 0 rgba(0,0,0,0.1);
}
:global(.npc-msg) {
    background-color: #eaeaea;
    padding: 20px;
    font-size: 13px;
    box-shadow: inset 1px 1px 0 rgba(255,255,255,0.4), 5px 5px 0 rgba(0,0,0,0.1);
    text-align: left;
}
:global(.npcWrap.player) {
    flex-direction: row-reverse;
    transition: all 0.2s ease-out;
    transform: rotate(-3deg) scale(1.1);
    position: relative;
    right: -20px;
    width: 95%;
    top: 10px;
    animation: slideInAlt 0.15s ease-out;
}
:global(.npcWrap.player .npc-pic img) {
    margin-left: 20px;
    margin-right: 0px;
    width: 84px;
    height: 84px;
    transition: all 0.2s ease-out;
}

#replyArea {
    display: flex;
    margin-top: 15px;
}

#userReply {
    height: 80px;
    border: 1px solid #000;
    width: 100%;
    overflow: hidden;
    padding: 10px;
    margin: 0 auto;
    background: #fff;
    text-align: left;
    flex-direction: column-reverse;
    display: flex;
    border-radius: 5px;
    font-size: 13px;
    box-shadow: inset 1px 1px 0 rgba(255,255,255,0.4), 5px 5px 0 rgba(0,0,0,0.1);
}

#clickMe {
    background-color: crimson;
    border: 1px solid #000;
    border-radius: 5px;
    box-shadow: inset 1px 1px 0 rgba(255,255,255,0.4), 5px 5px 0 rgba(0,0,0,0.1);
    text-shadow: -1px -1px 0 rgba(0,0,0,0.5);
    font-size: 16px;
    color: #fff;
    display: block;
    margin-left: 15px;
    padding: 10px 50px;
    cursor: pointer;
    outline: none;
    transition: background-color 0.3s;
    align-items: center;
    display: flex;
    letter-spacing: 2px;
    font-weight: bold;
}

#progressBarContainer {
    width: 100%;
    background-color: #000;
    border-radius: 3px;
    margin-top: 15px;
    border: 1px solid #fff;
    overflow: hidden;
    height: 20px;
}

#progressBarFill {
    height: 100%;
    background-color: crimson;
    border-radius: 0px;
    /* transition: width 0.5s ease-in-out;*/
}


@keyframes slideIn {
    0% { transform: translateX(-100%); }
    100% { transform: translateX(0px); }
}
@keyframes slideInAlt {
    0% { transform: translateX(100%) rotate(0deg) scale(1); }
    100% { transform: translateX(0px) rotate(-3deg) scale(1.1); }
}
</style>