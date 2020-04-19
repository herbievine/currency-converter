<template>
    <div id="app">

        <template v-if="this.width < 1000">
            <div id="overlay">
                <div id="overlay-box">
                    <p>This website was not designed to be on phones/tablets. Your phone/tablet may be high enough to put it in landscape mode, thus allowing this site to function properly on your device. Minimum width required: 1000px</p>
                </div>
            </div>
        </template>

        <div class="note">
            <h1 style="text-align: center;">Currency Converter</h1>
        </div>

        <div class="container">
            <div class="contain-left">
                <p>Amount to convert:</p>
            </div>
            <div class="contain-right">
                <label>
                    <input v-model="convertFrom.amount" type="text" name="amount" placeholder="Amount..." class="input box">
                </label>
            </div>
        </div>

        <div class="container">
            <div class="contain-left">
                <p>
                    Convert from:
                </p>
            </div>
            <div class="contain-right">
                <label>
                    <select v-model="convertFrom.currency" class="select box">
                        <option disabled style="background-color: #333333; color: whitesmoke;">
                            Popular:
                        </option>
                        <template v-for="value in allSymbols">
                            <option v-if="value.startsWith('EUR') || value.startsWith('GBP') || value.startsWith('USD') || value.startsWith('SGD') || value.startsWith('INR') || value.startsWith('AUD') || value.startsWith('CAD')" :key="'pop' + value">
                                {{ value.toString().replace(':', ': ') }}
                            </option>
                        </template>
                        <option disabled style="background-color: #333333; color: whitesmoke;">
                            All:
                        </option>
                        <option v-for="value in allSymbols" :key="'all' + value">
                            {{ value.toString().replace(':', ': ') }}
                        </option>
                    </select>
                </label>
            </div>
        </div>

        <div class="container">
            <div class="contain-left">
                <p>
                    Convert to:
                </p>
            </div>
            <div class="contain-right">
                <label>
                    <select v-model="convertTo.currency" class="select box">
                        <option disabled style="background-color: #333333; color: whitesmoke;">
                            Popular:
                        </option>
                        <template v-for="value in allSymbols">
                            <option v-if="value.startsWith('EUR') || value.startsWith('GBP') || value.startsWith('USD') || value.startsWith('SGD') || value.startsWith('INR') || value.startsWith('AUD') || value.startsWith('CAD')" :key="'pop' + value">
                                {{ value.toString().replace(':', ': ') }}
                            </option>
                        </template>
                        <option disabled style="background-color: #333333; color: whitesmoke;">
                            All:
                        </option>
                        <option v-for="value in allSymbols" :key="'all' + value">
                            {{ value.toString().replace(':', ': ') }}
                        </option>
                    </select>
                </label>
            </div>
        </div>

        <div class="container">
            <button @click="this.convert" type="button" name="button" class="box">Convert</button>
        </div>

        <div class="container">
            <p style="margin: 0; padding: 0;">
                <template v-if="!error.error && !isNaN(convertTo.amount) && !isNaN(convertFrom.amount)">
                    <span class="success"><i class="fas fa-check"></i> Successfully converted {{ convertFrom.amount + convertFrom.currency.toString().split(':')[1] }} to {{ convertTo.amount + convertTo.currency.toString().split(':')[1] }}</span>
                </template>
                <template v-if="error.error">
                    <span class="error"><i class="fas fa-times"></i> Sorry, {{ error.error.toString().toLowerCase() }}</span>
                </template>
            </p>
        </div>

        <div class="note">
            <p>
                <strong>
                    <u>Note:</u>
                </strong>
                Some currencies might not be available. Exchange rates provided by <a style="color: #43BAE8;" href="https://exchangeratesapi.io/" target="_BLANK">exchangeratesapi.io</a> and data sourced from the <a style="color: #43BAE8;" href="https://www.ecb.europa.eu/stats/policy_and_exchange_rates/euro_reference_exchange_rates/html/index.en.html" target="_BLANK">European Central Bank</a>. See on <a style="color: #43BAE8;" href="https://github.com/herbievine/currency-converter" target="_BLANK">GitHub <i style="font-size: 13px;" class="fas fa-external-link-alt"></i></a>
            </p>
        </div>
    </div>
</template>

<script>
    import currencies from './currencies';

    export default {
        name: 'App',
        data() {
            return {
                width: undefined,
                allSymbols: [],
                convertFrom: {
                    currency: '',
                    symbol: '',
                    amount: undefined,
                },
                convertTo: {
                    currency: '',
                    symbol: '',
                    amount: undefined,
                },
                error: {},
            }
        },
        methods: {
            onLoad() {
                console.log('%cVUE IS AWESOME 💖⚡', 'background-color: #34495E; color: #41B883; font-weight: bold;');
                console.log('%c¯\\_(ツ)_/¯', 'background-color: #34495E; color: #41B883; font-weight: bold;');

                let res = JSON.stringify(currencies);
                res = res.replace(/"/g, '');
                res = res.replace(/{/g, '');
                res = res.replace(/}/g, '');

                this.allSymbols = res.split(',');
            },
            convert() {
                this.convertFrom.symbol = this.convertFrom.currency.toString().split(':')[0];
                this.convertTo.symbol = this.convertTo.currency.toString().split(':')[0];

                if (this.convertFrom.amount === undefined) {
                    return this.error = {
                        error: 'you need to enter an amount to convert!'
                    }
                } else if (isNaN(this.convertFrom.amount)) {
                    return this.error = {
                        error: 'you need at valid number!'
                    }
                } else if (this.convertFrom.symbol === '' || this.convertTo.symbol === '') {
                    return this.error = {
                        error: 'you need two valid currencies!'
                    }
                }

                fetch(`https://api.exchangeratesapi.io/latest?base=${this.convertFrom.symbol}&symbols=${this.convertFrom.symbol},${this.convertTo.symbol}`)
                    .then(res => {
                        return res.json()
                    })
                    .then(res => {
                        if (Object.keys(res)[0] === 'error') {
                            this.error = res;
                        } else {
                            this.error = {};
                            this.convertTo.amount = this.convertFrom.amount * res.rates[this.convertTo.symbol];
                            this.convertTo.amount = this.convertTo.amount.toFixed(2);
                        }
                    })
            },
            checkWidth() {
                this.width = window.innerWidth;
                setTimeout(this.checkWidth, 1000);
            }
        },
        created() {
            this.onLoad();
            this.checkWidth();
        }
    }
</script>

<style>
    @import url('https://fonts.googleapis.com/css2?family=Open+Sans:wght@400;600&display=swap');

    * {
        box-sizing: border-box;
    }
    html {
        margin: 0;
        padding: 0;
        width: 100%;
        height: 100vh;
    }
    body {
        margin: 0;
        padding: 0;
        width: 100%;
        height: 100vh;
        background-color: #101010;
        overflow: hidden;
    }
    input:focus, button:focus, textarea:focus, select:focus {
        outline-width: 0;
    }
    button, select {
        cursor: pointer;
        border: none;
    }
    input {
        border: none;
    }
    #app {
        width: 100%;
        height: 100vh;
        margin: 3rem;
        color: whitesmoke;
    }
    .box {
        height: 40px;
        width: 100%;
        border-radius: 5px;
        margin: 10px 0;
        padding: 0 20px;
        font-size: 15px;
        font-family: 'Open Sans', sans-serif;
    }
    .container {
        width: 30%;
        margin: 0 35%;
        padding: 10px;
        font-size: 15px;
        font-family: 'Open Sans', sans-serif;
        display: flex;
    }
    .contain-left {
        flex: .5;
        padding-top: 5px;
    }
    .contain-right {
        flex: 1;
    }
    .success {
        color: #00ff00;
    }
    .error {
        color: #ff0000;
    }
    .note {
        width: 30%;
        margin: 0 35%;
        padding: 10px 50px;
        font-size: 15px;
        font-family: 'Open Sans', sans-serif;
        text-align: justify;
    }
    #overlay {
        position: fixed;
        display: block;
        width: 100%;
        height: 100%;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background-color: rgba(0,0,0,0.9);
        z-index: 2;
    }
    #overlay-box {
        position: fixed;
        display: block;
        width: 70%;
        height: 80%;
        margin: 10% 15%;
        align-items: center;
        background-color: #FFFFFF;
        z-index: 4;
    }
    #overlay-box p {
        padding: 30px 40px;
        color: #333;
        text-align: justify;
        font-size: 20px;
        font-family: 'Open Sans', sans-serif;
    }

</style>
