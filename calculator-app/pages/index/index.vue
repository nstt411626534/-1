<template>
    <view class="calculator-container">
        <!-- 状态栏占位 -->
        <view class="status-bar"></view>
        
        <!-- 显示区域 -->
        <view class="display-area">
            <view class="history-display" ref="historyDisplay">
                <view v-html="historyHtml" class="katex-container"></view>
            </view>
            <view class="main-display">
                <view v-html="displayHtml" class="katex-container main-katex"></view>
            </view>
        </view>
        
        <!-- 模式切换 -->
        <view class="mode-switch">
            <view 
                class="mode-item" 
                :class="{ active: currentMode === 'basic' }"
                @click="switchMode('basic')"
            >
                标准
            </view>
            <view 
                class="mode-item" 
                :class="{ active: currentMode === 'scientific' }"
                @click="switchMode('scientific')"
            >
                科学
            </view>
        </view>
        
        <!-- 按钮区域 -->
        <view class="buttons-area">
            <!-- 科学计算额外行 -->
            <view v-if="currentMode === 'scientific'" class="button-row">
                <view class="calc-btn btn-function" @click="inputFunc('sin')">sin</view>
                <view class="calc-btn btn-function" @click="inputFunc('cos')">cos</view>
                <view class="calc-btn btn-function" @click="inputFunc('tan')">tan</view>
                <view class="calc-btn btn-function" @click="inputFunc('log')">log</view>
            </view>
            
            <view v-if="currentMode === 'scientific'" class="button-row">
                <view class="calc-btn btn-function" @click="inputFunc('asin')">sin⁻¹</view>
                <view class="calc-btn btn-function" @click="inputFunc('acos')">cos⁻¹</view>
                <view class="calc-btn btn-function" @click="inputFunc('atan')">tan⁻¹</view>
                <view class="calc-btn btn-function" @click="inputFunc('ln')">ln</view>
            </view>
            
            <!-- 主控制行 -->
            <view class="button-row">
                <view class="calc-btn btn-clear" @click="clearAll">AC</view>
                <view class="calc-btn btn-clear" @click="backspace">⌫</view>
                <view class="calc-btn btn-operator" @click="inputOperator('%')">%</view>
                <view class="calc-btn btn-operator" @click="inputOperator('/')">÷</view>
            </view>
            
            <view class="button-row">
                <view class="calc-btn btn-number" @click="inputNumber('7')">7</view>
                <view class="calc-btn btn-number" @click="inputNumber('8')">8</view>
                <view class="calc-btn btn-number" @click="inputNumber('9')">9</view>
                <view class="calc-btn btn-operator" @click="inputOperator('*')">×</view>
            </view>
            
            <view class="button-row">
                <view class="calc-btn btn-number" @click="inputNumber('4')">4</view>
                <view class="calc-btn btn-number" @click="inputNumber('5')">5</view>
                <view class="calc-btn btn-number" @click="inputNumber('6')">6</view>
                <view class="calc-btn btn-operator" @click="inputOperator('-')">−</view>
            </view>
            
            <view class="button-row">
                <view class="calc-btn btn-number" @click="inputNumber('1')">1</view>
                <view class="calc-btn btn-number" @click="inputNumber('2')">2</view>
                <view class="calc-btn btn-number" @click="inputNumber('3')">3</view>
                <view class="calc-btn btn-operator" @click="inputOperator('+')">+</view>
            </view>
            
            <view class="button-row">
                <view class="calc-btn btn-number" @click="toggleSign">±</view>
                <view class="calc-btn btn-number" @click="inputNumber('0')">0</view>
                <view class="calc-btn btn-number" @click="inputDecimal">.</view>
                <view class="calc-btn btn-equals" @click="calculate">=</view>
            </view>
            
            <!-- 科学计算底行 -->
            <view v-if="currentMode === 'scientific'" class="button-row">
                <view class="calc-btn btn-function" @click="inputFunc('sqrt')">√</view>
                <view class="calc-btn btn-function" @click="inputPower">x²</view>
                <view class="calc-btn btn-function" @click="inputPi">π</view>
                <view class="calc-btn btn-function" @click="inputE">e</view>
            </view>
        </view>
    </view>
</template>

<script>
import katex from 'katex'
import 'katex/dist/katex.min.css'

export default {
    data() {
        return {
            currentMode: 'basic',
            expression: '',
            history: '',
            lastResult: null,
            waitingForOperand: false,
            displayHtml: '',
            historyHtml: ''
        }
    },
    methods: {
        renderExpression(expr) {
            try {
                let tex = expr
                    .replace(/\*/g, '\\times')
                    .replace(/\//g, '\\div')
                    .replace(/-/g, '−')
                    .replace(/sqrt/g, '\\sqrt')
                    .replace(/sin/g, '\\sin')
                    .replace(/cos/g, '\\cos')
                    .replace(/tan/g, '\\tan')
                    .replace(/log/g, '\\log')
                    .replace(/ln/g, '\\ln')
                    .replace(/pi/g, '\\pi')
                    .replace(/\^2/g, '^2')
                    .replace(/\^/g, '^')
                
                return katex.renderToString(tex || '0', {
                    throwOnError: false,
                    displayMode: false
                })
            } catch (e) {
                return katex.renderToString('0', { throwOnError: false })
            }
        },
        
        updateDisplay() {
            this.displayHtml = this.renderExpression(this.expression)
            this.historyHtml = this.renderExpression(this.history)
        },
        
        switchMode(mode) {
            this.currentMode = mode
        },
        
        inputNumber(num) {
            if (this.waitingForOperand) {
                this.expression = ''
                this.waitingForOperand = false
            }
            this.expression += num
            this.updateDisplay()
        },
        
        inputDecimal() {
            if (this.waitingForOperand) {
                this.expression = '0.'
                this.waitingForOperand = false
            } else if (!this.expression.includes('.')) {
                const parts = this.expression.split(/[+\-*/]/)
                const lastPart = parts[parts.length - 1]
                if (!lastPart.includes('.')) {
                    this.expression += '.'
                }
            }
            this.updateDisplay()
        },
        
        inputOperator(op) {
            if (this.waitingForOperand && this.lastResult !== null) {
                this.expression = this.lastResult.toString()
            }
            this.expression += op
            this.waitingForOperand = false
            this.updateDisplay()
        },
        
        inputFunc(func) {
            if (this.waitingForOperand) {
                this.expression = ''
                this.waitingForOperand = false
            }
            switch(func) {
                case 'sqrt':
                    this.expression += 'sqrt('
                    break
                case 'sin':
                case 'cos':
                case 'tan':
                case 'asin':
                case 'acos':
                case 'atan':
                case 'log':
                case 'ln':
                    this.expression += func + '('
                    break
            }
            this.updateDisplay()
        },
        
        inputPower() {
            this.expression += '^2'
            this.updateDisplay()
        },
        
        inputPi() {
            this.expression += 'pi'
            this.updateDisplay()
        },
        
        inputE() {
            this.expression += 'e'
            this.updateDisplay()
        },
        
        toggleSign() {
            if (this.expression) {
                if (this.expression.startsWith('-')) {
                    this.expression = this.expression.substring(1)
                } else {
                    this.expression = '-' + this.expression
                }
                this.updateDisplay()
            }
        },
        
        backspace() {
            if (this.expression.length > 0) {
                this.expression = this.expression.slice(0, -1)
                this.updateDisplay()
            }
        },
        
        clearAll() {
            this.expression = ''
            this.history = ''
            this.lastResult = null
            this.waitingForOperand = false
            this.updateDisplay()
        },
        
        calculate() {
            try {
                let evalExpr = this.expression
                    .replace(/pi/g, Math.PI.toString())
                    .replace(/e(?![xp])/g, Math.E.toString())
                    .replace(/sqrt\(/g, 'Math.sqrt(')
                    .replace(/sin\(/g, 'Math.sin(')
                    .replace(/cos\(/g, 'Math.cos(')
                    .replace(/tan\(/g, 'Math.tan(')
                    .replace(/asin\(/g, 'Math.asin(')
                    .replace(/acos\(/g, 'Math.acos(')
                    .replace(/atan\(/g, 'Math.atan(')
                    .replace(/log\(/g, 'Math.log10(')
                    .replace(/ln\(/g, 'Math.log(')
                    .replace(/\^2/g, '**2')
                    .replace(/\^/g, '**')
                
                const result = new Function(`return ${evalExpr}`)()
                
                this.history = this.expression + ' ='
                this.expression = result.toString()
                this.lastResult = result
                this.waitingForOperand = true
                this.updateDisplay()
            } catch (e) {
                this.expression = 'Error'
                this.updateDisplay()
                setTimeout(() => {
                    this.expression = ''
                    this.updateDisplay()
                }, 1500)
            }
        }
    },
    
    mounted() {
        this.updateDisplay()
    }
}
</script>

<style scoped>
.calculator-container {
    display: flex;
    flex-direction: column;
    height: 100vh;
    background: linear-gradient(180deg, #0F1419 0%, #1A1A2E 100%);
    padding: 0 20rpx;
    box-sizing: border-box;
}

.status-bar {
    height: var(--status-bar-height);
    width: 100%;
}

.display-area {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: flex-end;
    align-items: flex-end;
    padding: 40rpx 20rpx;
    min-height: 300rpx;
}

.history-display {
    font-size: 32rpx;
    color: rgba(255, 255, 255, 0.6);
    margin-bottom: 20rpx;
    min-height: 50rpx;
    max-width: 100%;
    overflow-x: auto;
}

.main-display {
    font-size: 72rpx;
    color: #FFFFFF;
    font-weight: 300;
    max-width: 100%;
    overflow-x: auto;
    line-height: 1.2;
}

.katex-container {
    text-align: right;
}

.main-katex {
    font-size: 72rpx;
}

.katex-container .katex {
    font-size: inherit;
    color: inherit;
}

.mode-switch {
    display: flex;
    justify-content: center;
    margin-bottom: 20rpx;
    gap: 20rpx;
}

.mode-item {
    padding: 16rpx 48rpx;
    background: rgba(255, 255, 255, 0.1);
    border-radius: 40rpx;
    color: rgba(255, 255, 255, 0.6);
    font-size: 28rpx;
    transition: all 0.3s;
}

.mode-item.active {
    background: #3CC7FF;
    color: #FFFFFF;
}

.buttons-area {
    padding-bottom: 60rpx;
}

.button-row {
    display: flex;
    justify-content: space-between;
    margin-bottom: 20rpx;
    gap: 16rpx;
}

.calc-btn {
    flex: 1;
    height: 120rpx;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 24rpx;
    font-size: 40rpx;
    font-weight: 500;
    transition: all 0.15s;
}

.calc-btn:active {
    transform: scale(0.95);
    opacity: 0.8;
}

.btn-number {
    background: rgba(255, 255, 255, 0.1);
    color: #FFFFFF;
}

.btn-operator {
    background: #3CC7FF;
    color: #FFFFFF;
}

.btn-function {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: #FFFFFF;
    font-size: 32rpx;
}

.btn-clear {
    background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
    color: #FFFFFF;
}

.btn-equals {
    background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);
    color: #FFFFFF;
}
</style>
