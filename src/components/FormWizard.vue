<template>
  <div>
    <div v-if="wizardInProgress" v-show="asyncState !== 'pending'">
      <keep-alive>
        <component :is="currentStep" ref="currentStep" @update="processStep" :wizard-data="form" @updateAsyncState="updateAsyncState"></component>
      </keep-alive>
      <div class="progress-bar">
        <div :style="`width: ${progress}%;`"></div>
      </div>

      <!-- Actions -->
      <div class="buttons">
        <button @click="goBack" v-if="currentStepNumber > 1" class="btn-outlined">Back
        </button>
        <button @click="goNext" :disabled="!canGoNext" class="btn">{{isLastStep ? "Complete order" : "Next"}}</button>
      </div>

      <pre><code>{{form}}</code></pre>
    </div>
    <div v-else>
      <h1 class="title">Thank You</h1>
      <h2 class="subtitle">We are looking forward to shipping your order!</h2>
      <p class="text-center"><a href="https://github.com/niyorn" class="btn">Go somewhere cool</a></p>
    </div>

    <div class="loading-wrapper" v-if="asyncState === 'pending'">
      <div class="loader">
        <img src="/spinner.svg" alt="">
        <p>Please wait, we're hitting our servers!</p>
      </div>
    </div>
  </div>
</template>

<script>
import {postFormToDB} from '../api'
import FormPlanPicker from './FormPlanPicker'
import FormUserDetails from './FormUserDetails'
import FormAddress from './FormAddress'
import FormReviewOrder from './FormReviewOrder'
export default {
  name: 'FormWizard',
  components: {
    FormPlanPicker,
    FormUserDetails,
    FormAddress,
    FormReviewOrder
  },
  data () {
    return {
      currentStepNumber: 1,
      canGoNext: false,
      asyncState: null,
      steps: [
        'FormPlanPicker',
        'FormUserDetails',
        'FormAddress',
        'FormReviewOrder'
      ],
      form: {
        plan: null,
        email: null,
        name: null,
        password: null,
        address: null,
        recipient: null,
        chocolate: false,
        otherTreat: false
      }
    }
  },
  computed: {
    length() {
      return this.steps.length
    },
    progress () {
      return this.currentStepNumber/this.length * 100
    },
    currentStep() {
      return this.steps[this.currentStepNumber -1]
    },
    isLastStep() {
      return this.currentStepNumber === this.length
    },
    wizardInProgress() {
      return this.currentStepNumber <= this.length
    }
  },
  validation: {},
  methods: {
    updateAsyncState(state) {
      this.asyncState = state
    },
    processStep(step) {
      Object.assign(this.form, step.data)
      this.canGoNext = step.valid
    },
    goBack () {
      this.currentStepNumber--
      this.canGoNext = true
    },
    goNext () {
      this.currentStepNumber++
      // this.canGoNext = false
      this.$nextTick(()=>{
        this.canGoNext = !this.$refs.currentStep.$v.$invalid
      })
    },
    nextButton() {
      if(this.isLastStep) {
        this.submitOrder()
      } else {
        this.goNext()
      }
    },
    submitOrder() {
    this.asyncState = 'pending'
    postFormToDB(this.form)
      .then(() => {
        console.log('form submitted', this.form)
        this.asyncState = 'success'
        this.currentStepNumber++
      })
    }
  }
}
</script>
