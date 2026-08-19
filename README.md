# code
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Random Rectangle Splitting</title>

<style>
    body {
        margin: 0;
        background: #111;
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
        overflow: hidden;
        font-family: Arial, sans-serif;
    }

    canvas {
        background: white;
        border: 2px solid #444;
    }

    .info {
        position: fixed;
        top: 15px;
        left: 15px;
        color: white;
        font-size: 20px;
    }
</style>
</head>
<body>

<div class="info">
    Pieces: <span id="count">1</span> / 50
</div>

<canvas id="canvas" width="900" height="600"></canvas>

<script>
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");
const countEl = document.getElementById("count");

let pieces = [
    {
        x: 50,
        y: 50,
        w: 800,
        h: 500,
        color: randomColor()
    }
];

function randomColor() {
    const h = Math.random() * 360;
    return `hsl(${h},70%,60%)`;
}

function draw(cutLine = null) {
    ctx.clearRect(0,0,canvas.width,canvas.height);

    pieces.forEach(piece => {
        ctx.fillStyle = piece.color;
        ctx.fillRect(piece.x, piece.y, piece.w, piece.h);

        ctx.strokeStyle = "#000";
        ctx.lineWidth = 2;
        ctx.strokeRect(piece.x, piece.y, piece.w, piece.h);
    });

    if (cutLine) {
        ctx.strokeStyle = "red";
        ctx.lineWidth = 4;
        ctx.beginPath();
        ctx.moveTo(cutLine.x1, cutLine.y1);
        ctx.lineTo(cutLine.x2, cutLine.y2);
        ctx.stroke();
    }

    countEl.textContent = pieces.length;
}

function largestPieceIndex() {
    let index = 0;
    let maxArea = 0;

    pieces.forEach((p, i) => {
        const area = p.w * p.h;
        if (area > maxArea) {
            maxArea = area;
            index = i;
        }
    });

    return index;
}

function animateCut(piece, vertical, cutPos) {
    return new Promise(resolve => {
        let progress = 0;

        function frame() {
            draw();

            ctx.strokeStyle = "red";
            ctx.lineWidth = 4;

            ctx.beginPath();

            if (vertical) {
                const x = piece.x + cutPos;
                ctx.moveTo(x, piece.y);
                ctx.lineTo(x, piece.y + piece.h * progress);
            } else {
                const y = piece.y + cutPos;
                ctx.moveTo(piece.x, y);
                ctx.lineTo(piece.x + piece.w * progress, y);
            }

            ctx.stroke();

            progress += 0.05;

            if (progress < 1) {
                requestAnimationFrame(frame);
            } else {
                resolve();
            }
        }

        frame();
    });
}

async function splitLargestPiece() {

    const idx = largestPieceIndex();
    const piece = pieces[idx];

    let vertical;

    if (piece.w > piece.h * 1.2) {
        vertical = true;
    } else if (piece.h > piece.w * 1.2) {
        vertical = false;
    } else {
        vertical = Math.random() < 0.5;
    }

    if (vertical) {

        const cut = piece.w * (0.3 + Math.random() * 0.4);

        await animateCut(piece, true, cut);

        const p1 = {
            x: piece.x,
            y: piece.y,
            w: cut,
            h: piece.h,
            color: randomColor()
        };

        const p2 = {
            x: piece.x + cut,
            y: piece.y,
            w: piece.w - cut,
            h: piece.h,
            color: randomColor()
        };

        pieces.splice(idx, 1, p1, p2);

    } else {

        const cut = piece.h * (0.3 + Math.random() * 0.4);

        await animateCut(piece, false, cut);

        const p1 = {
            x: piece.x,
            y: piece.y,
            w: piece.w,
            h: cut,
            color: randomColor()
        };

        const p2 = {
            x: piece.x,
            y: piece.y + cut,
            w: piece.w,
            h: piece.h - cut,
            color: randomColor()
        };

        pieces.splice(idx, 1, p1, p2);
    }

    draw();
}

async function run() {
    draw();

    while (pieces.length < 50) {
        await splitLargestPiece();
        await new Promise(r => setTimeout(r, 250));
    }
}

run();
</script>

</body>
</html>
