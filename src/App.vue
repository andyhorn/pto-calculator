<template>
  <b-container>
    <h1 class="text-center my-5">PTO Calculator</h1>
    <b-row class="mb-3">
      <b-col>
        <b-form-group label="Period length (days)">
          <b-form-input v-model="numDaysInPeriod" type="number" min="0"/>
        </b-form-group>
      </b-col>
      <b-col>
        <b-form-group label="Hours per period">
          <b-form-input v-model="hoursPerPeriod" type="number" min="0"/>
        </b-form-group>
      </b-col>
    </b-row>
    <b-row class="mb-3">
      <b-col>
        <b-form-group label="Current period start date">
          <b-form-input v-model="currentPeriodStart" type="date"/>
        </b-form-group>
      </b-col>
      <b-col>
        <b-form-group label="Current accrued PTO (hours)">
          <b-form-input v-model="currentHours" type="number" min="0"/>
        </b-form-group>
      </b-col>
    </b-row>
    <b-row class="mb-3">
      <b-col>
        <b-form-group label="Additional PTO (hours)">
          <b-form-input v-model="additionalHours" type="number" min="0"/>
        </b-form-group>
      </b-col>
    </b-row>
    <b-row class="mb-3">
      <b-col>
        <b-form-group label="Target date">
          <b-form-input v-model="targetDate" type="date" :min="currentPeriodStart"/>
        </b-form-group>
      </b-col>
    </b-row>
    <b-row class="text-center">
      <b-col class="mt-3 mb-5" cols="12">
        <h3>Totals</h3>
      </b-col>
      <b-col>
        <p>Current: {{ currentHoursString }}</p>
      </b-col>
      <b-col>
        <p>Accrued: {{ accruedHoursString }}</p>
      </b-col>
      <b-col>
        <p>Hours: {{ totalPtoHoursString }}</p>
      </b-col>
      <b-col>
        <p>Days: {{ totalPtoDaysString }}</p>
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      additionalHours: 0,
      currentHours: 0,
      currentPeriodStart: null,
      hoursPerPeriod: 0,
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
    isSameDate(dayOne, dayTwo) {
      return dayOne.getYear() == dayTwo.getYear() &&
        dayOne.getMonth() == dayTwo.getMonth() &&
        dayOne.getDate() == dayTwo.getDate();
    },
    hoursToDays(hours) {
      return hours / 8.0;
    }
  },
  computed: {
    today() { return new Date() },
    isTargetDateDisabled() {
      return this.currentPeriodStart == null ||
        this.numDaysInPeriod == 0 ||
        this.hoursPerPeriod == 0;
    },
    showTotalHours() {
      return !this.isTargetDateDisabled && this.targetDate != null;
    },
    totalPtoHours() {
      const currentHours = Number(this.currentHours) + Number(this.additionalHours);

      if (this.currentPeriodStart == null || this.targetDate == null) {
        return currentHours;
      }

      const numDaysIntoCurrentPeriod = this.isSameDate(this.today, new Date(this.currentPeriodStart))
        ? 0 : this.dateDiff(this.today, new Date(this.currentPeriodStart));
      const numDaysRemainingInCurrentPeriod = this.numDaysInPeriod - numDaysIntoCurrentPeriod;
      const endOfPeriod = this.today.setDate(this.today.getDate() + numDaysRemainingInCurrentPeriod);

      if (endOfPeriod > new Date(this.targetDate)) {
        return currentHours;
      }

      const numPeriodsUntilTargetDate = Math.floor(this.dateDiff(new Date(this.targetDate), endOfPeriod) / this.numDaysInPeriod) + 1;
      const hoursToBeAccrued = numPeriodsUntilTargetDate * this.hoursPerPeriod;

      return Number(hoursToBeAccrued) + Number(currentHours);
    },
    currentHoursString() {
      return Number(Number(this.currentHours) + Number(this.additionalHours)).toFixed(2);
    },
    accruedHoursString() {
      return Number(this.totalPtoHours - Number(this.currentHours) - Number(this.additionalHours)).toFixed(2);
    },
    totalPtoHoursString() {
      return Number(this.totalPtoHours).toFixed(2);
    },
    totalPtoDaysString() {
      return Number(this.hoursToDays(this.totalPtoHours)).toFixed(2);
    }
  }
}
</script>

<style>
</style>
