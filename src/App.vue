<template>
  <b-container>
    <h1 class="text-center mt-3">PTO Calculator</h1>
    <b-row class="mb-3">
      <b-col>
        <b-form-group label="Period length (days)"
          description="How many days are in one pay period?">
          <b-form-input v-model="numDaysInPeriod" type="number" min="0"/>
        </b-form-group>
      </b-col>
      <b-col>
        <b-form-group label="Hours per period"
          description="How many hours do you accrue each pay period?">
          <b-form-input v-model="hoursPerPeriod" type="number" min="0"/>
        </b-form-group>
      </b-col>
    </b-row>
    <b-row class="mb-3">
      <b-col>
        <b-form-group label="Current period start date"
          description="When did this most recent pay period begin?">
          <b-form-input v-model="currentPeriodStart" type="date"/>
        </b-form-group>
      </b-col>
      <b-col>
        <b-form-group label="Current accrued PTO (hours)"
          description="How many PTO hours do you currently have available?">
          <b-form-input v-model="currentHours" type="number" min="0"/>
        </b-form-group>
      </b-col>
    </b-row>
    <b-row class="mb-3">
      <b-col>
        <b-form-group label="Additional PTO (hours)"
          description="Do you have any additional PTO, such as flex time, holidays, etc?">
          <b-form-input v-model="additionalHours" type="number" min="0"/>
        </b-form-group>
      </b-col>
    </b-row>
    <b-row class="mb-3">
      <b-col>
        <b-form-group label="Target date"
          description="What date should these calculation target?">
          <b-form-input v-model="targetDate" type="date" :min="currentPeriodStart" :disabled="isTargetDateDisabled"/>
        </b-form-group>
      </b-col>
    </b-row>
    <b-row class="text-center">
      <b-col class="mt-5 mb-3" cols="12">
        <h3>Results</h3>
      </b-col>
      <b-col>
        <p>Current Hours:<br/>{{ currentHoursString }}</p>
      </b-col>
      <b-col>
        <p>Accrued by Target Date:<br/>{{ accruedHoursString }}</p>
      </b-col>
      <b-col>
        <p>Total Hours:<br/>{{ totalPtoHoursString }}</p>
      </b-col>
      <b-col>
        <p>Total Days:<br/>{{ totalPtoDaysString }}</p>
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
    totalCurrentHours() {
      return Number(this.currentHours) + Number(this.additionalHours);
    },
    totalPtoHours() {
      if (this.currentPeriodStart == null || this.targetDate == null) {
        return this.totalCurrentHours;
      }

      const numDaysIntoCurrentPeriod = this.isSameDate(this.today, new Date(this.currentPeriodStart))
        ? 0 : this.dateDiff(this.today, new Date(this.currentPeriodStart));
      const numDaysRemainingInCurrentPeriod = this.numDaysInPeriod - numDaysIntoCurrentPeriod;
      const endOfPeriod = this.today.setDate(this.today.getDate() + numDaysRemainingInCurrentPeriod);

      if (endOfPeriod > new Date(this.targetDate)) {
        return this.totalCurrentHours;
      }

      const numPeriodsUntilTargetDate = Math.floor(this.dateDiff(new Date(this.targetDate), endOfPeriod) / this.numDaysInPeriod) + 1;
      const hoursToBeAccrued = numPeriodsUntilTargetDate * this.hoursPerPeriod;

      return Number(hoursToBeAccrued) + this.totalCurrentHours;
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
