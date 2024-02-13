<script lang="ts">
let difficulty = 20;

let modulus = 9;

let timed_challenge = false;
let timed_target = 10;
let start_time = null;
let now = null;
let now_interval = null;
let total_solved = 0;
let total_time = null;
let last_wrong = null;

let number = [];

function random_digit() {
  return Math.floor(Math.random() * 10);
}

function random_number(n) {
  let number = [];
  for (let i = 0; i < n; i++) {
    number.push(random_digit());
  }
  return number;
}

function checksum(number, modulus) {
  let result = 0;
  for (let i = 0; i < number.length; i++) {
    let digit = number[number.length - i - 1];
    if (modulus === 9 || i % 2 == 0) {
      result += digit;
    } else {
      result -= digit;
    }
  }
  result += modulus * number.length;
  result %= modulus;
  return result;
}

function generate() {
  last_wrong = null;
  number = random_number(difficulty);
}

$: generate(difficulty);

function maybe_stop_timer() {
  if (total_solved >= timed_target) {
    total_time = Date.now() - start_time;
    timed_challenge = false;
  }
}

function check_answer(answer) {
  let actual_answer = checksum(number, modulus);
  if (answer === actual_answer) {
    total_solved++;
    maybe_stop_timer();
    generate();
  } else {
    last_wrong = answer;
  }
}

function update_now() {
  now = Date.now();
}

function handle_input(e) {
  if (e.key[0] >= '0' && e.key[0] <= '9') {
    check_answer(e.key[0] - '0');
  } else if (e.key === 't') {
    check_answer(10);
  }
  e.preventDefault();
  e.stopPropagation();
}

function start_timer() {
  timed_challenge = true;
  start_time = Date.now();
  total_solved = 0;
  total_time = null;
  generate();
  now = Date.now();
  if (now_interval === null) {
    now_interval = setInterval(update_now, 100);
  }
}

</script>

<h1>Checksums</h1>

<div class="config">
  <table>
    <tr>
      <td><label for="length">Length</label></td>
      <td><input id="length" bind:value={difficulty}/></td>
    </tr>
    <tr>
      <td><label for="modulus">Check digits</label></td>
      <td>
        <select id="modulus" bind:value={modulus}>
          <option value={9}>9</option>
          <option value={11}>11</option>
        </select>
      </td>
    </tr>
  </table>

  <div>
    <label for="timed-target">Target number checked</label>
    <input id="timed-target" type="number" bind:value={timed_target}/><br>
    <button on:click={start_timer}>Start timed challenge</button>
  </div>
</div>

{#if timed_challenge}
  <h3>Time: {#if start_time !== null}{Math.max(0, Math.ceil((now - start_time) / 1000))}{/if}s</h3>
{:else if total_time !== null}
  <h3>Completed {total_solved} in {Math.round(total_time / 1000)}s; {Math.round(total_time / 100 / total_solved) / 10}s per checksum</h3>
{/if}

<h3>Complete: {total_solved}</h3>

<table class="digits">
  <tr>
    <td></td>
    {#each number as d}
      <td>{d}</td>
    {/each}
  </tr>
</table>

<input on:keydown={handle_input}/>

{#if last_wrong !== null}
  <h2 class="wrong">{last_wrong}</h2>
{/if}

<style>
.digits {
  font-size: 48px;
}

.digits td {
  border-radius: 10px;
  width: 75px;
  height: 75px;
}

.config {
  position: absolute;
  left: 10px;
  top: 10px;
}

.wrong {
  color: red;
}
</style>
