<script lang="ts">
let difficulty = 20;

let modulus = 9;

let total_solved = 0;
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

function check_answer(answer) {
  let actual_answer = checksum(number, modulus);
  if (answer % modulus === actual_answer) {
    total_solved++;
    generate();
  } else {
    last_wrong = answer;
  }
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

function reset_total() {
  total_solved = 0;
  generate();
}

</script>

<h1>Checksums</h1>

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
  <h2 class="wrong">{last_wrong} is incorrect</h2>
{/if}

<p class="instructions">
  Instructions: find the remainder after dividing by {modulus} and enter in the box. {#if modulus === 11}If the remainder is 10, enter 't'.{/if}
</p>

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
    <button on:click={reset_total}>Reset total</button>
  </div>
</div>

<style>
.instructions {
  margin: 20px auto;
  max-width: 500px;
}
.digits {
  font-size: 30px;
}

.digits td {
  border-radius: 10px;
  width: 60px;
  height: 60px;
}

.wrong {
  color: red;
}
</style>
