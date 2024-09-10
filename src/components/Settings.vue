<template>
  <!-- Loan -->
  <div>
    <label for="loan">Nuvarande bolån</label>
  </div>
  <div>
    <input type="number" name="loan" id="loan" v-model="loan" />
  </div>

  <!-- Rent -->
  <div>
    <label for="rent">Nuvarande ränta</label>
  </div>
  <div>
    <input type="number" name="rent" id="rent" v-model="rentInPercentage" />
  </div>

  <!-- Market Price -->
  <div>
    <label for="market-value">Bostadens marknadsvärde</label>
  </div>
  <div>
    <input
      type="number"
      name="market-value"
      id="market_value"
      v-model="market_value"
    />
  </div>

  <!-- Income -->
  <div>
    <label for="income">Årsinkomst</label>
  </div>
  <div>
    <input type="number" name="income" id="income" v-model="income" />
  </div>

  <table>
    <thead>
      <tr>
        <th>År</th>
        <th>Nuvarande lån</th>
        <th>Nuvarande ränta</th>
        <th>Ränta kr/år</th>
        <th>Amortering %</th>
        <th>Amortering kr/år</th>
        <th>Lån efter amortering</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="year in years" :key="year">
        <td>{{ new Date().getFullYear() + year }}</td>
        <td>{{ formatPrice(currentLoan(year)) }} kr</td>
        <td>{{ rentInPercentage }} %</td>
        <td>{{ formatPrice(rentCost(currentLoan(year))) }} kr</td>
        <td>{{ currentAmortizationPercentage(currentLoan(year)) }} %</td>
        <td>
          {{ formatPrice(amortizationAmount(currentLoan(year), year)) }} kr
        </td>
        <td>
          {{
            formatPrice(
              currentLoan(year) - amortizationAmount(currentLoan(year), year)
            )
          }}
          kr
        </td>
      </tr>
    </tbody>
  </table>
</template>

<script>
export default {
  data() {
    return {
      loan: 5000000, // The current loan amount
      rentInPercentage: 4, // The interest rate in percentage
      market_value: 6500000, // Market value of the property
      income: 600000, // Annual income
      years: 50, // Number of years to display
    };
  },

  methods: {
    formatPrice(value) {
      let val = (value / 1).toFixed(0).replace(".", ",");
      return val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".");
    },

    // Calculate the interest cost for a given year
    rentCost(loan) {
      return loan * (this.rentInPercentage / 100);
    },

    // Calculate the amortization percentage based on LTV and income
    currentAmortizationPercentage(current_loan) {
      const loan = current_loan;
      const ltv = loan / this.market_value; // Loan-to-value ratio
      const incomeRule = this.income * 4.5; // Income rule

      let percentage = 0;

      if (loan > incomeRule) {
        console.log("loan", loan);
        console.log("incomeRule", incomeRule);
        percentage += 1;
      }

      if (ltv > 0.5 && ltv <= 0.7) {
        percentage += 1;
      } else if (ltv > 0.7) {
        percentage += 2;
      }

      return percentage;
    },

    // Calculate the current loan after amortization for a given year
    currentLoan(year) {
      let loan = this.loan;
      let amortizationRate = this.currentAmortizationPercentage(loan) / 100;

      // For each year, reduce the loan by the amortization amount
      for (let i = 1; i < year; i++) {
        let amortization = loan * amortizationRate;
        loan -= amortization;
      }

      return loan > 0 ? loan : 0; // Ensure loan doesn't go negative
    },

    // Calculate the amortization amount for a given year
    amortizationAmount(loan, year) {
      let amortizationRate = this.currentAmortizationPercentage(loan) / 100;
      return loan * amortizationRate;
    },
  },
};
</script>
