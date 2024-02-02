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
    if (operations.includes(lastElement as (typeof operations)[number])) {
        memory.value = [...memory.value, item];
        return;
    }
    if (!isNaN(Number(item))) {
        memory.value = [...memory.value.slice(0, -1), Number(lastElement + item)];
        return;
    }
    memory.value = [...memory.value, item];
}



function evaluateExpression(exp: Expression): number | string {
    const expression = [...exp];
    // first round, higher precedence operations
    for (let i = 0; i < expression.length; i++) {
        if (expression[i] === "*" || expression[i] === "/") {
            const number1 = expression[i - 1];
            const number2 = expression[i + 1];
            if (typeof number1 !== 'number' || typeof number2 !== 'number') {
                throw "Invalid expression";
            }
            const result = expression[i] === "*" ? number1 * number2 : number1 / number2;
            expression.splice(i - 1, 3, result);
            // move the index
        }
    }
    // additional rounds, until there is only the result left in the expression
    while (expression.length > 1) {
        for (let i = 0; i < expression.length; i++) {
            if (expression[i] === "+" || expression[i] === "-") {
                const number1 = expression[i - 1];
                const number2 = expression[i + 1];
                if (typeof number1 !== 'number' || typeof number2 !== 'number') {
                    throw "Invalid expression";
                }
                const result = expression[i] === "+" ? number1 + number2 : number1 - number2;
                expression.splice(i - 1, 3, result);
            }
        }
    }
    if (expression.length !== 1) {
        throw "Invalid expression";
    }
    // return the last number left
    return expression[0];
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
        error.value = typeof e === 'string' ? e : "Invalid expression";
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