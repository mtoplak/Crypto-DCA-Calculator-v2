<script lang="ts">
import { defineComponent } from 'vue';

export default {
    props: {
        selectedF: String,
        selectedC: String,
        selectedA: Number,
        selectedD: String
    },
    data() {
        return {
            cryptos: [
                { value: "bitcoin", text: "Bitcoin" },
                { value: "ethereum", text: "Ethereum" },
                { value: "tether", text: "Tether" },
                { value: "binancecoin", text: "BNB" },
                { value: "ripple", text: "XRP" },
                { value: "cardano", text: "Cardano" },
                { value: "dogecoin", text: "Dogecoin" },
                { value: "solana", text: "Solana" },
                { value: "polkadot", text: "Polkadot" },
            ],
            times: [
                { value: "day", text: "daily" },
                { value: "week", text: "weekly" },
                { value: "month", text: "monthly" },
                { value: "quarter", text: "quarterly" },
                { value: "year", text: "yearly" },
            ]
        }
    }
}
</script>

<template>
    <form method="" @submit.prevent="calculate">
        <div class="input-form-group">
            <label for="date">Start Date</label><br />
            <input class="form-control" type="date" name="date" id="date" v-model="selectedD" />
        </div>
        <br />
        <div class="input-form-group">
            <label for="coin">Cryptocurrency </label><br />
            <select name="coin" class="form-control" v-model="selectedC">
                <option v-for="option in cryptos" :value="option.value">
                    {{ option.text }}
                </option>
            </select>
        </div>
        <br />
        <div class="input-form-group">
            <label for="amount" style="text-align: center">Purchase Amount </label><br />
            <div class="input-group">
                <input class="form-control" type="number" min="0" step="any" name="amount"
                    style="min-width: 8em; max-width: 8em; margin: 0px" v-model="selectedA" />
                <div class="input-group-append">
                    <span class="input-group-text">$</span>
                </div>
            </div>
        </div>
        <br />
        <div class="input-form-group">
            <label for="time">Frequency</label><br />
            <select class="form-control" name="time" v-model="selectedF">
                <option v-for="option in times" :value="option.value">
                    {{ option.text }}
                </option>
            </select>
        </div>
        <br />
        <br />
        <input class="btn btn-outline-secondary" type="submit" value="Calculate" />
    </form>
</template>


<style scoped>
.input-group-text {
    margin-right: 0px;
}

.input-group {
    justify-content: center;
}

.btn-outline-secondary {
    max-height: 40px;
    min-width: 90px;
    max-width: 100px;
}
</style>
