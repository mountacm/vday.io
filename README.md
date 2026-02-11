<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Connections</title>

<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700&display=swap" rel="stylesheet">

<style>    body {
        font-family: Arial, sans-serif;
        text-align: center;
        margin: 0;
        background: #B46CC8; /* NYT Connections purple */
    }

    /* ---------------- PRE GAME SCREEN ---------------- */

    #startScreen {
        height: 100vh;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        color: black;
        padding: 20px;
    }

    .nyt-title {
        font-family: 'Playfair Display', Georgia, serif;
        font-size: 56px;
        font-weight: bold;
        margin-bottom: 10px;
    }

    .subtitle {
        font-size: 18px;
        max-width: 500px;
        margin-bottom: 30px;
    }

    .play-btn {
        background: black;
        color: white;
        border: none;
        padding: 14px 36px;
        font-size: 18px;
        border-radius: 999px;
        cursor: pointer;
        transition: transform 0.2s ease, opacity 0.2s ease;
    }

    .play-btn:hover {
        transform: scale(1.05);
        opacity: 0.9;
    }

    .byline {
        margin-top: 25px;
        font-size: 14px;
    }

    /* ---------------- GAME AREA ---------------- */

    #game {
        display: none;
        background: #000000;
        min-height: 100vh;
        padding-bottom: 40px;
    }

    h1 {
        color: #FF1493;
        padding-top: 20px;
    }

    .grid {
        display: grid;
        grid-template-columns: repeat(4, 120px);
        gap: 10px;
        justify-content: center;
        margin-top: 20px;
        transition: all 0.3s ease;
    }

    .word {
        background: white;
        padding: 10px;
        border-radius: 6px;
        cursor: pointer;
        border: 2px solid #ccc;
        transition: background 0.2s, border-color 0.2s, opacity 0.4s ease, transform 0.4s ease;
    }

    .word.selected {
        background: #d3d3d3;
        border-color: #a9a9a9;
    }

    .fade-out {
        opacity: 0;
        transform: scale(0.9);
    }

    .solved-rect {
        width: 520px;
        padding: 15px;
        border-radius: 14px;
        color: black;
        margin: 12px auto;
        text-align: center;
        font-weight: bold;
        font-size: 18px;
        opacity: 0;
        transform: scale(0.95);
        transition: opacity 0.4s ease, transform 0.4s ease;
    }

    .solved-rect.show {
        opacity: 1;
        transform: scale(1);
    }

    .solved-rect .clues {
        font-weight: normal;
        margin-top: 6px;
        font-size: 16px;
    }

    button.submit-btn {
        margin-top: 20px;
        padding: 10px 20px;
        font-size: 16px;
        cursor: pointer;
    }

    #mistakes {
        color: white;
        margin-top: 10px;
        font-size: 22px;
        letter-spacing: 4px;
    }

    #message {
        color: white;
        margin-top: 10px;
    }
</style>
</head>

<body>

<!-- PRE GAME SCREEN -->
<div id="startScreen">
    <div class="nyt-title">CONNECTIONS</div>
    <div class="subtitle">
        Group words that share a common thread between the two of us
    </div>
    <button class="play-btn" onclick="startGame()">Play</button>
    <div class="byline">
        February 11, 2026<br>
        By Christine Mountain
    </div>
</div>

<!-- GAME SCREEN -->
<div id="game">
    <h1>I love connecting with you &lt;3</h1>
    <p style="color:white;">Create four groups of four!</p>

    <div id="solved"></div>
    <div class="grid" id="grid"></div>

    <button class="submit-btn" onclick="checkSelection()">Submit</button>

    <p id="mistakes">● ● ● ●</p>
    <p id="message"></p>
</div>

<script>
function startGame() {
    document.getElementById("startScreen").style.display = "none";
    document.getElementById("game").style.display = "block";
}

/* ---------------- GAME LOGIC ---------------- */

const categories = {
    "PLACES WE'VE BEEN": ["CA", "TX", "NYC", "DE"],
    "TIME SPENT LOVING YOU": ["1 YR", "5 MOS", "518 DAYS", "746K MINS"],
    "LAST NAMES OF ARTISTS WE'VE SEEN TOGETHER": ["DACUS", "CHILDERS", "FENDER", "CAT"],
    "LAST WORD OF FREQUENTED RESTAURANTS": ["PASS", "PEZ", "SUSIE'S", "POINT"]
};

const categoryColors = ["#6AAA64", "#F7DA21", "#4C8EDA", "#B46CC8"];

let allWords = Object.values(categories)
    .flat()
    .sort(() => Math.random() - 0.5);

const grid = document.getElementById("grid");
const solved = document.getElementById("solved");
const message = document.getElementById("message");
const mistakesDisplay = document.getElementById("mistakes");

let selected = [];
let foundCategories = [];
let mistakesRemaining = 4;
let gameOver = false;

function updateMistakeDots() {
    mistakesDisplay.textContent = [...Array(4)]
        .map((_, i) => (i < mistakesRemaining ? "●" : "○"))
        .join(" ");
}
updateMistakeDots();

function renderGrid() {
    allWords.forEach(word => {
        const div = document.createElement("div");
        div.textContent = word;
        div.classList.add("word");
        div.onclick = () => toggleSelect(div, word);
        grid.appendChild(div);
    });
}

function toggleSelect(div, word) {
    if (gameOver) return;

    if (div.classList.contains("selected")) {
        div.classList.remove("selected");
        selected = selected.filter(w => w !== word);
    } else if (selected.length < 4) {
        div.classList.add("selected");
        selected.push(word);
    }
}

function checkSelection() {
    if (gameOver) return;

    if (selected.length !== 4) {
        message.textContent = "Select exactly 4 words!";
        return;
    }

    let matchedCategory = Object.entries(categories).find(
        ([cat, words]) =>
            words.every(w => selected.includes(w)) &&
            !foundCategories.includes(cat)
    );

    if (matchedCategory) {
        const [catName, words] = matchedCategory;
        foundCategories.push(catName);
        const color = categoryColors[foundCategories.length - 1];

        message.textContent = `Correct!`;

        const tiles = selected.map(word =>
            [...document.querySelectorAll(".word")]
                .find(el => el.textContent === word)
        );

        tiles.forEach(tile => tile.classList.add("fade-out"));

        setTimeout(() => {
            tiles.forEach(tile => tile.remove());

            requestAnimationFrame(() => {
                const rect = document.createElement("div");
                rect.classList.add("solved-rect");
                rect.style.background = color;
                rect.innerHTML = `
                    <div>${catName}</div>
                    <div class="clues">${words.join(", ")}</div>
                `;
                solved.appendChild(rect);
                requestAnimationFrame(() => rect.classList.add("show"));
            });
        }, 350);

    } else {
        mistakesRemaining--;
        updateMistakeDots();
        message.textContent = "Not a correct group. Try again!";

        document.querySelectorAll(".selected")
            .forEach(div => div.classList.remove("selected"));

        if (mistakesRemaining === 0) {
            message.textContent = "Game Over — no mistakes left!";
            gameOver = true;
        }
    }

    selected = [];

    if (foundCategories.length === Object.keys(categories).length) {
        message.textContent = "You found all categories!";
        gameOver = true;
    }
}

renderGrid();
</script>

</body>
</html>
