<script>
  let globals
  let snake
  let grid
  let numbers
  let active
  let mainLoopInterval

  globals = {
    grid_size:  10,
    speed: 100
  }

  // set snake direction and mitigate stupid moves
  function setSnakeDirection(event) {
    if      (event.key == 'ArrowLeft'  && snake.direction[2] != 'r') { snake.direction = [0, -1, 'l'] }
    else if (event.key == 'ArrowRight' && snake.direction[2] != 'l') { snake.direction = [0, 1,  'r'] }
    else if (event.key == 'ArrowUp'    && snake.direction[2] != 'd') { snake.direction = [-1, 0, 'u'] }
    else if (event.key == 'ArrowDown'  && snake.direction[2] != 'u') { snake.direction = [1, 0,  'd'] }
  }

  // render single cell
  function cellClass(row, cell) {
    if (snake.path_cache[String([row, cell])]) {
      return 'cell active'
    } else if (String(snake.food) == String([row, cell])) {
      return 'cell food'
    } else {
      return 'cell empty'
    }
  }

  // render place food
  function placeFood() {
    let start = [
      parseInt(Math.random() * globals.grid_size),
      parseInt(Math.random() * globals.grid_size)
    ]

    let next

    // over complicated loop that probably has bugs, to find a safe place to place food
    // for it not to land at snake
    while (String(start) != String(next)) {
      next    = next || start.map(x=>x)
      next[0] = next[0] + 1

      if (next[0] >= globals.grid_size) {
        next[0] = 0
        next[1] += 1

        if (next[1] >= globals.grid_size) {
          next[1] = 0
        }
      }

      if (!snake.path_cache[String([next])]) {
        snake.food = next
        return
      }
    }

    initGame('YOU WIN')
  }

  // init game
  function initGame(error) {
    if (error) {
      alert(error)
    }

    active = [0, 0]
    snake = {
      direction:  [],
      path:       [active],
      length:     2,
      direction:  [0, 1],
      path_cache: {}
    }

    // precache numbers list
    numbers = []
    for (let i=0; i<globals.grid_size; i++) {
      numbers.push(i)
    }

    placeFood()

    // main update loop
    clearInterval(mainLoopInterval)
    mainLoopInterval = setInterval(snakeMainLoop, globals.speed);
  }

  // handle animation frame
  function snakeMainLoop() {
    grid = [[]]

    active = [
      active[0] + snake.direction[0],
      active[1] + snake.direction[1]
    ]

    if (globals.force_kill) {
      if (
        active[0] < 0 ||
        active[1] < 0 ||
        active[0] > globals.grid_size-1 ||
        active[1] > globals.grid_size-1
      ) {
        initGame('You run to a wall!')
      }
    } else {
      // reposition snake if target out of bounds
      if (active[0] < 0) { active[0] = globals.grid_size-1 }
      if (active[1] < 0) { active[1] = globals.grid_size-1 }
      if (active[0] >= globals.grid_size) { active[0] = 0 }
      if (active[1] >= globals.grid_size) { active[1] = 0 }
    }

    // set snake path and active path
    snake.path.push(active)

    // eat yourself, lost
    if (snake.path_cache[String([active])]) {
      initGame('You ate yourself!')
    }

    // food eaten
    if (String(active) == String(snake.food)) {
      snake.length += 1
      placeFood()
    }

    // trim snake length, if needed
    if (snake.path.length > snake.length) {
      snake.path.shift()
    }

    // precache snake path
    snake.path_cache = {}
    for (let el of snake.path) {
      snake.path_cache[String(el)] = true
    }

    // force re-render
    numbers = numbers
  }

  // if globals change, rerun initGame(), replaces hooks in react
  $: if (globals) { initGame() }

</script>

<svelte:window on:keydown={setSnakeDirection} />

<style>
 .cell {
    width: 20px;
    height: 20px;
    display: inline-block;
    margin-right: 2px;
  }

 .grid .empty {
    background: #eee;
  }

 .grid .food {
    background: #800;
  }

  .grid .snake, .grid .active {
    background: #080;
  }

  table {
    margin-bottom: 30px;
  }

  table td {
    text-align: left;
    height: 30px;
    width: 180px;
  }
</style>

<table>
  <tr>
    <td>Update in ms ({globals.speed}):</td>
    <td><input type="range" min="20" max="1000" step="10" bind:value={globals.speed} /></td>
  </tr>
  <tr>
    <td>Grid size ({globals.grid_size}):</td>
    <td><input type="range" min="5" max="50" bind:value={globals.grid_size} /></td>
  </tr>
  <tr>
    <td>Bounds kills you:</td>
    <td><input type="checkbox" bind:checked={globals.force_kill} /></td>
  </tr>
  <tr>
    <td>Snake length:</td>
    <td>{snake.length}</td>
  </tr>
</table>

<div class="grid">
  {#each numbers as row}
    <div class="row">
      {#each numbers as cell}
        <div class={cellClass(row, cell)}></div>
      {/each}
    </div>
  {/each}
</div>
<br />

