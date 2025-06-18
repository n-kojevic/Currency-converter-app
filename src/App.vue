<template>
  <div id="app">
    <!-- App Title -->
    <h1>Currency Converter</h1>

    <form @submit.prevent="convertCurrency">

  <!-- Amount Input -->
  <input
    type="number"
    v-model.number="amount"
    required
  />

  <!-- From Currency Select + Flag -->
  <div class="select-group">
    <select v-model="fromCurrency" required>
      <option v-for="currency in currencies" :key="currency" :value="currency">
        {{ currency }}
      </option>
    </select>
    <img :src="getFlagUrl(fromCurrency)" alt="flag" class="flag" />
  </div>

  <!-- To Currency Select + Flag -->
  <div class="select-group">
    <select v-model="toCurrency" required>
      <option v-for="currency in currencies" :key="currency" :value="currency">
        {{ currency }}
      </option>
    </select>
    <img :src="getFlagUrl(toCurrency)" alt="flag" class="flag" />
  </div>

  <!-- Buttons: Convert and Swap -->
  <button type="submit">Convert</button>
  <button type="button" @click="swapCurrencies" class="swap-btn">↔ Swap Currencies</button>
</form>


    <!-- Conversion Result -->
    <div v-if="result !== null" class="result">
      Result: {{ result }}
    </div>

    <!-- Conversion History List -->
    <div v-if="history.length" class="history">
      <h2>Conversion History:</h2>
      <ul>
        <li v-for="(item, index) in history" :key="index">
          <img :src="getFlagUrl(item.from)" alt="flag" class="flag" />
          {{ item.amount }} {{ item.from }} →
          <img :src="getFlagUrl(item.to)" alt="flag" class="flag" />
          {{ item.result }} {{ item.to }}
        </li>
      </ul>
    </div>
  </div>
</template>


<script>
export default {
  data() {
    return {
      // App state
      history: [],              // List of previous conversions
      currencies: [],           // Loaded list of available currencies
      fromCurrency: 'EUR',      // Selected source currency
      toCurrency: 'USD',        // Selected target currency
      amount: 1,                // Amount to convert
      result: null              // Converted result
    };
  },

  // Load currency codes when component is mounted
  mounted() {
    this.loadCurrencies();
  },

  methods: {
    // Fetch available currency codes
    async loadCurrencies() {
      const url = `https://v6.exchangerate-api.com/v6/c3f1e058f1cc0bc538af3d98/codes`;
      try {
        const res = await fetch(url);
        const data = await res.json();
        if (data.result === 'success') {
          this.currencies = data.supported_codes.map(item => item[0]);
          this.currencies.sort(); // Sort alphabetically
        }
      } catch (e) {
        console.error("Error loading currencies:", e);
      }
    },

    // Swap the selected currencies
    swapCurrencies() {
      const temp = this.fromCurrency;
      this.fromCurrency = this.toCurrency;
      this.toCurrency = temp;
    },

    // Perform the currency conversion and update history
    async convertCurrency() {
      const url = `https://v6.exchangerate-api.com/v6/c3f1e058f1cc0bc538af3d98/pair/${this.fromCurrency}/${this.toCurrency}/${this.amount}`;
      try {
        const res = await fetch(url);
        const data = await res.json();

        if (data.conversion_result !== undefined) {
          this.result = data.conversion_result.toFixed(2);
          this.history.unshift({
            amount: this.amount,
            from: this.fromCurrency,
            to: this.toCurrency,
            result: this.result
          });
        } else {
          this.result = "Conversion error";
        }
      } catch (error) {
        console.error("Fetch error:", error);
        this.result = "An error occurred";
      }
    },

    // Return the URL for the currency flag icon
    getFlagUrl(code) {
      const exceptions = {
        EUR: 'eu', USD: 'us', RSD: 'rs', GBP: 'gb', CHF: 'ch', JPY: 'jp', AUD: 'au', CAD: 'ca',
        NOK: 'no', SEK: 'se', DKK: 'dk', CNY: 'cn', RUB: 'ru', PLN: 'pl', HUF: 'hu', CZK: 'cz',
        TRY: 'tr', BGN: 'bg', HRK: 'hr', BAM: 'ba', MKD: 'mk', INR: 'in', ZAR: 'za', MXN: 'mx',
        BRL: 'br', NZD: 'nz', SGD: 'sg', HKD: 'hk', KRW: 'kr', THB: 'th', MYR: 'my', IDR: 'id',
        PHP: 'ph', EGP: 'eg', SAR: 'sa', AED: 'ae', KWD: 'kw', ILS: 'il', ARS: 'ar', CLP: 'cl',
        COP: 'co', VND: 'vn', UAH: 'ua', IQD: 'iq'
      };

      const countryCode = exceptions[code] || code.slice(0, 2).toLowerCase();
      return `https://flagcdn.com/w40/${countryCode}.png`;
    }
  }
};
</script>


<style>
/* Base app styling */
#app {
  max-width: 420px;
  margin: 60px auto;
  padding: 40px 30px;
  background: linear-gradient(145deg, #ffffff, #e6f0ff);
  border-radius: 16px;
  box-shadow: 8px 8px 20px #b0b8d8, -8px -8px 20px #ffffff;
  font-family: 'Poppins', 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  color: #2c3e50;
  user-select: none;
}

/* Title */
h1 {
  font-weight: 900;
  font-size: 2rem;
  text-align: center;
  margin-bottom: 35px;
  color: #1a2733;
  letter-spacing: 1.1px;
}

/* Form layout */
form {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

/* Inputs and selects */
input[type="number"], select {
  padding: 14px 18px;
  font-size: 1.1rem;
  border-radius: 12px;
  border: 2.5px solid transparent;
  background: #f9faff;
  box-shadow: inset 3px 3px 6px #d1d9f0, inset -3px -3px 6px #ffffff;
  transition: border-color 0.35s ease, box-shadow 0.35s ease;
  font-weight: 600;
  color: #2c3e50;
}

/* Focus effect */
input[type="number"]:focus, select:focus {
  outline: none;
  border-color: #3a86ff;
  box-shadow: inset 3px 3px 6px #d1d9f0, inset -3px -3px 6px #ffffff, 0 0 8px #3a86ffaa;
  background: #ffffff;
}

/* Button style */
button {
  padding: 15px 0;
  font-size: 1.2rem;
  font-weight: 800;
  color: #fff;
  background: linear-gradient(90deg, #3a86ff, #005fcb);
  border: none;
  border-radius: 16px;
  cursor: pointer;
  box-shadow: 0 5px 15px rgba(58, 134, 255, 0.4);
  transition: background 0.3s ease, box-shadow 0.3s ease;
  user-select: none;
}

button:hover {
  background: linear-gradient(90deg, #005fcb, #3a86ff);
  box-shadow: 0 8px 20px rgba(58, 134, 255, 0.7);
}

/* Conversion result */
.result {
  margin-top: 40px;
  font-size: 1.5rem;
  font-weight: 900;
  text-align: center;
  color: #3a86ff;
  letter-spacing: 0.05em;
  user-select: text;
  word-break: break-word;
}

/* History block */
.history {
  margin-top: 30px;
  padding-top: 20px;
  border-top: 1px solid #ccc;
  font-size: 1rem;
  color: #555;
}

.history h2 {
  font-weight: 700;
  color: #1a2733;
  margin-bottom: 15px;
}

.history ul {
  list-style: none;
  padding-left: 0;
}

.history li {
  padding: 6px 0;
  border-bottom: 1px solid #eee;
}

/* Flag image */
.flag {
  width: 30px;
  height: 20px;
  margin-left: 8px;
  vertical-align: middle;
  border: 1px solid #ccc;
  border-radius: 3px;
}

/* Select + flag layout */
.select-group {
  display: flex;
  align-items: center;
  gap: 10px;
}

/* Swap currencies button */
.swap-btn {
  padding: 10px;
  font-size: 1rem;
  font-weight: bold;
  color: #3a86ff;
  background: none;
  border: 2px solid #3a86ff;
  border-radius: 12px;
  cursor: pointer;
  transition: background 0.3s ease, color 0.3s ease;
}

.swap-btn:hover {
  background: #3a86ff;
  color: white;
}

/* Responsive design */
@media (max-width: 480px) {
  #app {
    margin: 30px 20px;
    padding: 30px 20px;
  }

  input[type="number"], select {
    font-size: 1rem;
    padding: 12px 14px;
  }

  button {
    font-size: 1.1rem;
    padding: 14px 0;
  }

  .result {
    font-size: 1.3rem;
  }
}
</style>
