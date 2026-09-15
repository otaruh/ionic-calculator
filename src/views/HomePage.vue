<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Ionic Calculator</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <div class="calculator">

        <!-- Calculator display -->
        <div class="display">
          {{ display }}
        </div>

        <!-- Calculator buttons -->
        <ion-grid>
          <!-- Row 1 -->
          <ion-row>
            <ion-col size="3">
              <ion-button expand="block" fill="outline" @click="clear">
                C
              </ion-button>
            </ion-col>

            <ion-col size="3">
              <ion-button expand="block" fill="outline" @click="backspace">
                ⌫
              </ion-button>
            </ion-col>

            <ion-col size="3">
              <ion-button expand="block" fill="outline" @click="inputOperator('%')">
                %
              </ion-button>
            </ion-col>

            <ion-col size="3">
              <ion-button expand="block" color="warning" @click="inputOperator('/')">
                ÷
              </ion-button>
            </ion-col>
          </ion-row>

          <!-- Row 2 -->
          <ion-row>
            <ion-col size="3">
              <ion-button expand="block" @click="inputNumber('7')">
                7
              </ion-button>
            </ion-col>

            <ion-col size="3">
              <ion-button expand="block" @click="inputNumber('8')">
                8
              </ion-button>
            </ion-col>

            <ion-col size="3">
              <ion-button expand="block" @click="inputNumber('9')">
                9
              </ion-button>
            </ion-col>

            <ion-col size="3">
              <ion-button expand="block" color="warning" @click="inputOperator('*')">
                ×
              </ion-button>
            </ion-col>
          </ion-row>

          <!-- Row 3 -->
          <ion-row>
            <ion-col size="3">
              <ion-button expand="block" @click="inputNumber('4')">
                4
              </ion-button>
            </ion-col>

            <ion-col size="3">
              <ion-button expand="block" @click="inputNumber('5')">
                5
              </ion-button>
            </ion-col>

            <ion-col size="3">
              <ion-button expand="block" @click="inputNumber('6')">
                6
              </ion-button>
            </ion-col>

            <ion-col size="3">
              <ion-button expand="block" color="warning" @click="inputOperator('-')">
                −
              </ion-button>
            </ion-col>
          </ion-row>

          <!-- Row 4 -->
          <ion-row>
            <ion-col size="3">
              <ion-button expand="block" @click="inputNumber('1')">
                1
              </ion-button>
            </ion-col>

            <ion-col size="3">
              <ion-button expand="block" @click="inputNumber('2')">
                2
              </ion-button>
            </ion-col>

            <ion-col size="3">
              <ion-button expand="block" @click="inputNumber('3')">
                3
              </ion-button>
            </ion-col>

            <ion-col size="3">
              <ion-button expand="block" color="warning" @click="inputOperator('+')">
                +
              </ion-button>
            </ion-col>
          </ion-row>

          <!-- Row 5 -->
          <ion-row>
            <ion-col size="6">
              <ion-button expand="block" @click="inputNumber('0')">
                0
              </ion-button>
            </ion-col>

            <ion-col size="3">
              <ion-button expand="block" @click="inputDecimal">
                .
              </ion-button>
            </ion-col>

            <ion-col size="3">
              <ion-button expand="block" color="success" @click="calculate">
                =
              </ion-button>
            </ion-col>
          </ion-row>
        </ion-grid>

      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonGrid,
  IonRow,
  IonCol,
  IonButton
} from '@ionic/vue';

import { ref } from 'vue';

// What appears on the calculator screen
const display = ref('0');

// Stores the first number
const firstNumber = ref<number | null>(null);

// Stores the selected operation
const operator = ref<string | null>(null);

// Tells the calculator if we are entering a new number
const waitingForSecondNumber = ref(false);


// Clear calculator
function clear() {
  display.value = '0';
  firstNumber.value = null;
  operator.value = null;
  waitingForSecondNumber.value = false;
}


// Enter a number
function inputNumber(number: string) {
  if (display.value === 'Error') {
    clear();
  }

  if (waitingForSecondNumber.value) {
    display.value = number;
    waitingForSecondNumber.value = false;
  } else {
    if (display.value === '0') {
      display.value = number;
    } else {
      display.value += number;
    }
  }
}


// Enter decimal point
function inputDecimal() {
  if (display.value === 'Error') {
    clear();
  }

  if (waitingForSecondNumber.value) {
    display.value = '0.';
    waitingForSecondNumber.value = false;
    return;
  }

  if (!display.value.includes('.')) {
    display.value += '.';
  }
}


// Enter an operation
function inputOperator(newOperator: string) {
  if (display.value === 'Error') {
    return;
  }

  const currentNumber = parseFloat(display.value);

  if (firstNumber.value === null) {
    firstNumber.value = currentNumber;
  } else if (operator.value && !waitingForSecondNumber.value) {
    const result = performCalculation(
      firstNumber.value,
      currentNumber,
      operator.value
    );

    if (result === null) {
      display.value = 'Error';
      firstNumber.value = null;
      operator.value = null;
      return;
    }

    display.value = formatResult(result);
    firstNumber.value = result;
  }

  operator.value = newOperator;
  waitingForSecondNumber.value = true;
}


// Perform calculation
function performCalculation(
  first: number,
  second: number,
  operation: string
): number | null {

  switch (operation) {
    case '+':
      return first + second;

    case '-':
      return first - second;

    case '*':
      return first * second;

    case '/':
      if (second === 0) {
        return null;
      }
      return first / second;

    case '%':
      return first % second;

    default:
      return second;
  }
}


// Calculate final answer
function calculate() {
  if (
    firstNumber.value === null ||
    operator.value === null ||
    waitingForSecondNumber.value
  ) {
    return;
  }

  const secondNumber = parseFloat(display.value);

  const result = performCalculation(
    firstNumber.value,
    secondNumber,
    operator.value
  );

  if (result === null) {
    display.value = 'Error';
  } else {
    display.value = formatResult(result);
  }

  firstNumber.value = null;
  operator.value = null;
  waitingForSecondNumber.value = false;
}


// Remove the last digit
function backspace() {
  if (display.value === 'Error') {
    clear();
    return;
  }

  if (waitingForSecondNumber.value) {
    return;
  }

  if (display.value.length > 1) {
    display.value = display.value.slice(0, -1);
  } else {
    display.value = '0';
  }
}


// Format calculator result
function formatResult(result: number): string {
  if (!Number.isFinite(result)) {
    return 'Error';
  }

  return Number(result.toFixed(10)).toString();
}
</script>

<style scoped>
.calculator {
  max-width: 500px;
  margin: 20px auto;
}

.display {
  background: #222;
  color: white;
  min-height: 100px;
  border-radius: 12px;
  margin-bottom: 15px;
  padding: 20px;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  font-size: 40px;
  font-weight: bold;
  overflow-x: auto;
}

ion-button {
  height: 65px;
  font-size: 22px;
  margin: 4px 0;
}
</style>