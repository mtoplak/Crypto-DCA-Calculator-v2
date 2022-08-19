<script lang="ts">
import { defineComponent } from 'vue';

export default {
  props: {
    longResults: Boolean,
    show: Boolean,
    amount: String,
    averageBuyPrice: String,
    roi: String,
    timesInvested: Number,
    currentValue: String,
    totalInvested: Number,
    resultsToshare: String
  },
  data() {
    return {

    }
  }
}
</script>

<template>
  <div>
    <div v-if="show" class="input-form-group"><span class="title1">Amount Acquired</span><br />{{ amount }} <span
        class="coin"></span></div>
    <div v-if="show" class='input-form-group'><span class="title1">Average Buy Price</span><br />{{ averageBuyPrice }}$
    </div>
    <div v-if="longResults" class='input-form-group'><span class="title1">ROI</span><br />{{ roi }}%</div>
    <div v-if="longResults" class="input-form-group"><span class="title1">Times Invested</span><br />{{ timesInvested }}
    </div>
    <div v-if="longResults" class="input-form-group"><span class="title1">Current Value</span><br />{{ currentValue }}$
    </div>
    <div v-if="show" class="input-form-group"><span class="title1">Total Invested</span><br />{{ totalInvested }}$</div>
  </div>
  <div v-show="longResults" id="curve_chart"></div>
  <div id="result">
    <h3><span v-if="show">I have spent a total of <span id="total" class="strong"></span>$ on <span id="coinamount"
          class="strong"></span> <span class="coin strong"></span> since <span id="startdate" class="strong"> </span>.
      </span>
      <div v-if="timesInvested === 1">I'll keep waiting for my investment to grow.</div>
      <div v-if="longResults"> My total investment is <span id="profit" class="strong"></span> and I
        have invested <span id="times" class="strong"></span> times. My average buy price was <span id="average"
          class="strong"></span>$.</div>
    </h3>
  </div>
  <span v-if="show">Share the results:
    <button id="copy" title="Copy" @click="share">
      <img src="../assets/copy.png" /></button>
    <a class="twitter-share-button" href="https://twitter.com/intent/tweet" data-size="large"
      :data-text="resultsToShare" data-related="twitterapi,twitter">
      <!--<img style="cursor: pointer" src="./assets/twitter.png" title="Tweet" />-->
    </a></span>
</template>


<style scoped>
</style>
