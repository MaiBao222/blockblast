<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<title>Block Blast</title>
<style>
    body {
        font-family: Arial, sans-serif;
        background: #1e1e1e;
        color: white;
        text-align: center;
    }

    h1 {
        margin-top: 10px;
    }

    #game {
        display: flex;
        justify-content: center;
        gap: 30px;
        margin-top: 20px;
    }

    #board {
        display: grid;
        grid-template-columns: repeat(10, 40px);
        grid-template-rows: repeat(10, 40px);
        gap: 2px;
        background: #333;
        padding: 5px;
    }

    .cell {
        width: 40px;
        height: 40px;
        background: #222;
        border-radius: 4px;
    }

    .filled {
        background: #4caf50;
    }

    #blocks {
        display: flex;
        flex-direction: column;
        gap: 20px;
    }

    .block {
        display: grid;
        gap: 2px;
        cursor: grab;
    }

    .block div {
        width: 35px;
        height: 35px;
        background: #ff9800;
        border-radius: 4px;
    }

    #score {
        font-size: 20px;
        margin-top: 10px;
    }
</style>
</head>
<body>

<h1>🎮 Block Blast</h1>
<div id="score">Score: 0</div>

<div id="game">
    <div id="board"></div>
    <div id="blocks"></div>
</div>

<script>
const boardSize = 10;
let score = 0;

const board = document.getElementById("board");
const blocksDiv = document.getElementById("blocks");
const scoreDiv = document.getElementById("score");

let grid = Array(boardSize * boardSize).fill(0);

// ==== CREATE BOARD ====
for (let i = 0; i < boardSize * boardSize; i++) {
    const cell = document.createElement("div");
    cell.classList.add("cell");
    cell.dataset.index = i;
    board.appendChild(cell);
}

// ==== BLOCK SHAPES ====
const shapes = [
    [[1,1,1]],
    [[1],[1],[1]],
    [[1,1],[1,1]],
    [[1,1,1],[0,1,0]],
    [[1,1,0],[0,1,1]]
];

// ==== CREATE RANDOM BLOCKS ====
function generateBlocks() {
    blocksDiv.innerHTML = "";
    for (let i = 0; i < 3; i++) {
        const shape = shapes[Math.floor(Math.random() * shapes.length)];
        const block = document.createElement("div");
        block.classList.add("block");
        block.draggable = true;

        block.style.gridTemplateColumns = `repeat(${shape[0].length}, 35px)`;

        block.shape = shape;

        shape.forEach(row => {
            row.forEach(cell => {
                const d = document.createElement("div");
                if (!cell) d.style.visibility = "hidden";
                block.appendChild(d);
            });
        });

        block.addEventListener("dragstart", dragStart);
        blocksDiv.appendChild(block);
    }
}

let draggedBlock = null;

function dragStart(e) {
    draggedBlock = e.target;
}

// ==== DROP LOGIC ====
board.addEventListener("dragover", e => e.preventDefault());

board.addEventListener("drop", e => {
    if (!draggedBlock) return;
    const index = e.target.dataset.index;
    if (index === undefined) return;

    const row = Math.floor(index / boardSize);
    const col = index % boardSize;

    if (canPlace(draggedBlock.shape, row, col)) {
        placeBlock(draggedBlock.shape, row, col);
        draggedBlock.remove();
        checkLines();
        updateBoard();
        updateScore(10);
        if (blocksDiv.children.length === 0) {
            generateBlocks();
        }
    }
});

// ==== CHECK PLACE ====
function canPlace(shape, row, col) {
    for (let r = 0; r < shape.length; r++) {
        for (let c = 0; c < shape[r].length; c++) {
            if (shape[r][c]) {
                const nr = row + r;
                const nc = col + c;
                if (nr >= boardSize || nc >= boardSize) return false;
                if (grid[nr * boardSize + nc]) return false;
            }
        }
    }
    return true;
}

// ==== PLACE BLOCK ====
function placeBlock(shape, row, col) {
    for (let r = 0; r < shape.length; r++) {
        for (let c = 0; c < shape[r].length; c++) {
            if (shape[r][c]) {
                grid[(row + r) * boardSize + (col + c)] = 1;
            }
        }
    }
}

// ==== CHECK FULL LINES ====
function checkLines() {
    // rows
    for (let r = 0; r < boardSize; r++) {
        let full = true;
        for (let c = 0; c < boardSize; c++) {
            if (!grid[r * boardSize + c]) full = false;
        }
        if (full) {
            for (let c = 0; c < boardSize; c++) {
                grid[r * boardSize + c] = 0;
            }
            updateScore(100);
        }
    }

    // columns
    for (let c = 0; c < boardSize; c++) {
        let full = true;
        for (let r = 0; r < boardSize; r++) {
            if (!grid[r * boardSize + c]) full = false;
        }
        if (full) {
            for (let r = 0; r < boardSize; r++) {
                grid[r * boardSize + c] = 0;
            }
            updateScore(100);
        }
    }
}

// ==== UPDATE BOARD UI ====
function updateBoard() {
    document.querySelectorAll(".cell").forEach((cell, i) => {
        cell.classList.toggle("filled", grid[i]);
    });
}

function updateScore(val) {
    score += val;
    scoreDiv.textContent = "Score: " + score;
}

// INIT
generateBlocks();
updateBoard();
</script>

</body>
</html>
