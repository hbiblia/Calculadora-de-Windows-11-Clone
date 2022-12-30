<template>
  <h3>Calculadora de Windows 11 Clone</h3>
  <div class="cal-body">
    <div class="title">
      <label for=""><img :src="icon" width="16"/>&nbsp;Calculadora básica</label>
      <div class="windows-button"></div>
    </div>
    <div class="toolbar"></div>
    <div class="input-operacion-completa" v-show="(operadorActive != '' || isEqualEnterPress) && bShowSmallHistory">
      <span>{{ numberFirst }}</span>
      <span>{{ operadorActive }}</span>
      <span v-show="isEqualEnterPress"> {{ numberSecond }} = </span>
    </div>
    <div class="input-number">{{ formatResultNumber() }}</div>
    <div class="button-content">
      <button v-for="(btn, i) in map_buttons" :key="i" @click="OnClickButton(btn)" class="cal-btn"
        :class="{ equal: btn == '=' }">{{ btn }}</button>
    </div>
  </div>
</template>

<script setup>

import { ref } from 'vue';
import icon from './assets/calculadora.png'

const map_buttons = 
 ['%', 'CE', 'C', '⌫',
  ' ', ' ', ' ', '÷',
  '7', '8', '9', '✕',
  '4', '5', '6', '-',
  '1', '2', '3', '+',
  ' ', '0', '.', '=']
const operadoresTrans = { '÷': '/', '✕': '*', '-': '-', '+': '+', '%': '%' }

let numberResult = ref(0)
let numberFirst = ref(0)
let numberSecond = ref(0)
let operadorActive = ref('')
let isEqualEnterPress = ref(false)
let numberJumpPressOperator = ref(false)
let bNowCanUseBackspace = ref(false)
let bShowSmallHistory = ref(false)

function formatResultNumber()
{
  const value = parseFloat(numberResult.value)
  const result = value.toLocaleString('en-US',{ maximumSignificantDigits: 16 })
  return result
}

function operatingTheNumbers(value) {
  const operadores = Object.keys(operadoresTrans)
  // cuando press un operador registrado en 'const operadores'
  if (operadores.findIndex(o => o == value) != -1) {
    this.OnOperatorClicked(value)
  } else {
    this.OnNumberClicked(value)
  }
}

function resetAllNumber() {
  numberFirst.value = 0
  numberSecond.value = 0
  operadorActive.value = ''
  isEqualEnterPress.value = false
  numberJumpPressOperator.value = false
  this.resetNumberResult()
}

function resetNumberResult() {
  numberResult.value = 0
}

function eval_fake(n1, o, n2) {
  let operation = operadoresTrans[o]
  const number1 = parseFloat(n1)
  const number2 = parseFloat(n2)
  let result = 0

  if (operation == '/')result = number1 / number2
  if (operation == '+')result = number1 + number2
  if (operation == '-')result = number1 - number2
  if (operation == '*')result = number1 * number2

  return result
}

function showSmallHistory(active)
{
  return bShowSmallHistory.value = active
}

// Se activa cuando se presiona cualquier boton
function OnClickButton(value) {
  if (numberResult.value == '0') {
    numberResult.value = ''
  }

  switch (value) {
    case '%': this.OnPercentageResult(); break;
    case '⌫': this.OnBackSpace(); break;
    case '=': this.OnButtonEqualClicked(); break;
    case 'C': this.resetAllNumber(); break;
    case 'CE': {
      if (isEqualEnterPress.value) {
        return this.resetAllNumber();
      }
      this.resetNumberResult()
    } break;
    default: this.operatingTheNumbers(value)
  }
}

function OnPercentageResult() {
  const result = this.eval_fake(numberResult.value, '÷', 100)
  numberResult.value = result
  numberFirst.value = result
  numberSecond.value = ''
  operadorActive.value = ''
}

function OnBackSpace() {
  if (!bNowCanUseBackspace.value) return;

  const equalPress = isEqualEnterPress.value

  this.showSmallHistory(false)

  if (!equalPress) {
    numberResult.value = numberResult.value.slice(0, numberResult.value.length - 1)
  }

  if (numberJumpPressOperator.value) {
    numberSecond.value = parseFloat(numberResult.value)
  } else {
    numberFirst.value = parseFloat(numberResult.value)
  }
}

// Se activa cuando se presiona el boton (=)
function OnButtonEqualClicked() {

  if (isEqualEnterPress.value) {
    numberFirst.value = numberResult.value
  }

  if (numberSecond.value == 0) {
    numberSecond.value = numberFirst.value
  }

  if (operadorActive.value != '') {
    numberResult.value = this.eval_fake(numberFirst.value, operadorActive.value, numberSecond.value)
  }

  isEqualEnterPress.value = true
  numberJumpPressOperator.value = false
  this.showSmallHistory(true)
}

// Se activa cuando se presiona un operador (÷ X - + %) registrado
function OnOperatorClicked(operator) {
  operadorActive.value = operator
  numberJumpPressOperator.value = true
  bNowCanUseBackspace.value = false
  this.showSmallHistory(true)

  if (isEqualEnterPress.value) {
    numberFirst.value = numberResult.value
    numberSecond.value = 0
    isEqualEnterPress.value = false
  }
}

// Se activa cuando se presiona un numero
function OnNumberClicked(number) {
  if (numberJumpPressOperator.value) {

    if (numberSecond.value == 0) {
      numberResult.value = ''
    }

    numberSecond.value += number
    numberSecond.value = parseFloat(numberSecond.value)
  } else {
    numberFirst.value += number
    numberFirst.value = parseFloat(numberFirst.value)
  }

  bNowCanUseBackspace.value = true
  numberResult.value += number
}

</script>

<style>
html,
body {
  margin: 0;
  padding: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  background-color: #2c3e50;
  width: 100%;
  height: 100%;
  position: absolute;
}

h3 {
  color: white;
}

.cal-body {
  width: 400px;
  height: 620px;
  background-color: #eff4f9;
  border-radius: 10px;
  margin: auto;
  box-shadow: 0px 0px 30px rgba(0, 0, 0, 0.127);
  border: 1px solid rgba(0, 0, 0, 0.18);

  display: flex;
  flex-wrap: wrap;
  align-items: flex-end;
  align-content: flex-end;

  position: relative;
}

.title {
  position: absolute;
  top: 5px;
  left: 5px;

  font-size: 15px;
}

.input-operacion-completa {
  width: 100%;
  text-align: end;
  display: flex;
  align-items: center;
  justify-content: end;
  font-size: 18px;
  color: gray;
  margin-right: 10px;
}

.input-operacion-completa span {
  margin-left: 5px;
  font-family: monospace;
}

.input-number {
  width: 100%;
  height: 100px;
  font-size: 38px;
  font-weight: bold;
  text-align: right;
  padding: 10px;

  border: 2px solid transparent;
  border-radius: 5px;

  display: flex;
  justify-content: end;
  align-content: center;
  align-items: center;

  overflow: hidden;
}

.cal-btn {
  border-radius: 5px;
  width: 98px;
  height: 62px;
  margin: 1px;
  font-size: 20px;

  background-color: #f7f9fc;
  box-shadow: 0px 0px 2px rgba(0, 0, 0, 0.064);
  border: 1px solid rgba(0, 0, 0, 0.18);
}

.cal-btn:hover {
  background-color: #f4f6f9;
  cursor: pointer;
}

.cal-btn:active {
  background-color: #e8eaef;
}

.cal-btn.equal {
  background-color: #0067c0;
  color: #e8eaef;
}

.cal-btn.equal:hover {
  background-color: #006bc9;
  color: #e8eaef;
}

.cal-btn.equal:active {
  background-color: #0163b9;
  color: #e8eaef;
}

.button-content {
  display: flex;
  flex-wrap: wrap;
}
</style>
