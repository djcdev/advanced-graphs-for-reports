<!-- NetworkGraphForm.vue -->
<template>
    <form>
      <!-- Your form elements for gathering the required parameters -->
      <instrument-selector
        v-model="formData.instrument"
        :availableInstruments="availableInstruments"
        ></instrument-selector>
        <div v-if="formData.instrument !== null && typeof formData.instrument === 'string'">
            <div class="AG-two-panes">
              <div class="AG-pane-left">
              <!-- Title -->
              <helpful-parameter
                  :label-text="module.tt('title')"
                  :help-text="module.tt('title_help')">
                  <input type="text" v-model="formData.title">
              </helpful-parameter>
              <!-- Graph Type: network -->
                  <helpful-parameter
                          :label-text="module.tt('graph_type')"
                          :help-text="module.tt('graph_type_help')"
                  >
                      <radio-component
                              v-model="formData.graph_type"
                              :name="'graph_type'"
                              :values="['network_graph']"
                              :defaultValue="'network_graph'"
                              :labels="[module.tt('network_graph')]"
                      ></radio-component>
                  </helpful-parameter>
              <!-- Description -->
              <helpful-parameter
                  :label-text="module.tt('description')"
                  :help-text="module.tt('description_help')">
                  <input type="text" v-model="formData.description">
              </helpful-parameter>
                  <!-- Numeric Field X-->
                  <helpful-parameter
                          :label-text="module.tt('numeric_field_x')"
                          :help-text="module.tt('numeric_field_help')"
                  >
                      <numeric-field-selector
                              v-model="formData.numeric_field"
                              :fields="report_fields_by_repeat_instrument[formData.instrument].fields"
                      ></numeric-field-selector>
                  </helpful-parameter>
              </div>
            <div class="AG-pane-right">
              <!-- Numeric Field Y-->
              <helpful-parameter
                  :label-text="module.tt('numeric_field_y')"
                  :help-text="module.tt('numeric_field_help')"
              >
              <numeric-field-selector
                      v-model="formData.numeric_field_y"
                      :fields="report_fields_by_repeat_instrument[formData.instrument].fields"
                  ></numeric-field-selector>
              </helpful-parameter>
              <!-- NA Numeric -->
              <helpful-parameter
                  v-if="formData.numeric_field !== null
                    && typeof formData.numeric_field === 'string'
                    && formData.is_count != true"
                  :label-text="module.tt('na_numeric')"
                  :help-text="module.tt('na_numeric_help')"
              >
                <radio-component
                        v-model="formData.na_numeric"
                        :name="'na_numeric'"
                        :values="['drop', 'replace']"
                        :defaultValue="'drop'"
                        :labels="[module.tt('drop'), module.tt('replace')]"
                    ></radio-component>
                <input type="number" v-if="formData.na_numeric === 'replace'" v-model="formData.na_numeric_value" />
              </helpful-parameter>
               <!-- Palette -->
              <helpful-parameter
                  :label-text="module.tt('palette')"
                  :help-text="module.tt('palette_help')"
              >
                <palette-selector
                        v-model="formData.palette_brewer"
                    ></palette-selector>
              </helpful-parameter>
            </div>
          </div>
        </div>
      </form>
  </template>

  <script>
  import HelpfulParameter from "@/components/HelpfulParameter.vue";
  import InstrumentSelector from "@/components/InstrumentSelector.vue";
  import RadioComponent from "@/components/RadioComponent.vue";
  //import CategoricalFieldSelector from "@/components/CategoricalFieldSelector.vue";
  import NumericFieldSelector from "@/components/NumericFieldSelector.vue";
  import PaletteSelector from "@/components/PaletteSelector.vue";
  //import { isCategoricalField } from "@/utils";
  import { isNumericField } from "@/utils";

  export default {
    components: {
      HelpfulParameter,
      InstrumentSelector,
      RadioComponent,
    //  CategoricalFieldSelector,
      NumericFieldSelector,
      PaletteSelector,
    },
    inject: ["module", "report_fields_by_repeat_instrument"],
    props: {
      cellData: {
        type: Object,
        default: null,
      },
    },
    mounted() {
      this.$emit("isReady", this.isFormReady);
    },
    data() {
      return {
        formData: this.cellData || {},
      };
    },
    computed: {
      isFormReady() {
        // console.log('formData', this.formData);
        const instrument_selected = (typeof this.formData.instrument !== 'undefined') && this.formData.instrument !== null;
        const graph_type_selected = (typeof this.formData.graph_type !== 'undefined') && this.formData.graph_type !== null;
        // const categorical_field_selected = (typeof this.formData.categorical_field !== 'undefined') && this.formData.categorical_field !== null;
          const numeric_field_selected = (typeof this.formData.numeric_field !== 'undefined') && this.formData.numeric_field !== null;
          const numeric_field_y_selected = (typeof this.formData.numeric_field_y !== 'undefined') && this.formData.numeric_field_y !== null;
        //  const na_category_selected = (typeof this.formData.na_category !== 'undefined') && this.formData.na_category !== null;
        // const unused_categories_selected = (typeof this.formData.unused_categories !== undefined) && this.formData.unused_categories !== null;
        // const aggregation_function_selected = (typeof this.formData.aggregation_function !== 'undefined') && this.formData.aggregation_function !== null;
        // console.log('isFormReady', instrument_selected, graph_type_selected, categorical_field_selected, numeric_field_selected, na_category_selected, 'unused', unused_categories_selected, 'agg', aggregation_function_selected);
        // If any of the required fields are not selected, return false
        if (!instrument_selected || !graph_type_selected || !numeric_field_selected || !numeric_field_y_selected ) { //|| !unused_categories_selected) {
          return false;
        }

        // If na_numeric is replace, but no value is provided, return false
        if (this.formData.na_numeric === 'replace' && !this.formData.na_numeric_value) {
          return false;
        }

        // If numeric field is selected, but no aggregation function is selected, return false
        // if (numeric_field_selected && typeof this.formData.numeric_field === 'string' && this.formData.is_count != true && !aggregation_function_selected) {
        //   return false;
        // }

        // console.log('isFormReady', true);
        // Return true if there are enough selected parameters to create a bar or pie graph
        return true;
      },
      availableInstruments() {
        const instruments = {};
        for (const instrument in this.report_fields_by_repeat_instrument) {
          if (this.instrumentCanCreate(this.report_fields_by_repeat_instrument, instrument)) {
            instruments[instrument] = this.report_fields_by_repeat_instrument[instrument];
          }
        }
        return instruments;
      },
    },
    watch: {
      isFormReady(newVal) {
        // Emit an event with the new status whenever the conditions change
        this.$emit("isReady", newVal);
        // console.log('isReady', newVal);
      },
      formData: {
        handler(newVal) {
          newVal.is_count = newVal.numeric_field === "";

          // Emit an event with the new data whenever the form data changes
          this.$emit("updateCellData", newVal);
        },
        deep: true,
      },
    },
    methods: {
      isCreatable(report_fields_by_repeat_instrument) {
        // For each instrument in report_fields_by_reapeat_instrument
        for (const instrument in report_fields_by_repeat_instrument) {
          // If there is a numeric field
          if (this.instrumentCanCreate(report_fields_by_repeat_instrument, instrument)) {
            // Return true
            return true;
          }
        }
      },
      instrumentCanCreate(report_fields_by_repeat_instrument, instrument_name) {
        // If there is a categorical field
        if (report_fields_by_repeat_instrument[instrument_name].fields.some(field => isNumericField(field))) {
          // Return true
          return true;
        }
        return false;
      }
    }
  };
  </script>

  <style scoped>
    .AG-two-panes {
      display: flex;
      flex-direction: row;
      justify-content: space-between;
    }

    .AG-pane-left {
      width: 50%;
    }

    .AG-pane-right {
      width: 50%;
      margin-left: 20px;
    }

  </style>
