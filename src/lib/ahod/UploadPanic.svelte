<script>
    import { onMount, createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();
    export let timeLeft;

    // Logic
    let interval;
    let totalWins = 0;
    let message = '';
    let shuffleIntervals = [null, null, null];

    // Audio
    let gameStarted = false;
    let soundWin;
    let soundLose;

    onMount(() => {
        startGame();
    });

    function startGame() {
        gameStarted = true;
        soundWin = new Audio('/ahod/audio/win.wav');
        soundLose = new Audio('/ahod/audio/lose.wav');
        files = Array.from({ length: totalFiles }, (_, index) => ({
            id: index,
            uploaded: false,
            position: getRandomPosition() // Assign random position
        }));
        uploadedCount = 0;
    }

    const totalFiles = 5;
    let files = [];
    let uploadedCount = 0;

    function getRandomPosition() {
        const maxX = 40; // Constrain to the left half
        const maxY = 80; // Max percentage for Y position
        return {
            x: Math.floor(Math.random() * maxX),
            y: Math.floor(Math.random() * maxY)
        };
    }

    // Define the logic to handle file dragging and dropping
    function onDropFile(fileId) {
        // Logic to handle when a file is dropped on the target area
        files = files.map(file => file.id === fileId ? {...file, uploaded: true} : file);
        uploadedCount = files.filter(file => file.uploaded).length;
        checkSelection();
    }

    let draggedFileId = null;

    function onDragStart(event, fileId) {
        draggedFileId = fileId;
        event.dataTransfer.setData('text/plain', fileId); // Required for drag to work
    }

    function onDragOver(event) {
        event.preventDefault(); // Necessary to allow a drop
    }

    function onDrop(event, zone) {
        event.preventDefault();
        if (zone === 'drop-zone' && draggedFileId !== null) {
            const dropZoneRect = document.getElementById('drop-zone').getBoundingClientRect();
            files = files.map(file => {
                if (file.id === draggedFileId) {
                    return {
                        ...file, 
                        uploaded: true,
                        position: {
                            x: 50 + (Math.random() * 40), // Randomize within the right half
                            y: Math.random() * 80
                        }
                    };
                }
                return file;
            });

            uploadedCount++;
            /*if (uploadedCount === totalFiles) {
                checkSelection();
            }*/
            draggedFileId = null;
        }
    }

    function checkSelection() {
        if (uploadedCount === totalFiles) {
            if (soundWin) soundWin.play();
            dispatch('win');
        } else {
            if (soundLose) soundLose.play();
            dispatch('lose');
        }
    }

    $: if (timeLeft <= 0) {
        checkSelection();
    }
</script>


<div id="game-container">
    {#if gameStarted}
        <div id="browser">
            <div id="file-container">
                {#each files as file (file.id)}
                    <div class={`file ${file.uploaded ? 'uploaded' : ''}`}
                         draggable="true"
                         style="top: {file.position.y}%; left: {file.position.x}%;"
                         on:dragstart={(event) => onDragStart(event, file.id)}>
                        File {file.id + 1}
                    </div>
                {/each}
            </div>
            <div id="drop-zone" on:drop={(event) => onDrop(event, 'drop-zone')}
                 on:dragover={onDragOver}>
                Drop files here
            </div>
        </div>
    {/if}
</div>

<style>
#browser {
    display: grid;
    grid-template-columns: 1fr 1fr;
    grid-gap: 10px;
    margin-bottom: 20px;
    position: relative;
    overflow: hidden;
    height: 600px;
    width: 100%;
}

#file-container {
    display: grid;
    grid-template-columns: 1fr 1fr;
    grid-gap: 10px;
    margin-bottom: 20px;
    overflow: hidden;
    height: 100%;
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
    right: 0; /* Position on the right side */
    width: 50%; /* Adjust as needed */
    height: 600px;
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