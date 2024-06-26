<template>
  <div>
    <div class="header">
      <h1>{{ stepHeader }}</h1>
      <div v-if="displayErrorMessage">
        <p class="step-error-message">fill in all required fields</p>
      </div>
      <div v-else class="step-error-message-placeholder"></div>
    </div>
    <div class="content">
      <EntryFieldValues
        v-if="currentStep !== Steps.Summary"
        :step-data="stepData"
      />
      <Summary v-else :user-data="userData" />
    </div>
    <div class="footer">
      <div>
        <button class="form-btn" v-if="showBackButton" @click="handleBackClick">
          {{ backButtonLabel }}
        </button>
      </div>
      <div>
        <button class="form-btn" @click="handleNextClick">
          {{ nextButtonLabel }}
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import EntryFieldValues from "./steps/EntryFieldValues.vue";
import Summary from "@/components/form/steps/Summary";
import Steps from "@/js/Steps.js";
import UserData from "@/js/UserData.js";
import Vue from "vue";
import { Component } from "vue-property-decorator";

@Component({
  components: {
    Summary,
    EntryFieldValues,
  },
})
export default class FormLayout extends Vue {
  data() {
    return {
      Steps: Steps,
      currentStep: Steps.ClientData,
      userData: new UserData(),
      displayErrorMessage: false,
    };
  }

  get backButtonLabel() {
    return "back";
  }

  get nextButtonLabel() {
    return this.currentStep === Steps.Summary ? "send" : "next";
  }

  /**
   *
   * @returns {boolean}
   */
  get showBackButton() {
    return this.currentStep !== Steps.ClientData;
  }

  /**
   * @returns{StepData}
   */
  get stepData() {
    if (this.currentStep === Steps.ClientData) {
      return this.userData.clientData;
    } else if (this.currentStep === Steps.Evaluations) {
      return this.userData.evaluations;
    }
    return undefined;
  }

  /**
   *
   * @returns {string}
   */
  get stepHeader() {
    switch (this.currentStep) {
      case Steps.ClientData:
        return this.userData.clientData.getLabel();
      case Steps.Evaluations:
        return this.userData.evaluations.getLabel();
      case Steps.Summary:
        return this.userData.getLabel();
      default:
        return "Client Data";
    }
  }

  handleBackClick() {
    if (this.showBackButton) {
      this.currentStep--;
    }
  }

  handleNextClick() {
    const valid = this.isValidStep();
    this.displayErrorMessage = !valid;
    if (valid && this.currentStep < 2) {
      this.currentStep++;
    } else if (this.currentStep === 2) {
      this.fakeRequest()
        .then((data) => console.log(data))
        .catch((error) => console.log(error, error.status));
    }
  }

  /**
   *
   * @returns {boolean}
   */
  isValidStep() {
    return this.userData.isValidStepData(this.currentStep);
  }

  fakeRequest() {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        const error = new Error("Page Not Found");
        error.status = 404;
        reject(error);
      }, 1000);
    });
  }
}
</script>

<style>
.content {
  margin: 0 auto;
  max-width: 600px;
  gap: 32px;
}

.header {
  margin-bottom: 8px;
}

.footer {
  display: flex;
  justify-content: center;
  gap: 16px;
}

.form-btn {
  padding: 12px 36px;
  background-color: blue;
  border: 0;
  border-radius: 100px;
  color: white;
  font-size: 16px;
}

.step-error-message {
  font-size: 16px;
  color: red;
  margin: 0;
  padding: 0;
}

.step-error-message-placeholder {
  min-height: 18px;
}
</style>
