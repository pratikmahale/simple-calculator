<script setup lang="ts">
const buttonClasses = "bg-gray-300 rounded border w-12 h-12 hover:bg-gray-400 hover:text-white active:text-white active:bg-gray-500";
const currentResult = useState<number | null | string>('currentResult', () => null);
const error = useState<string>('error', () => '');
type Expression = (number | (typeof operations)[number])[];
const memory = useState<Array<number | typeof operations[number]>>('memory', () => []);
const operations = ["+", "-", "*", "/"] as const;

function addToMemory(item: number | typeof operations[number]) {
    if (currentResult.value !== null) {
        resetMemory();
    }
    const lastElement = memory.value.at(-1)?.toString();
    if (typeof lastElement === 'undefined') {
        memory.value = [item];
        return;
    }
    if (lastElement.includes("+") || lastElement.includes("-") || lastElement.includes("*") || lastElement.includes("/")) {
        memory.value = [...memory.value, item];
        return;
    }
    if (!isNaN(Number(item))) {
        memory.value = [...memory.value.slice(0, -1), Number(lastElement + item)];
        return;
    }
    memory.value = [...memory.value, item];
}


function evaluateExpression(expression: Expression): number | string {
    const { numbers, operators } = parseExpression(expression);
    // first round, higher precedence operations
    for (let i = 0; i < operators.length; i++) {
        let result = 0;
        if (operators[i] === "*") {
            result = numbers[i] * numbers[i + 1];
            numbers.splice(i, 2, result);
            operators.splice(i, 1);
        }
        if (operators[i] === "/") {
            result = numbers[i] / numbers[i + 1];
            numbers.splice(i, 2, result);
            operators.splice(i, 1);
        }
    }
    // additional rounds, until there are no more operators
    while (operators.length > 0) {
        if (operators[0] === "+") {
            const result = numbers[0] + numbers[1];
            numbers.splice(0, 2, result);
            operators.splice(0, 1);
        }
        if (operators[0] === "-") {
            const result = numbers[0] - numbers[1];
            numbers.splice(0, 2, result);
            operators.splice(0, 1);
        }
    }
    if (numbers.length !== 1) {
        throw "Invalid expression";
    }
    // return the last number left
    return numbers[0];
}

function parseExpression(expression: Expression) {
    let numberFound = false;
    let operatorFound = false;
    const numbers: number[] = [];
    const operators: (typeof operations)[number][] = [];
    for (const i in expression) {
        const value = expression[i];
        if (typeof value === "string") {
            if (Number(i) === 0) {
                throw new Error("Operator at the beginning");
            }
            if (operatorFound) {
                throw new Error("Two operators in a row");
            }
            operators.push(value);
            operatorFound = true;
            numberFound = false;
            continue;
        }
        if (typeof value === "number") {
            if (numberFound) {
                throw new Error("Two numbers in a row");
            }
            numbers.push(value);
            numberFound = true;
            operatorFound = false;
        }
    }
    return { numbers, operators };
}
function reverseSignLastItem(
    items: Array<number | (typeof operations)[number]>
) {
    const lastElement = items.at(-1)?.toString();
    if (typeof lastElement === "undefined") {
        return items;
    }
    if (operations.includes(lastElement as (typeof operations)[number])) {
        return items;
    }
    return [...items.slice(0, -1), Number(lastElement) * -1];
}

function calculate() {
    try {
        const result = evaluateExpression(memory.value);
        currentResult.value = result;
    } catch (e) {
        error.value = (e as {
            message: string;
        }).message;
    }

}
function reverseSign() {
    memory.value = reverseSignLastItem(memory.value);
}
function isOperator(item: number | string) {
    return operations.includes(item as (typeof operations)[number]);
}
function resetMemory() {
    memory.value = [];
    currentResult.value = null;
    error.value = "";
}

</script>
<template>
    <div class="h-content container mx-auto bg-gray-200 border w-[400px] mt-12 rounded shadow-xl">
        <div class="w-full h-24 px-12 bg-green-200">
            <div class="flex flex-wrap ">
                <div v-for="(item, index) in memory" @key="index"><span class="text-2xl leading-[3rem]"
                        v-if="isOperator(item)">{{ item }}</span> <span v-if="!isOperator(item)"
                        class="leading-[3rem] text-xl">{{ item }}</span></div>
            </div>
            <div v-if="typeof error !== 'undefined' && error.length" class="py-1 text-xs text-red-500">{{ error }}</div>
            <div class="text-3xl font-bold" v-if="currentResult !== null">{{ String(currentResult) }}</div>
        </div>
        <div class="grid w-full grid-cols-4 gap-12 px-12 py-12">
            <ul class="grid grid-cols-3 col-span-3 gap-12">
                <li v-for="(item, index) in [1, 2, 3, 4, 5, 6, 7, 8, 9, 0]" @key="index">
                    <button :class="buttonClasses" @click="addToMemory(item)">{{ item }}</button>
                </li>
                <li> <button :class="buttonClasses" @click="calculate">=</button> </li>
                <li> <button :class="buttonClasses" @click="resetMemory">MC</button> </li>
                <li> <button :class="buttonClasses" @click="reverseSign">-/+</button> </li>
            </ul>
            <div class="col-span-1">
                <ul class="flex flex-col col-span-3 gap-12">
                    <li v-for="(item, index) in operations" @key="index">
                        <button :class="buttonClasses" @click="addToMemory(item)">{{ item }}</button>
                    </li>
                </ul>
            </div>
        </div>

    </div>
</template>