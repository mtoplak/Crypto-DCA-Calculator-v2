<template>
  <AppHeader heading="Cryptocurrency DCA Calculator" />
  <div class="container">
    <div class="row">
      <div class="col-md-4 col-sm-12">
        <AppForm :selectedA="100" :selectedF="selectedF" :selectedD="selectedD" :selectedC="selectedC"
          @submit="calculate" />
      </div>
      <div class="col-md-8 col-sm-12">
        <AppResults :longResults="longResults" :show="show" :amount="amount" :averageBuyPrice="averageBuyPrice"
          :roi="roi" :timesInvested="timesInvested" :currentValue="currentValue" :totalInvested="totalInvested" />
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, vModelCheckbox, ref, VueElement } from "vue";
import AppHeader from "./components/AppHeader.vue";
import AppForm from "./components/AppForm.vue";
import AppResults from "./components/AppResults.vue";
import $ from "jquery";
import Vue from "vue";

var current: number;
google.charts.load("current", { packages: ["corechart"] });

export default defineComponent({
  components: {
    AppHeader,
    AppForm,
    AppResults
  },
  data() {
    return {
      show: false,
      longResults: false,
      selectedF: "",
      selectedC: "",
      selectedA: 0,
      selectedD: new Date().toISOString().slice(0, 10),
      amount: '',
      totalInvested: 0,
      averageBuyPrice: '',
      resultsToShare: '',
      currentValue: '',
      roi: '',
      timesInvested: 0
    };
  },
  methods: {
    calculate(
      e?: any,
      startDate?: string,
      cryptocurrency?: any,
      purchaseAmount?: any,
      frequency?: string
    ): boolean {
      if (e) {
        e.preventDefault();
      }
      this.show = true;
      try {
        if (!startDate) {
          this.calculateTimes();
        } else {
          this.calculateTimes(
            startDate,
            cryptocurrency,
            purchaseAmount,
            frequency
          );
        }
      } catch (err) {
        console.log(err);
      }
      return false;
    },
    load() {
      if (localStorage.getItem("cryptocurrency") != null) {
        let startDate: any = localStorage.getItem("startDate");
        let cryptocurrency: string = localStorage.getItem("cryptocurrency")!;
        let purchaseAmount: number = +localStorage.getItem("purchaseAmount")!;
        let frequency: string = localStorage.getItem("frequency")!;
        this.calculate(
          event,
          startDate,
          cryptocurrency,
          purchaseAmount,
          frequency
        );
        this.show = true;
        startDate = Date.parse(startDate);
        startDate = new Date(startDate).toISOString();
        this.selectedD = startDate.slice(0, 10);
        this.selectedC = cryptocurrency;
        this.selectedA = purchaseAmount;
        this.selectedF = frequency;
      } else {
        this.selectedC = "bitcoin";
        this.selectedF = "month";
        this.selectedA = 100;
      }
    },
    async calculateTimes(
      startDate?: string,
      cryptocurrency?: string,
      purchaseAmount?: number,
      frequency?: string
    ): Promise<void> {
      let start: any;
      let coin: string;
      let amount: any;
      let time: string;

      !startDate
        ? (start = (<HTMLInputElement>document.forms[0][0]).value)
        : (start = startDate);
      !frequency
        ? (time = (<HTMLInputElement>document.forms[0][3]).value)
        : (time = frequency);

      let day: number | string;
      let month: number | string;
      let year: number;
      let myDates: string[] | string = []; // dates when you bought
      let myPrices: number[] | number = []; // prices at which you bought
      let accAmounts: number[] | number = []; // accumulated amount
      let date: string;
      let count: number = 0;
      let cryptoAmount: number = 0;
      this.longResults = false;

      start = new Date(start);
      day = start.getDate();
      month = start.getMonth() + 1;
      year = start.getFullYear();

      if (month < 10) {
        month = "0" + month;
      }
      if (day < 10) {
        day = "0" + day;
      }

      let firstDate: string = day + "-" + month + "-" + year;
      myDates.push(firstDate);
      !purchaseAmount
        ? (amount = (document.forms[0][2] as HTMLInputElement).value)
        : (amount = purchaseAmount);
      let today: any = new Date();
      month = today.getMonth() + 1;
      year = today.getFullYear();

      if (month < 10) {
        month = "0" + month;
      }
      day = today.getDate();
      if (today.getDate() < 10) {
        day = "0" + day;
      }

      today = year + "-" + month + "-" + day;
      start = new Date(Date.parse(start));
      start = start.toString();

      localStorage.setItem("startDate", start);
      //localStorage.setItem('cryptocurrency', coin);
      localStorage.setItem("purchaseAmount", amount);
      localStorage.setItem("frequency", time);

      let meantime: number;
      switch (time) {
        case "day":
          meantime = 1 * 86400000;
          break;
        case "week":
          meantime = 7 * 86400000;
          break;
        case "month":
          meantime = 30 * 86400000;
          break;
        case "quarter":
          meantime = 30 * 4 * 86400000;
          break;
        case "year":
          meantime = 365 * 86400000;
          break;
      }

      do {
        !cryptocurrency
          ? (coin = (<HTMLInputElement>document.forms[0][1]).value)
          : (coin = cryptocurrency);
        localStorage.setItem("cryptocurrency", coin);

        const response = await fetch(
          `https://api.coingecko.com/api/v3/simple/price?ids=${coin}&vs_currencies=usd`,
          { method: "GET" }
        );
        const data = await response.json();
        current = data[coin].usd;

        if (Date.parse(start) == Date.parse(today)) {
          !cryptocurrency
            ? (coin = (<HTMLInputElement>document.forms[0][1]).value)
            : (coin = cryptocurrency);
          localStorage.setItem("cryptocurrency", coin);

          const response = await fetch(
            `https://api.coingecko.com/api/v3/simple/price?ids=${coin}&vs_currencies=usd`,
            { method: "GET" }
          );
          const data = await response.json();
          const price = data[coin].usd;
          current = data[coin].usd;
          cryptoAmount += amount / data[coin].usd;
          myPrices.push(price);
          accAmounts.push(cryptoAmount);
          $("#coinamount").text(cryptoAmount.toFixed(4));
          this.amount = cryptoAmount.toFixed(4);
          this.currentValue = (current * cryptoAmount).toFixed(2);
          count++;
          if (((current * cryptoAmount) / (count * amount)) * 100 - 100 > 0) {
            $("#profit").text(
              (
                ((current * cryptoAmount) / (count * amount)) * 100 -
                100
              ).toFixed(2) + "% up"
            );
            this.roi = (((current * cryptoAmount) / (count * amount)) * 100 - 100).toFixed(2);
          } else {
            $("#profit").text(
              (
                ((current * cryptoAmount) / (count * amount)) * 100 -
                100
              ).toFixed(2) + "% down"
            );
            this.roi = (((current * cryptoAmount) / (count * amount)) * 100 - 100).toFixed(2);
          }
          this.showResults(count, cryptoAmount, amount, coin);
          let sum: number = 0;
          for (let i = 0; i < myPrices.length; i++) {
            sum += myPrices[i] * 1;
          }
          this.drawChart(myDates, myPrices, accAmounts);
          $("#average").text((sum / myPrices.length).toFixed(2));
          this.averageBuyPrice = (sum / myPrices.length).toFixed(2);
          break;
        } else if (Date.parse(start) < Date.parse(today)) {
          start = new Date(Date.parse(start));
          month = start.getMonth() + 1;
          year = start.getFullYear();
          day = start.getDate();

          if (month < 10) {
            month = "0" + month;
          }
          if (day < 10) {
            day = "0" + day;
          }

          start = year + "-" + month + "-" + day;
          count++;
          !cryptocurrency
            ? (coin = (<HTMLInputElement>document.forms[0][1]).value)
            : (coin = cryptocurrency);
          localStorage.setItem("cryptocurrency", coin);
          date = day + "-" + month + "-" + start.substr(0, 4);

          const response = await fetch(
            `https://api.coingecko.com/api/v3/coins/${coin}/history?date=${date}&localization=false`,
            { method: "GET" }
          );
          const data = await response.json();
          const price = await data.market_data.current_price.usd;
          myPrices.push(price);
          cryptoAmount += amount / data.market_data.current_price.usd;
          accAmounts.push(cryptoAmount);
          $("#coinamount").text(cryptoAmount.toFixed(4));
          this.amount = cryptoAmount.toFixed(4);
          this.currentValue = (current * cryptoAmount).toFixed(2);
          if (((current * cryptoAmount) / (count * amount)) * 100 - 100 > 0) {
            $("#profit").text(
              (
                ((current * cryptoAmount) / (count * amount)) * 100 -
                100
              ).toFixed(2) + "% up"
            );
            this.roi = (((current * cryptoAmount) / (count * amount)) * 100 - 100).toFixed(2);
          } else {
            $("#profit").text(
              (
                ((current * cryptoAmount) / (count * amount)) * 100 -
                100
              ).toFixed(2) + "% down"
            );
            this.roi = (((current * cryptoAmount) / (count * amount)) * 100 - 100).toFixed(2);
          }
          let sum: number = 0;
          for (let i = 0; i < myPrices.length; i++) {
            sum += myPrices[i] * 1;
          }
          this.drawChart(myDates, myPrices, accAmounts);
          $("#average").text((sum / myPrices.length).toFixed(2));
          this.averageBuyPrice = (sum / myPrices.length).toFixed(2);
          this.showResults(count, cryptoAmount, amount, coin);

          if (Date.parse(today) - Date.parse(start) < meantime!) {
            this.showResults(count, cryptoAmount, amount, coin);
            break;
          }

          switch (time) {
            case "day":
              start = this.addMilliseconds(start, 86400000);
              break;
            case "week":
              start = this.addMilliseconds(start, 86400000 * 7);
              break;
            case "month":
              start = this.addOneMonth(start);
              break;
            case "quarter":
              start = this.addAQuarter(start);
              break;
            case "year":
              start = this.addMilliseconds(start, 86400000 * 365);
              break;
          }
          start = new Date(start + "Z").toISOString();
          year = start.substring(0, 4);
          month = start.substr(5, 2);
          day = start.substr(8, 2);
          date = day + "-" + month + "-" + year;
          myDates.push(date);

          if (Date.parse(start) == Date.parse(today)) {
            const response = await fetch(
              `https://api.coingecko.com/api/v3/simple/price?ids=${coin}&vs_currencies=usd`,
              { method: "GET" }
            );
            const data = await response.json();
            const price = await data[coin].usd;
            current = data[coin].usd;
            myPrices.push(price);
            cryptoAmount += amount / data[coin].usd;
            accAmounts.push(cryptoAmount);
            $("#coinamount").text(cryptoAmount.toFixed(4));
            this.amount = cryptoAmount.toFixed(4);
            this.currentValue = (current * cryptoAmount).toFixed(2);
            count++;
            if (((current * cryptoAmount) / (count * amount)) * 100 - 100 > 0) {
              $("#profit").text(
                (
                  ((current * cryptoAmount) / (count * amount)) * 100 -
                  100
                ).toFixed(2) + "% up"
              );
              this.roi = (((current * cryptoAmount) / (count * amount)) * 100 - 100).toFixed(2);
            } else {
              $("#profit").text(
                (
                  ((current * cryptoAmount) / (count * amount)) * 100 -
                  100
                ).toFixed(2) + "% down"
              );
              this.roi = (((current * cryptoAmount) / (count * amount)) * 100 - 100).toFixed(2);
            }
            let sum: number = 0;
            for (let i = 0; i < myPrices.length; i++) {
              sum += myPrices[i] * 1;
            }
            this.drawChart(myDates, myPrices, accAmounts);
            $("#average").text((sum / myPrices.length).toFixed(2));
            this.averageBuyPrice = (sum / myPrices.length).toFixed(2);
            this.showResults(count, cryptoAmount, amount, coin);
            break;
          }
        } else {
          const response = await fetch(
            `https://api.coingecko.com/api/v3/simple/price?ids=${coin}&vs_currencies=usd`,
            { method: "GET" }
          );
          const data = await response.json();
          const price = await data[coin].usd;
          current = data[coin].usd;
          myPrices.push(price);
          cryptoAmount += amount / data[coin].usd;
          accAmounts.push(cryptoAmount);
          count++;
          $("#coinamount").text(cryptoAmount.toFixed(4));
          this.amount = cryptoAmount.toFixed(4);
          this.currentValue = (current * cryptoAmount).toFixed(2);
          if (((current * cryptoAmount) / (count * amount)) * 100 - 100 > 0) {
            $("#profit").text(
              (
                ((current * cryptoAmount) / (count * amount)) * 100 -
                100
              ).toFixed(2) + "% up"
            );
            this.roi = (((current * cryptoAmount) / (count * amount)) * 100 - 100).toFixed(2);
          } else {
            $("#profit").text(
              (
                ((current * cryptoAmount) / (count * amount)) * 100 -
                100
              ).toFixed(2) + "% down"
            );
            this.roi = (((current * cryptoAmount) / (count * amount)) * 100 - 100).toFixed(2);
          }
          this.showResults(count, cryptoAmount, amount, coin);
          this.drawChart(myDates, myPrices, accAmounts);
          break;
        }
      } while (true);
    },
    async showResults(
      count: number,
      cryptoAmount: number,
      amount: number,
      coin: string
    ): Promise<void> {
      if (count == 0) {
        count = 1;
      }
      $("#times").text(count);
      this.timesInvested = count;
      $("#total").text(count * amount);
      this.totalInvested = count * amount;

      switch (localStorage.getItem("cryptocurrency")) {
        case "bitcoin":
          $(".coin").text("$BTC");
          break;
        case "ethereum":
          $(".coin").text("$ETH");
          break;
        case "tether":
          $(".coin").text("$USDT");
          break;
        case "binancecoin":
          $(".coin").text("$BNB");
          break;
        case "ripple":
          $(".coin").text("$XRP");
          break;
        case "cardano":
          $(".coin").text("$ADA");
          break;
        case "dogecoin":
          $(".coin").text("$DOGE");
          break;
        case "solana":
          $(".coin").text("$SOL");
          break;
        case "polkadot":
          $(".coin").text("$DOT");
          break;
      }
      if (current == NaN) {
        const response = await fetch(
          `https://api.coingecko.com/api/v3/simple/price?ids=${coin}&vs_currencies=usd`,
          { method: "GET" }
        );
        const data = await response.json();
        current = data[coin].usd;
        cryptoAmount += amount / data[coin].usd;
        $("#coinamount").text(cryptoAmount.toFixed(4));
        this.amount = cryptoAmount.toFixed(4);
      }
      if (count != 1) {
        this.longResults = true;
      }

      let startD = (<HTMLInputElement>document.forms[0][0]).value;
      startD =
        startD.substr(8, 10) +
        "/" +
        startD.substring(5, 7) +
        "/" +
        startD.substr(0, 4);
      $("#startdate").text(startD);
      this.show = true;
      this.resultsToShare = $(
        "#result"
      ).text();
    },
    addMilliseconds: (date: any, milliseconds: number) => {
      const result = new Date(date);
      result.setMilliseconds(result.getMilliseconds() + milliseconds);
      return result;
    },
    share(): void {
      let copyText = $("#result").text();
      navigator.clipboard.writeText(copyText);
      $("#copy").attr("title", "Copied!");
    },
    drawChart(
      myDates: string[],
      myPrices: number[],
      accAmounts: number[]
    ): void {
      var data = new google.visualization.DataTable();
      data.addColumn("string", "Date");
      data.addColumn("number", "Value");

      for (var i = 0; i < myDates.length; i++) {
        data.addRow([myDates[i], myPrices[i] * accAmounts[i]]);
      }

      var options: string | number | object = {
        title: "Portfolio Value",
        titleTextStyle: {
          color: "#d6d2de",
          fontName: "Space Grotesk",
          fontSize: 18,
        },
        backgroundColor: "#242535",
        legend: "none",
        hAxis: {
          textStyle: {
            color: "#d6d2de",
            fontName: "Space Grotesk",
          },
          fontName: "Space Grotesk",
        },
        vAxis: {
          minValue: 0,
          viewWindow: { min: 0 },
          textStyle: {
            color: "#d6d2de",
            fontName: "Space Grotesk",
          },
          fontName: "Space Grotesk",
        },
        curveType: "function",
        intervals: { style: "area" },
        lineWidth: 3,
        animation: {
          startup: true,
          duration: 1000,
          easing: "out",
        },
        chartArea: { width: "80%", height: "80%" },
      };

      var chart = new google.visualization.LineChart(
        document.getElementById("curve_chart")!
      );
      chart.draw(data, options);
    },
    addOneMonth(start: any): Date {
      start = new Date(start);
      let month: number = start.getMonth() + 2;
      let year: number = start.getFullYear();
      if (month == 13) {
        month = 1;
        year = start.getFullYear() + 1;
      }
      start = year + "-" + month + "-" + start.getDate();
      start = new Date(start);
      return start;
    },
    addAQuarter(start: any): Date {
      start = new Date(start);
      let month: number = start.getMonth() + 5;
      let year: number = start.getFullYear();
      if (month == 13) {
        month = 1;
        year = start.getFullYear() + 1;
      }
      if (month == 14) {
        month = 2;
        year = start.getFullYear() + 1;
      }
      if (month == 15) {
        month = 3;
        year = start.getFullYear() + 1;
      }
      if (month == 16) {
        month = 4;
        year = start.getFullYear() + 1;
      }
      start = year + "-" + month + "-" + start.getDate();
      start = new Date(start);
      return start;
    },
  },
  mounted() {
    this.load();
  },
});
</script>

<style scoped>
.input-group {
  justify-content: center;
}

.input-group-text {
  margin-right: 0px;
}

.btn-outline-secondary {
  max-height: 40px;
  min-width: 90px;
  max-width: 100px;
}

#curve_chart {
  width: 100%;
  height: 500px;
}
</style>
