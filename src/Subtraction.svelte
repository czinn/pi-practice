<script lang="ts">
let difficulty = 10;
let forward = false;
let check_digits = true;
let auto_complete = true;
let timer_duration = 100;

let minuend = [];
let subtrahend = [];
let actual_answer = [];
let answer = [];
let answer_index = 0;

let total_correct = 0;

function handle_forward(forward) {
  for (let i = 0; i < minuend.length; i++) {
    answer[i] = null;
  }
  answer_index = forward ? 0 : answer.length - 1;
}

$: handle_forward(forward);

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

function compare_numbers(a, b) {
  for (let i = 0; i < a.length; i++) {
    if (a[i] < b[i]) {
      return -1;
    } else if (b[i] < a[i]) {
      return 1;
    }
  }
  return 0;
}

function subtract(a, b) {
  let result = [];
  let carry = 0;
  for (let i = a.length; i >= 0; i--) {
    let value = a[i] - b[i] + carry;
    if (value < 0) {
      carry = -1;
      value += 10;
    } else {
      carry = 0;
    }
    result.unshift(value);
  }
  return result;
}

function generate() {
  minuend = random_number(difficulty);
  subtrahend = random_number(difficulty);
  if (compare_numbers(minuend, subtrahend) < 0) {
    [minuend, subtrahend] = [subtrahend, minuend];
  }

  answer = [];
  for (let i = 0; i < minuend.length; i++) {
    answer.push(null);
  }
  actual_answer = subtract(minuend, subtrahend);
  if (forward) {
    answer_index = 0;
  } else {
    answer_index = answer.length - 1;
  }
}

$: generate(difficulty);

function check_answer() {
  let correct = true;
  for (let i = 0; i < answer.length; i++) {
    if (answer[i] !== actual_answer[i] && !(actual_answer[i] === 0 && answer[i] === null)) {
      correct = false;
      break;
    }
  }
  if (correct) {
    total_correct++;
    generate();
  }
}

function enter_digit(digit) {
  if (answer_index < 0 || answer_index >= answer.length) {
    return;
  }
  answer[answer_index] = digit;
  if (forward) {
    answer_index++;
  } else {
    answer_index--;
  }
  if (auto_complete) {
    check_answer();
  }
}

function do_backspace() {
  if (forward) {
    if (answer_index === 0) {
      return;
    }
    answer_index--;
  } else {
    if (answer_index === answer.length - 1) {
      return;
    }
    answer_index++;
  }
  answer[answer_index] = null;
}

function handle_input(e) {
  if (e.key[0] >= '0' && e.key[0] <= '9') {
    enter_digit(e.key[0] - '0');
  } else if (e.key === 'Backspace') {
    do_backspace();
  } else if (e.key === 'Enter') {
    check_answer();
  }
  e.preventDefault();
  e.stopPropagation();
}

function reset_total() {
  total_solved = 0;
  generate();
}


</script>

<h1>Subtraction</h1>

<h3>Complete: {total_correct}</h3>

<table class="digits">
  <tr>
    <td></td>
    {#each minuend as d}
      <td>{d}</td>
    {/each}
  </tr>
  <tr>
    <td>-</td>
    {#each subtrahend as d}
      <td>{d}</td>
    {/each}
  </tr>
  <tr>
    <td></td>
    {#each answer as d, i}
      <td class={d === actual_answer[i] && check_digits ? 'correct' : (d !== null && check_digits ? 'incorrect' : i === answer_index ? 'next' : '')}>
        {#if d !== null}
          {d}
        {/if}
      </td>
    {/each}
  </tr>
</table>

<input on:keydown={handle_input}/>

<p class="instructions">
  Instructions: perform the subtraction. Your cursor must be in the input box to enter digits. If "Check digits" is enabled, each digit will be highlighted in green or red if it is correct or incorrect. If "Auto-complete" is turned off, press enter once you have finished; if your answer is correct, it will be replaced by a new problem.
</p>

<div class="config">
  <table>
    <tr>
      <td><label for="length">Length</label></td>
      <td><input id="length" bind:value={difficulty}/></td>
    </tr>
    <tr>
      <td><label for="check-digits">Check digits</label></td>
      <td><input id="check-digits" type="checkbox" bind:checked={check_digits}/></td>
    </tr>
    <tr>
      <td><label for="auto-complete">Auto-complete</label></td>
      <td><input id="check-digits" type="checkbox" bind:checked={auto_complete}/></td>
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
.correct {
  color: green;
}
.incorrect {
  color: red;
}
.next {
  background-color: #ffff0088;
}

.digits {
  font-size: 30px;
  margin: 10px;
}

.digits td {
  border-radius: 10px;
  width: 60px;
  height: 60px;
}
</style>
