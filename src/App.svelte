<script>
  import Main from "./components/Main.svelte";
  import { onMount } from "svelte";

  // config
  export let velocity = 500;
  export let fpi;
  export let cpi;
  export let rot = 0;
  export let tableMatriz;
  export let piece = 0;
  export let nextPiece = 0;
  export let record = 0;
  export let lines = 0;
  export let start;
  export let level = 0;

  export let tds = [];
  export let trs = [];

  export let pos = [];
  export let pieces = [];
  export let gamePaused = false;

  createPos();
  createpieces();
  loadGame();
  generateView();

  function createPos() {
    console.log("paso por aca");
    pos = [
      [0, 0],
      [0, 1],
      [-1, 0],
      [1, 0],
      [-1, -1],
      [0, -1],
      [1, -1],
      [0, -2]
    ];
  }

  function createpieces() {
    console.log("paso por aca 2");
    pieces = [
      [4, 0, 1, 2, 3],
      [4, 0, 1, 5, 6],
      [4, 0, 1, 5, 4],
      [2, 0, 1, 5, 7],
      [2, 0, 2, 5, 6],
      [2, 0, 3, 5, 4],
      [1, 0, 5, 6, 3]
    ];
  }

  function newGame() {
    velocity = 500;
    tableMatriz = new Array(20);
    for (let n = 0; n < 20; n++) {
      tableMatriz[n] = new Array(9);
      for (let m = 0; m < 9; m++) {
        tableMatriz[n][m] = 0;
      }
    }
    lines = 0;
    level = 0;
    start = setInterval(() => {
      tick();
    }, velocity);
    newPiece();
    setPiece();
  }

  function BoxIsNotAvailable(f, c) {
    if (f < 0) return false;
    return c < 0 || c >= 9 || f >= 20 || tableMatriz[f][c] > 0;
  }

  function collisionPiece() {
    for (let v = 1; v < 5; v++) {
      const des = pieces[piece][v];
      const pos2 = rotateBox(pos[des]);
      if (BoxIsNotAvailable(pos2[0] + fpi, pos2[1] + cpi)) {
        return true;
      }
    }
    return false;
  }

  function DetectLines() {
    for (let f = 0; f < 20; f++) {
      let count = 0;
      for (let c = 0; c < 9; c++) {
        if (tableMatriz[f][c] > 0) {
          count++;
        }
      }
      if (count === 9) {
        for (let f2 = f; f2 > 0; f2--) {
          for (let c2 = 0; c2 < 9; c2++) {
            tableMatriz[f2][c2] = tableMatriz[f2 - 1][c2];
          }
        }
        lines++;
      }
    }
  }

  function downPiece() {
    if (gamePaused) {
      return;
    }
    fpi = fpi + 1;
    if (collisionPiece()) {
      fpi = fpi - 1;
      for (let v = 1; v < 5; v++) {
        const des = pieces[piece][v];
        const pos2 = rotateBox(pos[des]);
        if (
          pos2[0] + fpi >= 0 &&
          pos2[0] + fpi < 20 &&
          pos2[1] + cpi >= 0 &&
          pos2[1] + cpi < 9
        ) {
          tableMatriz[pos2[0] + fpi][pos2[1] + cpi] = piece + 1;
        }
      }
      DetectLines();
      let reset = 0;
      for (let c = 0; c < 9; c++) {
        if (tableMatriz[0][c] !== 0) {
          reset = 1;
        }
      }
      if (reset === 1) {
        if (lines > record) {
          record = lines;
        }
        start.unsubscribe();
      } else {
        setPiece();
      }
    }
  }

  function movePiece(des) {
    if (gamePaused) {
      return;
    }
    cpi = cpi + des;
    if (collisionPiece()) {
      cpi = cpi - des;
    }
  }

  function rotatePiece() {
    if (gamePaused) {
      return;
    }
    rot = rot + 1;
    console.log(pieces);
    debugger;
    if (rot === pieces[piece][0]) {
      rot = 0;
    }
    if (collisionPiece()) {
      rot = rot - 1;
      if (rot === 1) {
        rot = pieces[piece][0] - 1;
      }
    }
  }

  function rotateBox(cell) {
    const pos2 = [cell[0], cell[1]];
    for (let n = 0; n < rot; n++) {
      const f = pos2[1];
      const c = -pos2[0];
      pos2[0] = f;
      pos2[1] = c;
    }
    return pos2;
  }

  function newPiece() {
    cpi = 3;
    fpi = 0;
    rot = 0;
    const newPiece = Math.floor(Math.random() * 7);
    setLevel();
    nextPiece = newPiece;
  }

  function setPiece() {
    piece = nextPiece;
    newPiece();
  }

  function setLevel() {
    let newLevel = false;
    switch (lines) {
      case 3:
        newLevel = true;
        level = 1;
        velocity = 450;
        break;
      case 6:
        newLevel = true;
        level = 2;
        velocity = 400;
        break;
      case 9:
        newLevel = true;
        level = 3;
        velocity = 350;
        break;
      case 12:
        newLevel = true;
        level = 4;
        velocity = 300;
        break;
      case 15:
        newLevel = true;
        level = 5;
        velocity = 250;
        break;
      case 18:
        newLevel = true;
        level = 6;
        velocity = 200;
        break;
      case 21:
        newLevel = true;
        level = 7;
        velocity = 150;
        break;
      case 24:
        newLevel = true;
        level = 8;
        velocity = 100;
        break;
      case 27:
        newLevel = true;
        level = 9;
        velocity = 50;
        break;
    }

    if (newLevel) {
      start.complete();
      start = setInterval(() => {
        tick();
      }, velocity);
    }
  }

  function pauseGame() {
    start.complete();
    gamePaused = true;
  }

  function resumeGame() {
    gamePaused = false;
    start = setInterval(() => {
      tick();
    }, velocity);
  }

  function tick() {
    downPiece();
    generateView();
  }

  function generateView() {
    let des;
    trs = [];
    for (let f = 0; f < 20; f++) {
      for (let c = 0; c < 9; c++) {
        let color = tableMatriz[f][c];
        if (color === 0) {
          for (let v = 1; v < 5; v++) {
            des = pieces[piece][v];
            const pos2 = rotateBox(pos[des]);
            if (f === fpi + pos2[0] && c === cpi + pos2[1]) {
              color = piece + 1;
            }
          }
        }
        tds.push(color);
      }
      trs.push(tds);
      tds = [];
    }
    velocity = Math.max(velocity - 1, 500);
  }

  function loadGame() {
    newGame();
  }

  function keyMove(event) {
    const e = event.target.innerHTML;
    switch (e) {
      case "izquierda":
        movePiece(-1);
        break;
      case "arriba":
        rotatePiece();
        break;
      case "derecha":
        movePiece(1);
        break;
      case "abajo":
        downPiece();
        break;
    }
    generateView();
  }
</script>

<style>
  .tetris {
    border-collapse: collapse;
  }
  .tetris tr td {
    width: 20px;
    height: 20px;
    padding: 0px;
  }

  .game {
    max-width: 50%;
    float: left;
  }

  .actions {
    float: left;
  }

  .next {
    height: 100px;
  }

  .next > img {
    margin-left: 80px;
  }

  td {
    border: 1px solid grey;
  }

  .celda0 {
    background-color: #000000;
  }
  .celda1 {
    background-color: #0000ff;
  }
  .celda2 {
    background-color: #ff0000;
  }
  .celda3 {
    background-color: #00ff00;
  }
  .celda4 {
    background-color: #ffff00;
  }
  .celda5 {
    background-color: #00ffff;
  }
  .celda6 {
    background-color: #ff00ff;
  }
  .celda7 {
    background-color: #888888;
  }

  img {
    width: 50px;
  }
  .up {
    -webkit-transform: rotate(-90deg);
    -moz-transform: rotate(-90deg);
    -ms-transform: rotate(-90deg);
    transform: rotate(-90deg);
  }

  .left {
    -webkit-transform: rotate(-180deg);
    -moz-transform: rotate(-180deg);
    -ms-transform: rotate(-180deg);
    transform: rotate(-180deg);
  }

  .down {
    -webkit-transform: rotate(90deg);
    -moz-transform: rotate(90deg);
    -ms-transform: rotate(90deg);
    transform: rotate(90deg);
  }

  .actions {
    margin-left: 30px;
    margin-top: 20px;
  }

  .up-container {
    width: 150px;
  }
  .up-container > a {
    margin-left: 50px;
  }

  .button-container {
    width: 150px;
    height: 50px;
  }

  .button-container > div {
    float: left;
  }

  .container-left,
  .container-right {
    margin-top: -25px;
  }
</style>

<Main>
  <div class="game">
    <table class="tetris">
      {#each trs as tr}
        <tr>
          {#each tr as td}
            <td class="celda{td}" />
          {/each}
        </tr>
      {/each}
    </table>
  </div>
  <div class="next">
    <img src={`/assets/${nextPiece}.png`} alt="piece" />
  </div>
  <div class="actions">
    <div class="up-container">
      <button on:click={keyMove}>arriba</button>
    </div>
    <div class="button-container">
      <button class="container-left" on:click={keyMove}>izquierda</button>
      <button on:click={keyMove}>abajo</button>
      <button class="container-right" on:click={keyMove}>derecha</button>
    </div>

    <h3>Level: {level}</h3>
    <h3>Lines: {lines}</h3>
    <h3>Record: {record}</h3>
    <button on:click={newGame}>New Game</button>
    {#if !gamePaused}
      <button on:click={pauseGame}>Pause Game</button>
    {:else}
      <button on:click={resumeGame}>Resume Game</button>
    {/if}
  </div>
</Main>
