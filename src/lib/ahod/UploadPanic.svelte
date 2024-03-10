<script>
    import { onMount, onDestroy, createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();
    export let timeLeft;

    // Logic
    let interval;
    let totalWins = 0;
    let message = '';
    let shuffleIntervals = [null, null, null];

    const totalFiles = 5;
    let files = [];
    let uploadedCount = 0;

    let draggedFileId = null;
    let isDragging = false;
    let currentFileElement = null;
    let offsetX = 0, offsetY = 0; // To store the initial click position within the element

    // Audio
    let gameStarted = false;
    let soundWin;
    let soundLose;


    function getImagePath(fileId) {
        return `/ahod/upload-panic/file${fileId + 1}.png`;
    }

    function onMouseDown(event, fileId) {
        const fileElement = event.target;
        if (!fileElement.classList.contains('file')) return; // Ensures we are clicking on a file

        isDragging = true;
        currentFileElement = fileElement;
        draggedFileId = fileId; // Use the existing draggedFileId variable

        // Calculate offset
        const rect = fileElement.getBoundingClientRect();
        offsetX = event.clientX - rect.left;
        offsetY = event.clientY - rect.top;

        // Prevent default action to avoid selecting text or other elements while dragging
        event.preventDefault();
    }

    function getRandomPosition(width, height, yOffset = 0) {
        const fileWidth = 100; // Width of the file elements
        const fileHeight = 100; // Height of the file elements
        // Adjust x and y to account for the size of the file icons
        const x = Math.floor(Math.random() * (width - fileWidth));
        const y = Math.floor(Math.random() * (height * 0.4 - fileHeight)) + yOffset; // Place files in the bottom 40%
        return {
            x: `${x}px`,
            y: `${y}px`
        };
    }

    function onMouseMove(event) {
        if (!isDragging || !currentFileElement) return;

        const browserRect = document.getElementById('browser').getBoundingClientRect();
        let newX = event.clientX - browserRect.left - offsetX;
        let newY = event.clientY - browserRect.top - offsetY;

        // Constrain newX and newY within the #browser boundaries
        newX = Math.max(0, Math.min(newX, browserRect.width - currentFileElement.offsetWidth));
        newY = Math.max(0, Math.min(newY, browserRect.height - currentFileElement.offsetHeight));

        currentFileElement.style.left = `${newX}px`;
        currentFileElement.style.top = `${newY}px`;
    }

    function onMouseUp(event) {
        if (!isDragging || !currentFileElement) return;

        const browserRect = document.getElementById('browser').getBoundingClientRect();
        const mousePosX = event.clientX - browserRect.left;
        const mousePosY = event.clientY - browserRect.top;

        // Check if drop is within the top 60% of #browser
        if (mousePosY >= 0 && mousePosY <= browserRect.height * 0.6) {
            const newX = mousePosX - offsetX;
            const newY = mousePosY - offsetY;

            updateFilePosition(draggedFileId, newX, newY);
            onDropFile(draggedFileId); // Mark as uploaded
        }

        isDragging = false;
        currentFileElement = null;
        draggedFileId = null;
    }

    function updateFilePosition(fileId, newX, newY) {
        files = files.map(file => {
            if (file.id === fileId) {
                // Convert newX and newY to 'px' units for CSS
                return { ...file, position: { x: newX + 'px', y: newY + 'px' }, uploaded: true };
            }
            return file;
        });
    }


    onMount(startGame);


    function startGame() {
        gameStarted = true;
        soundWin = new Audio('/ahod/audio/win.wav');
        soundLose = new Audio('/ahod/audio/lose.wav');

        // Get the dimensions of #browser to pass to getRandomPosition
        const browserRect = document.getElementById('browser').getBoundingClientRect();
        files = Array.from({ length: totalFiles }, (_, index) => ({
            id: index,
            uploaded: false,
            // yOffset is set to the height of the drop zone to ensure it starts in the bottom 40%
            position: getRandomPosition(browserRect.width, browserRect.height, browserRect.height * 0.6)
        }));
        uploadedCount = 0;
    }

    function onDropFile(fileId) {
        files = files.map(file => file.id === fileId ? {...file, uploaded: true} : file);
        uploadedCount = files.filter(file => file.uploaded).length;
        
        // Only call checkSelection if all files have been attempted to be uploaded
        /*if (uploadedCount === files.length) {
            checkSelection();
        }*/
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

<div id="browser">
    {#if gameStarted}
        <div id="drop-zone">
          Drop files here
        </div>
        <div id="file-container">
          {#each files as file (file.id)}
            <img class={`file ${file.uploaded ? 'uploaded' : ''}`}
                 src={getImagePath(file.id)}
                 alt={`File ${file.id + 1}`}
                 style="left: {file.position.x}; top: {file.position.y};"
                 on:mousedown={(event) => onMouseDown(event, file.id)}
                 draggable="false" /> <!-- Images are draggable by default, so we disable it -->
          {/each}
        </div>
    {/if}
</div>

<svelte:window on:mousemove={onMouseMove} on:mouseup={onMouseUp} />

<style>
#browser {
    margin-bottom: 20px;
    position: relative;
    overflow: hidden;
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
}

#drop-zone {
    display: flex;
    align-content: center;
    position: relative;
    right: 0;
    height: 60%;
    flex-wrap: wrap;
    background-color: #fff;
    justify-content: center;
    text-transform: uppercase;
    letter-spacing: 2px;
    font-size: 1.5rem;
    color: #aaa;
}
#drop-zone:after {
    content: '';
    position: absolute;
    display: block;
    border: 2px dashed #ccc;
    width: 90%;
    height: 82%;
    left: 0;
    right: 0;
    margin: auto;
    top: 50%;
    transform: translateY(-50%);
}

#file-container {
    background-color: #ccc;
    height: 40%;
    border-top: 10px solid #aaa;
}

.file {
    padding: 0;
    width: 100px;
    height: 100px;
    cursor: pointer;
    position: absolute;
}

.file.uploaded {
    opacity: 0.5;
}
</style>