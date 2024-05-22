<template>
  <div id="diceRoller"></div>

  <Link class="artemlink" text="created by Molchanov" href="https://t.me/var_molchanov" />
  <main id="diceRollerUI">
    <div class="top_field">
      <input
        id="textInput"
        type="text"
        spellcheck="false"
        inputmode="none"
        virtualkeyboardpolicy="manual"
        value="1d4+1d6+1d8+1d10+1d12+1d20"
      />
    </div>
    <div id="diceLimit" style="display: none">
      Wow that's a lot of dice! <br />
      [Limit: 20]
    </div>
    <div id="center_div" class="center_field">
      <!-- swipe bound to this -->
      <div id="instructions" style="display: none">
        <p>Swipe to roll dice</p>
      </div>
    </div>
    <div id="numPad" class="center_field" style="display: none">
      <table class="numPad">
        <tr>
          <td onclick="main.input('del')" colspan="2">del</td>
          <td onclick="main.input('bksp')" colspan="2">bksp</td>
        </tr>
        <tr>
          <td onclick="main.input('7')">7</td>
          <td onclick="main.input('8')">8</td>
          <td onclick="main.input('9')">9</td>
          <td onclick="main.input('+')" rowspan="2">+</td>
        </tr>
        <tr>
          <td onclick="main.input('4')">4</td>
          <td onclick="main.input('5')">5</td>
          <td onclick="main.input('6')">6</td>
        </tr>
        <tr>
          <td onclick="main.input('1')">1</td>
          <td onclick="main.input('2')">2</td>
          <td onclick="main.input('3')">3</td>
          <td onclick="main.input('-')" rowspan="2">-</td>
        </tr>
        <tr>
          <td onclick="main.input('0')" colspan="2">0</td>
          <td onclick="main.input('d')">d</td>
        </tr>
      </table>
      <button onclick="main.clearInput()">CLEAR</button>
      <button onclick="main.setInput()">OK</button>
    </div>
    <div class="bottom_field">
      <span id="result"></span>
    </div>
  </main>

</template>

<script setup>
import { onMounted } from 'vue';
import { Link } from '@tok/ui/components/Link';

onMounted(() => {
      if (window.main && typeof window.main.init === 'function') {
        window.main.init();
      } else {
        console.error('main is not defined or main.init is not a function');
      }
    });
</script>

<style lang="scss">
.artemlink {
  color: white;
  display: flex;
  flex-direction: row-reverse;
}
* {
  box-sizing: border-box; /* padding and border are included in total width/height of all elements */
  margin: 0px;
  font-size: 14pt;
  font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande',
    'Lucida Sans', Arial, sans-serif;
}

body {
  margin: 0;
  overflow-x: hidden;
  background: #202020;
  /* background-image: url("assets/background.svg"); */
  background-repeat: no-repeat;
  background-position: right bottom;
  background-size: cover;
  height: 100vh;
  width: 100vw;
}

header {
  text-align: center;
  color: white;
  padding: 5px;
}

#helpBtn {
  position: absolute;
  top: 5px;
  right: 5px;
  width: 1.2rem;
  height: 1.2rem;
  border: none;
  border-radius: 0.6rem;
}

/* ******* DICE ROLLER *********** */

#diceRoller {
  position: absolute;
  top: 0;
  height: calc(100% - 4.5rem);
  width: 100%;
  margin-top: 1.5rem;
  z-index: -1;
  overflow: hidden;
}

#canvas {
  width: 100%;
  height: 100%;
}

/* ************* UI ************* */

#diceRollerUI {
  width: 100vw;
  height: 100vh;
  margin: 0;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

#diceLimit {
  text-align: center;
  color: red;
  background-color: #050505;
  padding: 0.5rem;
  font-size: 10pt;
  font-weight: bold;
}

.top_field {
  text-align: center;
  width: 100%;
  background-color: rgba(150, 150, 150, 0.1);
}

#textInput {
  text-align: center;
  border: none;
  border-radius: 8px;
  color: rgba(0, 0, 0, 0.8);
  background-color: rgba(199, 199, 199, 0.7);
  width: calc(100% - 100px);
  text-overflow: ellipsis;
  padding: 0.2em;
  margin-top: 10px;
  margin-right: 30px;
  margin-left: 30px;
  text-decoration: none;
}

#textInput:focus {
  background-color: rgba(255, 255, 255, 0.7);
  outline: none;
}

.center_field {
  position: relative;
  flex: 1;
  text-align: center;
  height: 80%;
  width: 100%;
  z-index: 100;
}

.center_field * {
  position: relative;
  background-color: rgba(255, 255, 255, 0.6);
  padding: 5px 15px;
  user-select: none;
}

#instructions {
  background: none;
  height: 100%;
  width: 100%;
}

#instructions p {
  color: rgb(255, 255, 255);
  background: none;
  margin: auto;
  top: 35%;
  padding: 1em;
}

#numPad {
  /* container */
  position: absolute;
  width: 200px;
  height: auto;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.numPad {
  /* table */
  width: 200px;
  margin: auto;
  padding-top: 15px;
  padding-bottom: 15px;
}

.numPad td {
  cursor: pointer;
}

#numPad button {
  padding: 0.5em;
  margin-top: 5px;
  width: 97px;
  border: none;
  background-color: rgba(255, 255, 255, 0.8);
  color: black;
}

.bottom_field {
  position: absolute;
  bottom: 0;
  text-align: center;
  min-height: 3rem;
  background: rgba(0, 0, 0, 0.4);
  width: inherit;
  padding: 0px;
}

#result {
  position: relative;
  display: block;
  min-height: 1.5em;
  bottom: 5px;
  word-spacing: 0.5em;
  color: rgba(255, 255, 255, 0.9);
  padding: 0.5em;
  margin-left: 5px;
  margin-right: 5px;
}
</style>
