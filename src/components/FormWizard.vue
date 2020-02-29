<template>
  <div>
    <div v-if="wizardInProgress" v-show="asyncState !== 'pending'">
      <keep-alive>
        <component ref="currentStep" :is="currentStep" @update="processStep" :wizard-data="form"></component>
      </keep-alive>
      <div class="progress-bar">
        <div :style="`width: ${progress}%;`"></div>
      </div>

      <!-- Actions -->
      <div class="buttons">
        <button @click="goBack" v-if="currentStepNumber > 1" class="btn-outlined">Voltar</button>
        <button
          @click="nextButtonAction"
          :disabled="!canGoNext"
          class="btn"
        >{{ isLastStep ? 'Completar Pedido' : 'Avançar' }}</button>
      </div>
    </div>
    <div v-else>
      <h1 class="title">Obrigado!</h1>
      <h2 class="subtitle">Estamos ansiosos para enviar sua primeira caixa!</h2>

      <p class="text-center">
        <a href="#" target="_blank" class="btn">Vá a algum lugar legal</a>
      </p>
    </div>
    <div class="loading-wrapper" v-if="asyncState === 'pending'">
      <div class="loader">
        <img src="/spinner.svg" alt />
        <p>Porfavor aguarde...</p>
      </div>
    </div>
  </div>
</template>

<script>
import { postFormToDB } from "../api";
import FormPlanPicker from "./FormPlanPicker";
import FormUserDetails from "./FormUserDetails";
import FormAddress from "./FormAddress";
import FormReviewOrder from "./FormReviewOrder";
export default {
  name: "FormWizard",
  components: {
    FormPlanPicker,
    FormUserDetails,
    FormAddress,
    FormReviewOrder
  },
  data() {
    return {
      currentStepNumber: 1,
      canGoNext: false,
      asyncState: null,
      steps: [
        "FormPlanPicker",
        "FormUserDetails",
        "FormAddress",
        "FormReviewOrder"
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
    };
  },
  computed: {
    isLastStep() {
      return this.currentStepNumber === this.length;
    },
    wizardInProgress() {
      return this.currentStepNumber <= this.length;
    },
    currentStep() {
      return this.steps[this.currentStepNumber - 1];
    },
    length() {
      return this.steps.length;
    },
    progress() {
      return (this.currentStepNumber / this.length) * 100;
    }
  },
  methods: {
    submitOrder() {
      this.asyncState = "pending";
      postFormToDB(this.form).then(() => {
        console.log("ok");
        this.currentStepNumber++;
        this.asyncState = "success";
      });
    },
    nextButtonAction() {
      if (this.isLastStep) {
        this.submitOrder();
      } else {
        this.goNext();
      }
    },
    processStep(step) {
      Object.assign(this.form, step.data);
      this.canGoNext = step.valid;
    },
    goBack() {
      this.currentStepNumber--;
      this.canGoNext = true;
    },
    goNext() {
      this.currentStepNumber++;
      this.$nextTick(() => {
        this.canGoNext = !this.$refs.currentStep.$v.$invalid;
      });
    }
  }
};
</script>
