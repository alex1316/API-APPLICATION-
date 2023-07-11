<template>
 <div class="container" >
    <form class="currency-form" @submit.prevent="getCurrentBitcoinValue">
      <label for="currency-select">Choose currency:</label>
      <select id="currency-select" v-model="currency">
        <option value="EUR">EUR</option>
        <option value="USD">USD</option>
      </select>
      <button type="submit">Get Current Value</button>
    </form>
    <p class="current-value">Current Bitcoin value: {{ currentValue }} {{ currency }}</p>
    <p class="average-value">Average value for the last 2 minutes: {{ averageValue.twoMinutes }} {{ currency }}</p>
    <p class="average-value">Average value for the last 5 minutes: {{ averageValue.fiveMinutes }} {{ currency }}</p>
    <p class="average-value">Average value for the last 30 minutes: {{ averageValue.thirtyMinutes }} {{ currency }}</p>
    <p class="average-value">Average value for the last hour: {{ averageValue.hour }} {{ currency }}</p>
    
  </div>
</template>

<script>
import axios from 'axios';


export default {
  data() {
    return {
      bitcoinValues: [],
      currentValue: null,
      averageValue: {
        twoMinutes: null,
        fiveMinutes: null,
        thirtyMinutes: null,
        hour: null,
      },
      currency: 'EUR',
    }
  },
  
    
   mounted() {
    // Call the API every minute for an hour
    let count = 0;
    const intervalId = setInterval(() => {
      axios.get(`https://api.coinbase.com/v2/prices/spot?currency=${this.currency}`)
        .then(response => {
          const value = parseFloat(response.data.data.amount);
          this.currentValue = value;
          this.bitcoinValues.push(value);
          this.calculateAverage();
          
          count++;
          if (count === 60) {
            clearInterval(intervalId);
          }
        })
        .catch(error => {
          console.log(error);
        });
    }, 60000);

    
    // Calculate the average every 2 minutes
    setInterval(() => {
      this.averageValue.twoMinutes = this.calculateAverage(2);
    }, 120000);

    // Calculate the average every 5 minutes
    setInterval(() => {
      this.averageValue.fiveMinutes = this.calculateAverage(5);
    }, 300000);

    // Calculate the average every 30 minutes
    setInterval(() => {
      this.averageValue.thirtyMinutes = this.calculateAverage(30);
    }, 1800000);

    // Calculate the average every hour
    setInterval(() => {
      this.averageValue.hour = this.calculateAverage(60);
    }, 3600000);
  },
  methods: {
    calculateAverage(minutes) {
      const numValues = minutes * 60;
      const values = this.bitcoinValues.slice(-numValues);
      const sum = values.reduce((a, b) => a + b, 0);
      return (sum / values.length).toFixed(2);
    },
    getCurrentBitcoinValue() {
      axios.get(`https://api.coinbase.com/v2/prices/spot?currency=${this.currency}`)
        .then(response => {
          this.currentValue = parseFloat(response.data.data.amount);
        })
        .catch(error => {
          console.log(error);
        });
    },

  

    
},
};
</script>

<style>
  .container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 80vh;
}

.current-value {
  font-size: 30px;
  color: green;
}

.average-value {
  font-size: 17px;
  color: red;
}

</style>

