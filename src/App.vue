<template>
  <b-container>
    <h1 class="text-center my-5">PTO Calculator</h1>
    <b-row class="mb-3">
      <b-col>
        <b-form-group label="Period length (days)"
          description="How many days are in one pay period?">
          <b-form-input v-model="numDaysInPeriod" type="number" min="0"/>
        </b-form-group>
      </b-col>
      <b-col>
        <b-form-group label="Hours per period"
          description="How many hours of PTO do you accrue each pay period?">
          <b-form-input v-model="hoursPerPeriod" type="number" min="0"/>
        </b-form-group>
      </b-col>
      <b-col>
        <b-form-group label="Current accrued PTO (hours)"
          description="How many PTO hours do you currently have available?">
          <b-form-input v-model="currentHours" type="number" min="0"/>
        </b-form-group>
      </b-col>
      <b-col>
        <b-form-group label="Additional PTO (hours)"
          description="Do you have any additional PTO, such as flex time, holidays, etc?">
          <b-form-input v-model="additionalHours" type="number" min="0"/>
        </b-form-group>
      </b-col>
    </b-row>
    <b-row class="mb-4">
      <b-col>
        <b-form-group label="Last period end date"
          description="When did the last pay period end?">
          <b-form-input v-model="lastPeriodEnd" type="date"/>
        </b-form-group>
      </b-col>
      <b-col>
        <b-form-group label="Target date"
          description="What date should these calculation target?">
          <b-form-input v-model="targetDate" type="date" 
            :min="lastPeriodEnd" :disabled="isTargetDateDisabled"/>
        </b-form-group>
      </b-col>
    </b-row>
    <b-row>
      <b-col>
        <table class="table table-light text-center">
          <thead>
            <tr>
              <th>Total Current Hours</th>
              <th>Hours Accrued by Target Date</th>
              <th>Total PTO Hours</th>
              <th>Total PTO Days</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>{{ currentHoursString }}</td>
              <td>{{ accruedHoursString }}</td>
              <td>{{ totalPtoHoursString }}</td>
              <td>{{ totalPtoDaysString }}</td>
            </tr>
          </tbody>
        </table>
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
const PRECISION = 3;

export default {
  name: 'App',
  data() {
    return {
      additionalHours: 0,
      currentHours: 0,
      hoursPerPeriod: 0,
      lastPeriodEnd: null,
      numDaysInPeriod: 14,
      targetDate: null,
    }
  },
  methods: {
    dateDiff(dayOne, dayTwo) {
      const diffTime = Math.abs(dayTwo - dayOne);
      const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));

      return diffDays;
    },
    hoursToDays(hours) {
      return hours / 8.0;
    }
  },
  computed: {
    today() { return new Date() },
    isTargetDateDisabled() {
      return this.lastPeriodEnd == null ||
        this.numDaysInPeriod == 0 ||
        this.hoursPerPeriod == 0;
    },
    showTotalHours() {
      return !this.isTargetDateDisabled && this.targetDate != null;
    },
    totalCurrentHours() {
      return Number(this.currentHours) + Number(this.additionalHours);
    },
    lastPeriodEndDate() {
      return this.lastPeriodEnd == null ? null : new Date(this.lastPeriodEnd);
    },
    targetDateDate() {
      return this.targetDate == null ? null : new Date(this.targetDate);
    },
    totalPtoHours() {
      if (this.lastPeriodEnd == null || this.targetDate == null) {
        return this.totalCurrentHours;
      }

      const numDaysIntoCurrentPeriod = this.dateDiff(this.today, this.lastPeriodEndDate);
      const numDaysRemainingInCurrentPeriod = this.numDaysInPeriod - numDaysIntoCurrentPeriod;
      const endOfPeriod = this.today.setDate(this.today.getDate() + numDaysRemainingInCurrentPeriod);

      if (endOfPeriod > this.targetDateDate) {
        return this.totalCurrentHours;
      }

      const numPeriodsUntilTargetDate = Math.floor(this.dateDiff(this.targetDateDate, endOfPeriod) / this.numDaysInPeriod) + 1;
      const hoursToBeAccrued = numPeriodsUntilTargetDate * this.hoursPerPeriod;

      return Number(hoursToBeAccrued + this.totalCurrentHours);
    },
    currentHoursString() {
      return this.totalCurrentHours.toFixed(PRECISION);
    },
    accruedHoursString() {
      return Number(this.totalPtoHours - this.totalCurrentHours).toFixed(PRECISION);
    },
    totalPtoHoursString() {
      return Number(this.totalPtoHours).toFixed(PRECISION);
    },
    totalPtoDaysString() {
      return Number(this.hoursToDays(this.totalPtoHours)).toFixed(PRECISION);
    }
  }
}
</script>

<style>
</style>
