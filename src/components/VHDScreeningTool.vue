<template>
  <v-container class="pa-2"
    style="max-width: 900px; height: 100vh; display: flex; flex-direction: column; overflow: hidden;">
    <v-card rounded="xl" variant="outlined" class="d-flex flex-column" style="height: 100%; max-height: 100%;">
      <!-- HEADER -->
      <v-card-title class="bg-grey-darken-4 text-white py-2">
        <div class="d-flex align-center">
          <v-icon size="14" class="mr-2">mdi-virus</v-icon>
          <span class="text-body-2 font-weight-medium">Viral Hemorrhagic Fever Screening</span>
          <v-spacer></v-spacer>
          <v-chip size="x-small" color="white" variant="outlined" text-color="white">
            {{ currentGuidelineLabel }}
          </v-chip>
        </div>
      </v-card-title>

      <v-card-text class="pa-3 flex-grow-1 d-flex flex-column" style="overflow: hidden;">
        <!-- GUIDELINE TOGGLER - Compact -->
        <v-card variant="tonal" class="mb-2" color="grey-lighten-3">
          <v-card-text class="pa-2">
            <div class="d-flex align-center justify-space-between">
              <div class="text-caption font-weight-medium">
                <v-icon size="14" class="mr-1">mdi-file-document</v-icon>
                Guidelines
              </div>
              <v-btn-toggle v-model="guidelineType" mandatory divided density="compact" color="primary"
                class="guideline-toggle">
                <v-btn value="kenya" size="x-small">
                  <v-icon start size="12">mdi-flag</v-icon> Kenya
                </v-btn>
                <v-btn value="cdc" size="x-small">
                  <v-icon start size="12">mdi-earth</v-icon> CDC
                </v-btn>
              </v-btn-toggle>
            </div>
          </v-card-text>
        </v-card>

        <!-- STEPPER - Flexible height -->
        <v-stepper v-model="step" :items="steps" show-actions class="flex-grow-1 d-flex flex-column"
          style="border: none; min-height: 0;">
          <!-- STEP 1: Symptoms -->
          <template v-slot:item.1>
            <div class="py-1" style="overflow-y: auto; flex: 1;">
              <div class="text-subtitle-2 font-weight-bold mb-1">
                <v-icon size="16" class="mr-1" color="primary">mdi-thermometer</v-icon>
                Symptoms Present Today
              </div>
              <v-alert type="info" variant="tonal" density="compact" class="mb-2 py-1">
                <span class="text-caption">Select all symptoms currently experiencing</span>
              </v-alert>

              <!-- Symptom Chips - Compact -->
              <template v-if="guidelineType === 'kenya'">
                <div class="d-flex flex-wrap ga-1 mb-2">
                  <v-chip v-for="sym in kenyaSymptoms" :key="sym.key"
                    :color="patient.symptoms[sym.key] ? 'warning' : 'grey-lighten-1'"
                    :variant="patient.symptoms[sym.key] ? 'flat' : 'outlined'" @click="toggleSymptom(sym.key)"
                    size="small" class="ma-0">
                    <v-icon v-if="patient.symptoms[sym.key]" start size="10">mdi-check-circle</v-icon>
                    {{ sym.label }}
                  </v-chip>
                </div>
              </template>

              <template v-else>
                <div class="d-flex flex-wrap ga-1 mb-2">
                  <v-chip v-for="sym in cdcSymptoms" :key="sym.key"
                    :color="patient.symptoms[sym.key] ? 'warning' : 'grey-lighten-1'"
                    :variant="patient.symptoms[sym.key] ? 'flat' : 'outlined'" @click="toggleSymptom(sym.key)"
                    size="x-small" class="ma-0">
                    <v-icon v-if="patient.symptoms[sym.key]" start size="10">mdi-check-circle</v-icon>
                    {{ sym.label }}
                  </v-chip>
                </div>
              </template>

              <!-- Temperature - Compact -->
              <v-text-field v-model="temperature" label="Temperature (°C)" type="number" density="compact"
                variant="outlined" prepend-inner-icon="mdi-thermometer" step="0.1" min="35" max="42" hide-details
                class="mt-1" />
              <div class="text-caption text-grey mt-0 mb-1">
                <v-icon size="10">mdi-information</v-icon> Normal: 36.1°C - 37.2°C
              </div>
            </div>
          </template>

          <!-- STEP 2: Risk Assessment -->
          <template v-slot:item.2>
            <div class="py-1" style="overflow-y: auto; flex: 1;">
              <div class="text-subtitle-2 font-weight-bold mb-1">
                <v-icon size="16" class="mr-1" color="warning">mdi-shield-alert</v-icon>
                Risk Assessment (Last 21 Days)
              </div>
              <v-alert type="warning" variant="tonal" density="compact" class="mb-2 py-1">
                <span class="text-caption" v-if="guidelineType === 'kenya'">Kenya BVD risk screening</span>
                <span class="text-caption" v-else>CDC Ebola/Marburg risk assessment</span>
              </v-alert>

              <!-- Risk Questions - Compact -->
              <template v-if="guidelineType === 'kenya'">
                <div v-for="risk in kenyaRiskFields" :key="risk.key"
                  class="d-flex align-center justify-space-between py-1 border-bottom">
                  <div class="mr-2 text-caption" style="flex: 1;">
                    <v-icon size="12" class="mr-1"
                      :color="risks[risk.key] === true ? 'error' : 'grey'">mdi-alert-circle</v-icon>
                    {{ risk.label }}
                  </div>
                  <div class="d-flex" style="min-width: 80px;">
                    <v-btn size="x-small" class="rounded-s-pill"
                      :color="risks[risk.key] === true ? 'error' : 'grey-lighten-1'"
                      :variant="risks[risk.key] === true ? 'flat' : 'tonal'" @click="risks[risk.key] = true"
                      :disabled="risks[risk.key] === true">Yes</v-btn>
                    <v-btn size="x-small" class="rounded-e-pill"
                      :color="risks[risk.key] === false ? 'success' : 'grey-lighten-1'"
                      :variant="risks[risk.key] === false ? 'flat' : 'tonal'" @click="risks[risk.key] = false"
                      :disabled="risks[risk.key] === false">No</v-btn>
                  </div>
                </div>
                <v-text-field v-model="exposureDetails" label="Exposure Details" density="compact" variant="outlined"
                  prepend-inner-icon="mdi-text-box" class="mt-2" hide-details />
              </template>

              <template v-else>
                <div class="text-caption font-weight-medium mb-1">Initial Screening</div>
                <div v-for="risk in cdcRiskFields" :key="risk.key"
                  class="d-flex align-center justify-space-between py-1 border-bottom">
                  <div class="mr-2 text-caption" style="flex: 1;">
                    <v-icon size="12" class="mr-1"
                      :color="risks[risk.key] === true ? 'error' : 'grey'">mdi-alert-circle</v-icon>
                    {{ risk.label }}
                  </div>
                  <div class="d-flex" style="min-width: 80px;">
                    <v-btn size="x-small" class="rounded-s-pill"
                      :color="risks[risk.key] === true ? 'error' : 'grey-lighten-1'"
                      :variant="risks[risk.key] === true ? 'flat' : 'tonal'" @click="risks[risk.key] = true"
                      :disabled="risks[risk.key] === true">Yes</v-btn>
                    <v-btn size="x-small" class="rounded-e-pill"
                      :color="risks[risk.key] === false ? 'success' : 'grey-lighten-1'"
                      :variant="risks[risk.key] === false ? 'flat' : 'tonal'" @click="risks[risk.key] = false"
                      :disabled="risks[risk.key] === false">No</v-btn>
                  </div>
                </div>

                <v-divider class="my-2" />
                <div class="text-caption font-weight-medium mb-1">Public Health Assessment</div>
                <div v-for="risk in cdcAdditionalFields" :key="risk.key"
                  class="d-flex align-center justify-space-between py-1 border-bottom">
                  <div class="mr-2 text-caption" style="flex: 1;">
                    <v-icon size="12" class="mr-1"
                      :color="risks[risk.key] === true ? 'error' : 'grey'">mdi-hospital-building</v-icon>
                    {{ risk.label }}
                  </div>
                  <div class="d-flex" style="min-width: 80px;">
                    <v-btn size="x-small" class="rounded-s-pill"
                      :color="risks[risk.key] === true ? 'error' : 'grey-lighten-1'"
                      :variant="risks[risk.key] === true ? 'flat' : 'tonal'" @click="risks[risk.key] = true"
                      :disabled="risks[risk.key] === true">Yes</v-btn>
                    <v-btn size="x-small" class="rounded-e-pill"
                      :color="risks[risk.key] === false ? 'success' : 'grey-lighten-1'"
                      :variant="risks[risk.key] === false ? 'flat' : 'tonal'" @click="risks[risk.key] = false"
                      :disabled="risks[risk.key] === false">No</v-btn>
                  </div>
                </div>

                <!-- Healthcare Worker Section -->
                <v-expand-transition>
                  <div v-if="risks.healthcareWorker === true">
                    <v-divider class="my-2" />
                    <div class="text-caption font-weight-medium mb-1">Healthcare Worker Assessment</div>
                    <div v-for="risk in cdcHealthcareFields" :key="risk.key"
                      class="d-flex align-center justify-space-between py-1 border-bottom">
                      <div class="mr-2 text-caption" style="flex: 1;">
                        <v-icon size="12" class="mr-1"
                          :color="risks[risk.key] === true ? 'error' : 'grey'">mdi-stethoscope</v-icon>
                        {{ risk.label }}
                      </div>
                      <div class="d-flex" style="min-width: 80px;">
                        <v-btn size="x-small" class="rounded-s-pill"
                          :color="risks[risk.key] === true ? 'error' : 'grey-lighten-1'"
                          :variant="risks[risk.key] === true ? 'flat' : 'tonal'" @click="risks[risk.key] = true"
                          :disabled="risks[risk.key] === true">Yes</v-btn>
                        <v-btn size="x-small" class="rounded-e-pill"
                          :color="risks[risk.key] === false ? 'success' : 'grey-lighten-1'"
                          :variant="risks[risk.key] === false ? 'flat' : 'tonal'" @click="risks[risk.key] = false"
                          :disabled="risks[risk.key] === false">No</v-btn>
                      </div>
                    </div>
                  </div>
                </v-expand-transition>
              </template>

              <div class="d-flex ga-2 mt-2">
                <v-btn size="x-small" color="warning" variant="tonal" prepend-icon="mdi-restore"
                  @click="resetRiskFields">
                  Reset Risks
                </v-btn>
              </div>
            </div>
          </template>

          <!-- STEP 3: Screening Decision -->
          <template v-slot:item.3>
            <div class="py-1" style="overflow-y: auto; flex: 1;">
              <div class="text-subtitle-2 font-weight-bold mb-1">
                <v-icon size="16" class="mr-1" color="success">mdi-clipboard-check</v-icon>
                Screening Decision
              </div>

              <v-expand-transition>
                <div v-if="completionPercentage === 100" class="d-flex flex-column" style="gap: 8px;">
                  <!-- Result Card -->
                  <v-card :color="screeningResult.positive ? 'error-lighten-5' : 'success-lighten-5'" variant="tonal"
                    class="pa-2">
                    <div class="d-flex align-center">
                      <v-icon :color="screeningResult.positive ? 'error' : 'success'" size="28" class="mr-2">
                        {{ screeningResult.positive ? 'mdi-alert-octagon' : 'mdi-check-circle' }}
                      </v-icon>
                      <div>
                        <div class="text-subtitle-1 font-weight-bold" style="line-height: 1.2;">
                          {{ screeningResult.label }}
                        </div>
                        <div class="text-caption">{{ screeningResult.description }}</div>
                        <div class="text-caption text-grey">Based on {{ currentGuidelineLabel }}</div>
                      </div>
                    </div>
                  </v-card>

                  <!-- Actions -->
                  <v-expand-transition>
                    <div v-if="screeningResult.positive">
                      <v-alert type="error" variant="tonal" density="compact" class="py-1">
                        <div class="font-weight-bold text-caption mb-1">
                          <v-icon size="14" class="mr-1">mdi-alert</v-icon> Immediate Actions
                        </div>
                        <ul class="text-caption" style="margin: 0; padding-left: 16px;">
                          <li v-for="action in immediateActions" :key="action">{{ action }}</li>
                        </ul>
                      </v-alert>
                    </div>
                  </v-expand-transition>

                  <!-- Summary -->
                  <v-card variant="outlined" class="pa-2">
                    <div class="text-caption font-weight-bold text-grey">{{ currentGuidelineLabel }} Summary</div>
                    <div class="d-flex ga-1 mt-1">
                      <v-chip size="x-small" color="grey" variant="outlined">
                        {{ totalRisksPresent }} risk{{ totalRisksPresent !== 1 ? 's' : '' }}
                      </v-chip>
                      <v-chip size="x-small" color="grey" variant="outlined">
                        {{ allRisksAnswered ? 'Complete' : 'Incomplete' }}
                      </v-chip>
                    </div>
                  </v-card>
                </div>

                <div v-else class="text-center py-4">
                  <v-icon size="36" color="grey-lighten-2">mdi-clipboard-text-clock</v-icon>
                  <div class="text-subtitle-1 mt-1 text-grey">Screening Incomplete</div>
                  <div class="text-caption text-grey">Complete all steps to view decision</div>
                  <v-btn color="primary" @click="step = 1" variant="tonal" size="small" class="mt-2">
                    Go to Step 1
                  </v-btn>
                </div>
              </v-expand-transition>
            </div>
          </template>
        </v-stepper>

        <!-- RESET BUTTON - Fixed at bottom -->
        <v-btn block color="grey-darken-4" variant="tonal" prepend-icon="mdi-refresh" class="mt-2" size="small"
          @click="resetForm">
          Reset Screening
        </v-btn>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script setup lang="ts">
import { computed, reactive, ref, watch } from 'vue';

type BinaryAnswer = boolean | null;

interface Risks {
  travelledOutbreak: BinaryAnswer;
  contactSick: BinaryAnswer;
  highRiskOccupation: BinaryAnswer;
  healthcareWorkerIll: BinaryAnswer;
  outbreakArea: BinaryAnswer;
  contactSickCDC: BinaryAnswer;
  contactBodyFluids: BinaryAnswer;
  visitClinic: BinaryAnswer;
  touchDeadBody: BinaryAnswer;
  providedCare: BinaryAnswer;
  residedWithSick: BinaryAnswer;
  funeralAttendance: BinaryAnswer;
  healthcareWorker: BinaryAnswer;
  usedPPE: BinaryAnswer;
  physicalContact: BinaryAnswer;
  contaminatedEnvironment: BinaryAnswer;
  unprotectedExposure: BinaryAnswer;
}

interface Patient {
  symptoms: Record<string, boolean>;
}

const step = ref(1);
const guidelineType = ref<'kenya' | 'cdc'>('kenya');
const temperature = ref('');
const exposureDetails = ref('');
const hasSubmitted = ref(false);

const steps = ['Symptoms', 'Risk Assessment', 'Screening Decision'];

const currentGuidelineLabel = computed(() =>
  guidelineType.value === 'kenya' ? 'Kenya BVD' : 'CDC'
);

const patient = reactive<Patient>({
  symptoms: {
    fever: false,
    vomiting: false,
    diarrhea: false,
    abdominalPain: false,
    headache: false,
    bodyAche: false,
    lethargy: false,
    soreThroat: false,
    bleeding: false,
    cough: false,
    difficultyBreathing: false,
  },
});

const risks = reactive<Risks>({
  travelledOutbreak: null,
  contactSick: null,
  highRiskOccupation: null,
  healthcareWorkerIll: null,
  outbreakArea: null,
  contactSickCDC: null,
  contactBodyFluids: null,
  visitClinic: null,
  touchDeadBody: null,
  providedCare: null,
  residedWithSick: null,
  funeralAttendance: null,
  healthcareWorker: null,
  usedPPE: null,
  physicalContact: null,
  contaminatedEnvironment: null,
  unprotectedExposure: null,
});

const kenyaSymptoms = [
  { key: 'fever', label: 'Fever / Body Hotness' },
  { key: 'vomiting', label: 'Vomiting' },
  { key: 'diarrhea', label: 'Diarrhea' },
  { key: 'abdominalPain', label: 'Abdominal Pain' },
  { key: 'headache', label: 'Headache' },
  { key: 'bodyAche', label: 'Body Ache / Weakness' },
  { key: 'lethargy', label: 'Lethargy / Unusual Weakness' },
  { key: 'soreThroat', label: 'Sore Throat' },
  { key: 'bleeding', label: 'Unexplained Bleeding' },
];

const cdcSymptoms = [
  { key: 'fever', label: 'Fever' },
  { key: 'vomiting', label: 'Vomiting' },
  { key: 'diarrhea', label: 'Diarrhea' },
  { key: 'abdominalPain', label: 'Abdominal Pain' },
  { key: 'headache', label: 'Headache' },
  { key: 'bodyAche', label: 'Body Ache' },
  { key: 'lethargy', label: 'Lethargy / Fatigue' },
  { key: 'soreThroat', label: 'Sore Throat' },
  { key: 'bleeding', label: 'Unexplained Bleeding' },
  { key: 'cough', label: 'Cough' },
  { key: 'difficultyBreathing', label: 'Difficulty Breathing' },
];

const kenyaRiskFields = [
  { key: 'travelledOutbreak', label: 'Travelled from/transited DRC, Uganda, or another country reporting Ebola cases?' },
  { key: 'contactSick', label: 'Had contact with a person who had/died after similar symptoms or unexplained bleeding?' },
  { key: 'highRiskOccupation', label: 'Healthcare worker, burial handler, commercial sex worker, hunter, animal handler, forest ranger, or caver?' },
  { key: 'healthcareWorkerIll', label: 'If healthcare worker: became ill after managing a patient with similar symptoms?' },
];

const cdcRiskFields = [
  { key: 'outbreakArea', label: 'Were you in the area where the outbreak is occurring?' },
  { key: 'contactSickCDC', label: 'Contact with or around a person sick with Ebola/Marburg, or compatible illness?' },
  { key: 'contactBodyFluids', label: 'Exposure to blood or other body fluids?' },
  { key: 'visitClinic', label: 'Visit a health clinic or hospital?' },
  { key: 'touchDeadBody', label: 'Touch a dead body or attend a funeral?' },
];

const cdcAdditionalFields = [
  { key: 'providedCare', label: 'Provide direct care to a sick person?' },
  { key: 'residedWithSick', label: 'Reside with a sick person?' },
  { key: 'funeralAttendance', label: 'Attend funeral or burial?' },
  { key: 'healthcareWorker', label: 'Are you a healthcare worker?' },
];

const cdcHealthcareFields = [
  { key: 'usedPPE', label: 'Used recommended PPE during all encounters?' },
  { key: 'physicalContact', label: 'Had physical contact with the patient?' },
  { key: 'contaminatedEnvironment', label: 'Contact with contaminated environment/surfaces?' },
  { key: 'unprotectedExposure', label: 'Unprotected exposure to blood/body fluids?' },
];

const immediateActions = [
  'Immediately separate patient (isolate)',
  'Notify clinician in charge',
  'Notify Infection Prevention & Control (IPC) focal person',
  'Use appropriate personal protective equipment (PPE)',
  'Follow outbreak standard operating procedures (SOP)',
  'Monitor for symptom development for 21 days',
];

const toggleSymptom = (key: keyof Patient['symptoms']) => {
  patient.symptoms[key] = !patient.symptoms[key];
};

const activeSymptomsCount = computed(() =>
  Object.values(patient.symptoms).filter(v => v).length
);

const hasAnySymptom = computed(() =>
  Object.values(patient.symptoms).some(v => v)
);

const hasAnyRisk = computed(() => {
  if (guidelineType.value === 'kenya') {
    const kenyaKeys: (keyof Risks)[] = ['travelledOutbreak', 'contactSick', 'highRiskOccupation', 'healthcareWorkerIll'];
    return kenyaKeys.some(key => risks[key] === true);
  } else {
    const cdcKeys: (keyof Risks)[] = ['outbreakArea', 'contactSickCDC', 'contactBodyFluids', 'visitClinic', 'touchDeadBody'];
    return cdcKeys.some(key => risks[key] === true);
  }
});

const totalRisksPresent = computed(() => {
  if (guidelineType.value === 'kenya') {
    const kenyaKeys: (keyof Risks)[] = ['travelledOutbreak', 'contactSick', 'highRiskOccupation', 'healthcareWorkerIll'];
    return kenyaKeys.filter(key => risks[key] === true).length;
  } else {
    const cdcKeys: (keyof Risks)[] = ['outbreakArea', 'contactSickCDC', 'contactBodyFluids', 'visitClinic', 'touchDeadBody'];
    return cdcKeys.filter(key => risks[key] === true).length;
  }
});

const allRisksAnswered = computed(() => {
  if (guidelineType.value === 'kenya') {
    const kenyaKeys: (keyof Risks)[] = ['travelledOutbreak', 'contactSick', 'highRiskOccupation', 'healthcareWorkerIll'];
    return kenyaKeys.every(key => risks[key] !== null);
  } else {
    const cdcKeys: (keyof Risks)[] = ['outbreakArea', 'contactSickCDC', 'contactBodyFluids', 'visitClinic', 'touchDeadBody'];
    return cdcKeys.every(key => risks[key] !== null);
  }
});

const completionPercentage = computed(() => {
  let total = 3;
  let completed = 0;
  if (activeSymptomsCount.value > 0) completed++;
  if (temperature.value && temperature.value.trim() !== '') completed++;
  if (allRisksAnswered.value) completed++;
  return Math.round((completed / total) * 100);
});

const isPositive = computed(() => {
  return hasAnySymptom.value && hasAnyRisk.value;
});

const screeningResult = computed(() => {
  if (isPositive.value) {
    return {
      positive: true,
      label: 'SCREEN POSITIVE / BVD ALERT',
      description: 'Patient meets criteria for BVD alert - Symptoms + Risk History Present',
    };
  }
  if (hasAnySymptom.value && !hasAnyRisk.value) {
    return {
      positive: false,
      label: 'SCREEN NEGATIVE',
      description: 'Symptoms present but no risk history - Continue monitoring',
    };
  }
  return {
    positive: false,
    label: 'SCREEN NEGATIVE',
    description: 'No symptoms and no risk factors - Routine care',
  };
});

const resetRiskFields = () => {
  Object.keys(risks).forEach(key => {
    risks[key as keyof Risks] = null;
  });
  exposureDetails.value = '';
  hasSubmitted.value = false;
};

const resetForm = () => {
  temperature.value = '';
  exposureDetails.value = '';
  step.value = 1;
  Object.keys(patient.symptoms).forEach(key => {
    patient.symptoms[key as keyof Patient['symptoms']] = false;
  });
  Object.keys(risks).forEach(key => {
    risks[key as keyof Risks] = null;
  });
  hasSubmitted.value = false;
};

watch(
  [() => activeSymptomsCount.value, temperature, () => allRisksAnswered.value],
  ([symptoms, temp, risksAnswered]) => {
    if (symptoms > 0 && temp && temp.trim() !== '' && risksAnswered) {
      step.value = 3;
    }
  },
  { deep: true }
);

watch(guidelineType, () => {
  resetRiskFields();
  step.value = 1;
});
</script>

<style scoped>
.v-btn {
  text-transform: none;
}

ul {
  padding-left: 16px;
  margin: 0;
}

.border-bottom {
  border-bottom: 1px solid rgba(0, 0, 0, 0.06);
}

.border-bottom:last-child {
  border-bottom: none;
}

:deep(.v-stepper) {
  box-shadow: none;
  background: transparent;
}

:deep(.v-stepper-header) {
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.04);
  padding: 4px 8px;
}

:deep(.v-stepper-actions) {
  padding: 4px 0;
}

:deep(.v-stepper-item) {
  padding: 4px 0;
}

:deep(.v-stepper-item .v-avatar) {
  width: 24px;
  height: 24px;
  font-size: 12px;
}

:deep(.v-stepper-item .v-stepper-item__title) {
  font-size: 10px;
}

.guideline-toggle :deep(.v-btn) {
  text-transform: none;
  font-weight: 500;
  font-size: 10px;
}

:deep(.v-card-text) {
  padding: 8px;
}

:deep(.v-alert) {
  font-size: 11px;
}

:deep(.v-chip) {
  font-size: 10px;
  height: 20px;
}

:deep(.v-text-field) {
  font-size: 12px;
}

:deep(.v-text-field input) {
  font-size: 12px;
}
</style>