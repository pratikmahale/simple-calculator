<script setup lang="ts">
const operations: Array<'+' | '-' | '*' | '/'> = ['+', '-', '*', '/'];
const memory = useState<Array<number | typeof operations[number]>>('memory', () => []);

const currentResult = useState<number | null | string>('currentResult', () => null);
const buttonClasses = "bg-gray-300 rounded border w-12 h-12 hover:bg-gray-400 hover:text-white active:text-white active:bg-gray-500";
function updateNumber(item: number | typeof operations[number]) {
    if(currentResult.value !== null) {
        resetMemory();
    }
    const lastElement = memory.value.at(-1)?.toString();
    if (typeof lastElement === 'undefined') {
        memory.value = [item];
        return;
    }
    if(lastElement.includes("+") || lastElement.includes("-") || lastElement.includes("*") || lastElement.includes("/")) {
        memory.value = [...memory.value, item];
        return;
    }
    if(!isNaN(Number(item))) {
        memory.value = [...memory.value.slice(0, -1), Number(lastElement + item)];
        return;
    }
    memory.value = [...memory.value, item];
}

function evaluateExpression(expression: (number | typeof operations[number])[]): number | string {
  if (expression.length % 2 === 0) {
    return 'Invalid expression';
  }

  const isValidOperator = (op: string) => operations.includes(op as typeof operations[number]);
  const precedence = { '+': 1, '-': 1, '*': 2, '/': 2 };

  const applyOperation = (operator: string, operand1: number, operand2: number): number => {
    if (operator === '+') {
      return operand1 + operand2;
    } else if (operator === '-') {
      return operand1 - operand2;
    } else if (operator === '*') {
      return operand1 * operand2;
    } else if (operator === '/') {
      if (operand2 === 0) {
        throw new Error('Division by zero');
      }
      return operand1 / operand2;
    }
    throw new Error('Invalid operator');
  };

  const numbers: number[] = [];
  const operators: string[] = [];

  for (const token of expression) {
    if (typeof token === 'number') {
      numbers.push(token);
    } else if (typeof token === 'string' && isValidOperator(token)) {
      while (
        operators.length > 0 &&
        isValidOperator(operators[operators.length - 1]) &&
        precedence[token] <= precedence[operators[operators.length - 1] as typeof operations[number]]
      ) {
        const operator = operators.pop() as string;
        const operand2 = numbers.pop() as number;
        const operand1 = numbers.pop() as number;
        numbers.push(applyOperation(operator, operand1, operand2));
      }
      operators.push(token);
    } else {
      return 'Invalid expression';
    }
  }

  while (operators.length > 0) {
    const operator = operators.pop() as string;
    const operand2 = numbers.pop() as number;
    const operand1 = numbers.pop() as number;
    numbers.push(applyOperation(operator, operand1, operand2));
  }

  const result = numbers[0];

  if (!isFinite(result)) {
    return 'Division by zero';
  }

  return result;
}

function reverseSignLastItem() {
    const lastElement = memory.value.at(-1)?.toString();
    if (typeof lastElement === 'undefined') {
        return;
    }
    if(lastElement.includes("+") || lastElement.includes("-") || lastElement.includes("*") || lastElement.includes("/")) {
        return;
    }
    memory.value = [...memory.value.slice(0, -1), Number(lastElement) * -1];
}
function evaluateResult() { 
    const result = evaluateExpression(memory.value);
    currentResult.value = result;
}
function resetMemory() {
    memory.value = [];
    currentResult.value = null;   
}    
function isOperator(item: number | string) {
    return operations.includes(item as typeof operations[number]);
}
</script>
<template>
   <div class="container mx-auto bg-gray-200 border w-[400px] h-[600px] mt-12 rounded shadow-xl">
        <div class="bg-green-200 w-full h-24 px-12">
            <div class=" flex flex-wrap">
                <div v-for="(item, index) in memory" @key="index"><span class="text-2xl leading-[3rem]" v-if="isOperator(item)">{{ item }}</span> <span v-if="!isOperator(item)" class="leading-[3rem] text-xl">{{ item }}</span></div>
            </div>
            <div class="text-3xl font-bold" v-if="currentResult !== null">{{ String(currentResult)  }}</div>
        </div>
        <div class="w-full grid grid-cols-4 px-12 py-12 gap-12">
            <ul class="grid grid-cols-3 col-span-3 gap-12">
                <li v-for="(item, index) in [1,2,3,4,5,6,7,8,9,0]" @key="index">
                   <button  :class="buttonClasses" @click="updateNumber(item)">{{ item }}</button> 
                </li>
                <li> <button :class="buttonClasses" @click="evaluateResult">=</button> </li>
                <li> <button :class="buttonClasses" @click="resetMemory">MC</button> </li>
                <li> <button :class="buttonClasses" @click="reverseSignLastItem">-/+</button> </li>
            </ul>
            <div class="col-span-1">
                <ul class="col-span-3 flex flex-col gap-12">
                    <li v-for="(item, index) in operations" @key="index">
                    <button :class="buttonClasses" @click="updateNumber(item)">{{ item }}</button> 
                    </li>
                </ul>
            </div>
        </div>
        
   </div>
</template>