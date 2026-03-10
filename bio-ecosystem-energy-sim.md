layout: page
title: "Energy Through The Ecosystem Simulation"
permalink: https://mrmoscienceland.github.io/bio-ecosystem-energy-sim
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body { font-family: 'Segoe UI', sans-serif; background: #e0e6ed; margin: 0; padding: 10px; color: #234; overflow: hidden; }
        .container { display: flex; gap: 10px; max-width: 1150px; margin: auto; height: 98vh; }
        .sidebar { width: 260px; display: flex; flex-direction: column; gap: 8px; }
        .panel { background: #fff; padding: 10px; border-radius: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        .player-card { background: #2c3e50; color: white; border-left: 5px solid #3498db; background-size: cover; background-position: center; min-height: 80px; }
        .game-area { flex-grow: 1; display: flex; flex-direction: column; gap: 8px; }
        .summary-box { background: #fff; padding: 12px; border-radius: 8px; border-left: 5px solid #27ae60; min-height: 125px; font-size: 0.95em; }
        
        /* Grid fixed for 13" Chromebook screens */
        .grid { display: grid; grid-template-columns: repeat(10, 1fr); gap: 4px; background: #bdc3c7; padding: 8px; border-radius: 8px; }
        .card { 
            aspect-ratio: 1/1; background: #34495e; border-radius: 4px; cursor: pointer;
            display: flex; align-items: center; justify-content: center; color: white;
            font-size: 14px; background-size: cover; background-position: center; border: 2px solid transparent;
            min-height: 50px; /* Ensures visibility even if aspect-ratio fails */
        }
        .card.revealed { border: 2px solid #f1c40f; cursor: not-allowed; opacity: 1; }
        .card.npc-used { border: 2px solid #e74c3c; } 
        .card.dead { 
			filter: grayscale(100%) brightness(40%); 
			border: 2px solid #666;
			opacity: 0.6;
			cursor: default;
			position: relative;
		}

		/* Optional: adds a small skull or cross icon in the corner of dead cards */
		.card.dead::after {
			content: "💀";
			position: absolute;
			top: 2px;
			right: 2px;
			font-size: 12px;
		}
        
        button { padding: 10px; background: #27ae60; color: white; border: none; border-radius: 5px; cursor: pointer; font-weight: bold; width: 100%; }
        button:disabled { background: #95a5a6; cursor: not-allowed; }
		.setup-btn {
			width: 160px;
			height: 160px;
			margin: 10px;
			border: 4px solid #fff;
			border-radius: 15px;
			cursor: pointer;
			font-weight: bold;
			font-size: 1.1em;
			color: white;
			text-shadow: 2px 2px 4px #000;
			background-size: cover;
			background-position: center;
			transition: all 0.2s ease;
			display: inline-flex;
			align-items: flex-end;
			justify-content: center;
			padding-bottom: 15px;
			box-shadow: 0 4px 8px rgba(0,0,0,0.2);
		}

		.setup-btn:hover {
			transform: scale(1.05);
			box-shadow: 0 6px 12px rgba(0,0,0,0.3);
			border-color: #27ae60;
		}
    </style>
</head>
<body>

<div id="setup" style="text-align:center; padding-top: 15vh;">
    <h1>Ecosystem Energy Lab</h1>
    <p>Select your species (Plankton are autotrophs, Shrimp ONLY eat Plankton, Cod ONLY eat Shrimp, Sharks ONLY eat Cod):</p>
    
	<button class="setup-btn" style="background-image: linear-gradient(rgba(0,0,0,0), rgba(0,0,0,0.6)), url('https://lh3.googleusercontent.com/u/0/d/1zfUJtVmS-MC_uOWPMRx6q-yfAX6mfNa5');" onclick="startGame('Plankton')">Plankton</button>
    
    <button class="setup-btn" style="background-image: linear-gradient(rgba(0,0,0,0), rgba(0,0,0,0.6)), url('https://lh3.googleusercontent.com/u/0/d/1h_AbkT6tG0tT2S5XOVWjoYA-CHipfCtD');" onclick="startGame('Shrimp')">Shrimp</button>
    
    <button class="setup-btn" style="background-image: linear-gradient(rgba(0,0,0,0), rgba(0,0,0,0.6)), url('https://lh3.googleusercontent.com/u/0/d/1a-KvhTTH_nQSfviDCipGqyL2DmVC-TFV');" onclick="startGame('Cod')">Cod</button>
    
    <button class="setup-btn" style="background-image: linear-gradient(rgba(0,0,0,0), rgba(0,0,0,0.6)), url('https://lh3.googleusercontent.com/u/0/d/1WwDfTRMyZg-2ySIi_gpmdfYDc4Ts-6AN');" onclick="startGame('Shark')">Shark</button>
</div>

<div id="game" class="container" style="display:none;">
    <div class="sidebar">
        <div id="pCard" class="panel player-card">
            <strong>ROLE: <span id="pRole"></span></strong><br>
            Energy: <span id="pEnergyDisplay">15</span><br>
            Round: <span id="pRound">1</span> / 10
        </div>
        <div class="panel">
            <strong>Living Population:</strong><br>
            🌿 Plankton: <span id="c-Plankton">--</span><br>
            🦐 Shrimp: <span id="c-Shrimp">--</span><br>
            🐟 Cod: <span id="c-Cod">--</span><br>
            🦈 Shark: <span id="c-Shark">--</span>
        </div>
        <button id="nextBtn" onclick="advanceRound()" disabled>End Hunt Phase</button>
    </div>

    <div class="game-area">
        <div id="summary" class="summary-box"><strong>Hunt Phase:</strong> Click a hidden card!</div>
        <div id="grid" class="grid"></div>
    </div>
</div>

<div id="results" style="display:none; text-align:center; padding: 50px;">
    <h2>Simulation Complete</h2>
    <div id="finalTable" style="display:inline-block; background:white; padding:20px; border-radius:10px;"></div><br><br>
    <button style="width:200px;" onclick="location.reload()">Start New Session</button>
</div>

<script>
let playedRoles = []; // Tracks which organisms the student has finished
// --- PASTE YOUR IMAGE LINKS HERE ---
const imageMap = {
    'Plankton': 'https://lh3.googleusercontent.com/u/0/d/1zfUJtVmS-MC_uOWPMRx6q-yfAX6mfNa5',
    'Shrimp':   'https://lh3.googleusercontent.com/u/0/d/1h_AbkT6tG0tT2S5XOVWjoYA-CHipfCtD',
    'Cod':      'https://lh3.googleusercontent.com/u/0/d/1a-KvhTTH_nQSfviDCipGqyL2DmVC-TFV',
    'Shark':    'https://lh3.googleusercontent.com/u/0/d/1WwDfTRMyZg-2ySIi_gpmdfYDc4Ts-6AN'
};

let round = 1;
let playerRole = '';
let organisms = [];
let studentRevealedIds = []; 
let npcUsedThisRound = [];   
const preyMap = { 'Shrimp': 'Plankton', 'Cod': 'Shrimp', 'Shark': 'Cod' };

function shuffleArray(array) {
    for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
    }
}

function startGame(role) {
    if (!playedRoles.includes(role)) playedRoles.push(role);
	playerRole = role;
    document.getElementById('setup').style.display = 'none';
    document.getElementById('game').style.display = 'flex';
    document.getElementById('pRole').innerText = role;
    document.getElementById('pCard').style.backgroundImage = `linear-gradient(rgba(0,0,0,0.6), rgba(0,0,0,0.6)), url(${imageMap[role]})`;
    
    let id = 0;
    const counts = { 'Plankton': 20, 'Shrimp': 12, 'Cod': 6, 'Shark': 2 };
    for (let type in counts) {
        for (let i = 0; i < counts[type]; i++) {
            organisms.push({ id: id++, type: type, energy: 15, alive: true });
        }
    }
    shuffleArray(organisms);
	
	updateUI();
    renderGrid();
}

function renderGrid() {
    const grid = document.getElementById('grid');
    grid.innerHTML = '';
    
    organisms.forEach(org => {
        const card = document.createElement('div');
        card.className = 'card';
        
        // Use a variable to determine if the image should be shown
        const isDead = !org.alive;
        const isStudentClicked = studentRevealedIds.includes(org.id);

        if (isDead) {
            // Priority 1: Dead cards (Always show image, but grayscale)
            card.classList.add('dead');
            card.style.backgroundImage = `url(${imageMap[org.type]})`;
            card.innerText = ""; 
        } 
        else if (isStudentClicked) {
            // Priority 2: Living cards the student clicked (Show full color image)
            card.classList.add('revealed');
            card.style.backgroundImage = `url(${imageMap[org.type]})`;
            card.innerText = "";
        } 
        else if (npcUsedThisRound.includes(org.id)) {
            // Priority 3: NPC currently "on" this card (Show X, hide image)
            card.classList.add('npc-used');
            card.innerText = "✕";
        } 
        else {
            // Priority 4: Default hidden state
            card.innerText = "?";
        }
        
        card.onclick = () => handleHunt(org);
        grid.appendChild(card);
    });
}

function handleHunt(target) {
    if (studentRevealedIds.includes(target.id) || npcUsedThisRound.includes(target.id) || !target.alive || !document.getElementById('nextBtn').disabled) return;

    studentRevealedIds.push(target.id);
    let log = "<strong>Hunt Results:</strong><br>";
    
    if (preyMap[playerRole] === target.type) {
        organisms.filter(o => o.type === playerRole && o.alive).forEach(o => o.energy += 5);
        target.energy -= 5;
        log += `🟢 Success: You found a ${target.type} (+5 Energy).<br>`;
    } else if (preyMap[target.type] === playerRole) {
        target.energy += 5;
        organisms.filter(o => o.type === playerRole && o.alive).forEach(o => o.energy -= 5);
        log += `🔴 Danger: A ${target.type} found you! (-5 Energy).<br>`;
    } else {
        log += `⚪ Neutral encounter with a ${target.type}. No energy transfer.<br>`;
    }

    // NPC Hunt
    organisms.forEach(npc => {
        if (npc.alive && npc.type !== 'Plankton' && !studentRevealedIds.includes(npc.id) && !npcUsedThisRound.includes(npc.id)) {
            // --- Replace the 'available' line inside the handleHunt function with this: ---
			let available = organisms.filter(o => !npcUsedThisRound.includes(o.id) && o.alive);
            if (available.length > 0) {
                let pick = available[Math.floor(Math.random() * available.length)];
                npcUsedThisRound.push(pick.id);
                if (preyMap[npc.type] === pick.type && pick.type !== playerRole) {
                    npc.energy += 5;
                    pick.energy -= 5;
                }
            }
        }
    });

    document.getElementById('summary').innerHTML = log;
    document.getElementById('nextBtn').disabled = false;
    updateUI();
    renderGrid();
}

function advanceRound() {
    // 1. Energy Logic
    organisms.forEach(org => {
        if (org.alive) {
            org.energy -= 2; 
            if (org.type === 'Plankton') org.energy += 10; 
            if (org.energy <= 0) { org.energy = 0; org.alive = false; }
        }
    });

    // 2. Population Check
    const shrimpCount = organisms.filter(o => o.type === 'Shrimp' && o.alive).length;
    const codCount = organisms.filter(o => o.type === 'Cod' && o.alive).length;
    let collapse = false;

    if (shrimpCount < 4 || codCount < 2) {
        collapse = true;
    }

    npcUsedThisRound = []; 
    updateUI();

    if (collapse) {
        document.getElementById('summary').innerHTML = `<strong style="color:red;">ECOSYSTEM COLLAPSE!</strong><br>Shrimp: ${shrimpCount}, Cod: ${codCount}. Population levels are too low to sustain the food web.`;
        setTimeout(showResults, 3000); 
    } else if (round < 10) {
        round++;
        document.getElementById('pRound').innerText = round;
        document.getElementById('nextBtn').disabled = true;
        document.getElementById('summary').innerHTML = "<strong>Energy Updated:</strong> Metabolic loss (-2) and Solar gain (+10) applied. <br>Pick a new hidden card!";
        renderGrid();
    } else {
        showResults();
    }
}

function updateUI() {
    ['Plankton', 'Shrimp', 'Cod', 'Shark'].forEach(t => {
        const count = organisms.filter(o => o.type === t && o.alive).length;
        document.getElementById(`c-${t}`).innerText = count;
    });
    const pSample = organisms.find(o => o.type === playerRole && o.alive);
    document.getElementById('pEnergyDisplay').innerText = pSample ? pSample.energy : "0";
}

function showResults() {
    document.getElementById('game').style.display = 'none';
    document.getElementById('results').style.display = 'block';
    
    let resHtml = "<h3>Final Ecosystem Biomass</h3><table border='1' cellpadding='10' style='width:100%; border-collapse:collapse; background:white;'><tr><th>Organism</th><th>Total Energy (Biomass)</th><th>Survivors</th></tr>";
    
    ['Plankton', 'Shrimp', 'Cod', 'Shark'].forEach(t => {
        let totalE = organisms.filter(o => o.type === t).reduce((sum, o) => sum + o.energy, 0);
        let count = organisms.filter(o => o.type === t && o.alive).length;
        resHtml += `<tr><td>${t}</td><td>${totalE} Units</td><td>${count}</td></tr>`;
    });
    
    resHtml += "</table><br>"; // Close the table
    
    // Determine button text based on progress
    let buttonText = playedRoles.length < 4 ? "Play Next Organism" : "All Trials Complete - Restart Lab";
    
    // Add the button to the HTML string
    resHtml += `<button style="width:250px; background:#27ae60; color:white; padding:15px; border:none; border-radius:5px; cursor:pointer; font-weight:bold;" onclick="resetToMenu()">${buttonText}</button>`;
    
    document.getElementById('finalTable').innerHTML = resHtml;
}
function resetToMenu() {
    // 1. Reset round and board tracking
    round = 1;
    organisms = [];
    studentRevealedIds = [];
    npcUsedThisRound = [];

    // 2. Update UI labels
    document.getElementById('pRound').innerText = "1";
    document.getElementById('summary').innerHTML = "<strong>Hunt Phase:</strong> Click a hidden card!";
    document.getElementById('nextBtn').disabled = true;
    // 3. If they finished all 4, clear the history for a fresh lab start
    if (playedRoles.length >= 4) playedRoles = [];

    // 4. Switch screens
    document.getElementById('results').style.display = 'none';
    document.getElementById('setup').style.display = 'block';

    // 5. Visually update the setup buttons
    const buttons = document.querySelectorAll('.setup-btn');
    buttons.forEach(btn => {
        // We check the original name from a data attribute or clean text
        let role = btn.innerText.split(' ')[0]; 
        if (playedRoles.includes(role)) {
            btn.innerText = role + " (Completed)";
            btn.style.filter = "grayscale(80%)";
            btn.style.opacity = "0.7";
        } else {
            btn.innerText = role;
            btn.style.filter = "none";
            btn.style.opacity = "1";
        }
    });
}
</script>
</body>
</html>
