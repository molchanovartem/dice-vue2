<template>
  <div id="diceRoller"></div>

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

onMounted(() => {
      // if (window.main && typeof window.main.init === 'function') {
      //   window.main.init();
      // } else {
      //   console.error('main is not defined or main.init is not a function');
      // }


      window.onkeydown = function(e) {
    //console.log(e.code);
    if(e.code === "Enter" || e.code === "Escape") {
        main.setInput(); //closes numPad
    }
}

 var main = (function() {
    var that = {}; 
    var elem = {}; 
    var vars = {
        numpadShowing: false,
        lastVal: '',
        userTyping: false
    }
    var box = null;

    that.init = function() {
        elem.container = $t.id('diceRoller');
        elem.result = $t.id('result');
        elem.textInput = $t.id('textInput'); 
        elem.numPad = $t.id('numPad');
        elem.instructions = $t.id('instructions');
        elem.center_div = $t.id('center_div');
        elem.diceLimit = $t.id('diceLimit');

        box = new DICE.dice_box(elem.container);
        box.bind_swipe(elem.center_div, before_roll, after_roll);

        $t.bind(elem.textInput, 'change', function(ev) { //shows instructions
            show_instructions(); 
        }); 
        $t.bind(elem.textInput, 'input', function(ev) { 
            let size = elem.textInput.value.length;
            elem.textInput.size = size > 0 ? size : 1;
            box.setDice(textInput.value);
        });
        $t.bind(elem.textInput, 'focus', function(ev) {
            elem.diceLimit.style.display = 'none';
            //ev.preventDefault();
            if(!vars.numpadShowing) {
                show_instructions(false);
                show_numPad(true);
            } else if(vars.userTyping) {
                _handleInput();
                vars.userTyping = false;
            }
        });
        $t.bind(elem.textInput, 'blur', function(ev) {
            //necessary to do this here for iOS compatibility
            //because they put cursor back to zero on blur
            vars.caretPos = elem.textInput.selectionStart;
            vars.selectionEnd = elem.textInput.selectionEnd;
        });
        $t.bind(elem.textInput, 'mouseup', function(ev) {
            ev.preventDefault();
        });

        box.setDice(textInput.value);
        //box.start_throw(); //start by throwing all the dice on the table

        show_instructions(true);
    }

    that.setInput = function() {
        let inputVal = elem.textInput.value;
        //check for d100 and add tens place die
        if(inputVal.includes('d100')) {
            let dIdx = inputVal.indexOf('d100');
            let numD100 = '';
            for(let i = dIdx - 1; i >= 0; i--) {
                let digit = inputVal[i];
                if(!isNaN(digit)) {
                    numD100 = digit + numD100;
                } else {
                    break;
                }                
            }
            if(numD100 === '') numD100 = '1';
            //console.log('num d100s: ' + numD100);
            for(let i = 0; i < numD100; i++) {
                inputVal += '+d9';
            }
        }
        //check for too many dice
        let d = DICE.parse_notation(inputVal);
        let numDice = d.set.length;
        if(numDice > 20) {
            elem.diceLimit.style.display = 'block';
        } else {
            box.setDice(inputVal);
            show_numPad(false);
            show_instructions(true);
        }
    }

    that.clearInput = function() {
        elem.textInput.value = '';
    }

    //called from numPad onclicks
    that.input = function(value) {
        vars.lastVal = value;
        vars.userTyping = true;
        elem.textInput.focus();
    }

    function _handleInput() {
        let text = elem.textInput.value;
        let selectedText = (vars.caretPos === vars.selectionEnd) ? false : true;
        if(vars.lastVal === "del") {
            if(selectedText) {
                deleteText();
            } else {
                text = text.substring(0, vars.caretPos) + text.substring(vars.caretPos+1, text.length);
            }
        } else if(vars.lastVal === "bksp") {
            if(selectedText) {
                deleteText();
            } else {
                text = text.substring(0, vars.caretPos-1) + text.substring(vars.caretPos, text.length);
                vars.caretPos--;
            }
        } else {
            deleteText();
            text = text.substring(0, vars.caretPos) + vars.lastVal + text.substring(vars.caretPos, text.length);
            vars.caretPos++;
        }
        elem.textInput.value = text;
        setTimeout(() => {
            elem.textInput.setSelectionRange(vars.caretPos, vars.caretPos);
        }, 1);

        function deleteText() {
            text = text.substring(0, vars.caretPos) + text.substring(vars.selectionEnd, text.length);
            setTimeout(() => {
                elem.textInput.setSelectionRange(vars.caretPos, vars.caretPos);
            }, 1);
        }
    }

    // show 'Roll Dice' swipe instructions
    // param show = bool
    function show_instructions(show) {
        if(show) {
            elem.instructions.style.display = 'inline-block';
        } else {
            elem.instructions.style.display = 'none';
        }
    }

    // show input options
    // param show = bool
    function show_numPad(show) {
        if(show) {
            vars.numpadShowing = true;
            elem.numPad.style.display = 'inline-block';
            elem.textInput.focus();
        } else {
            vars.numpadShowing = false;
            elem.textInput.blur();
            elem.numPad.style.display = 'none';
        }
    }

    // @brief callback function called when dice roll event starts
    // @param notation indicates which dice are going to roll
    // @return null for random result || array of desired results
    function before_roll(notation) {
        //console.log('before_roll notation: ' + JSON.stringify(notation));
        show_instructions(false);
        elem.result.innerHTML = '';       
        return null;
    }

    // @brief callback function called once dice stop moving
    // @param notation now includes results
    function after_roll(notation) {
        //console.log('after_roll notation: ' + JSON.stringify(notation));
        if(notation.result[0] < 0) {
            elem.result.innerHTML = "Oops, your dice fell off the table. <br> Refresh and roll again."
        } else {
            elem.result.innerHTML = notation.resultString;
        }
    }

    return that;
}());

main.init()
    });
</script>

<style lang="scss">
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
  border-radius: 4px;
  color: rgba(0, 0, 0, 0.8);
  background-color: rgba(199, 199, 199, 0.7);
  width: 500px;
  text-overflow: ellipsis;
  padding: 0.2em;
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
