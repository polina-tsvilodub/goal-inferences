<!-- FreetypingScreen.vue -->
<template>
    <Screen :progress="progress">
        <Slide>
          <Record
            :data="{
              trialNr: index + 1,
              itemName: trial.itemName,
              settingName: trial.settingName,
              trial_type: trial_type,
              correct_response: trial.correct_response,
              explanation: trial.answer,
              question: trial.question
            }"
          />

          <span
            v-for="(line, lineNumber) of createScreen.split('\\n')"
            :key="lineNumber"
          >
            {{ line }}<br />
          </span>
          <br />
          <textarea
            v-model="$magpie.measurements.answer1"
            style="width: 500px; height: 50px"
          />
          <textarea
            v-model="$magpie.measurements.answer2"
            style="width: 500px; height: 50px"
          />
          <textarea
            v-model="$magpie.measurements.answer3"
            style="width: 500px; height: 50px"
          />

          <button
            v-if="
              $magpie.measurements.answer1 &&
              $magpie.measurements.answer1.length > 3 &&
              $magpie.measurements.answer2 &&
              $magpie.measurements.answer2.length > 3 &&
              $magpie.measurements.answer3 &&
              $magpie.measurements.answer3.length > 3
            "
            @click="$magpie.saveAndNextScreen()"
          >
            Submit
          </button>
        </Slide>
      </Screen>
</template>

<script>
import _ from 'lodash';

function createText(trial){
      // shuffle the order of the alternatives
      var itemOrder = _.shuffle(['competitor', 'sameCategory', 'otherCategory'])
      console.log(itemOrder)
      var vignette_start = trial.vignette_start
      // only use the context without options: drop last chunk after a full stop, concat the senstences before again
      var vignette_start_clean = vignette_start.split(".").slice(0, -1).join(".")
      var vignette_full = [vignette_start_clean, trial.vignette_continuation].join(".")
      var vignette_continuation = trial.vignette_continuation
      var question = trial.question
      var character_response = trial.character_template
      console.log(trial.correct_response)
      var context = itemOrder.map(x => trial[x])
      context.splice(-1, 1, "and ".concat(context.at(-1)));
      var context = context.join(", ").concat(".");
      if(trial.correct_response != "main"){
        var slide_text = [vignette_start, context, vignette_continuation, "\"".concat(question).concat("\""), "\\n\\n"].join(" ");
      } else {
        var response = trial.answer
        var character = character_response.replace(" replies:", "").toLowerCase();
        console.log(character)
        // vignette_start, context, vignette_continuation
        var slide_text = [vignette_full, "\"".concat(question).concat("\""), "\\n\\n", "Please name at least three plausible goals that ", trial.questioner, " might have in mind when asking their question."].join(" ");
      }
      return slide_text
};

export default {
    name: 'FreeTypingScreen',
    props: {
        trial: {
          type: Object,
          required: true
        },
        trial_type: {
          type: String,
          required: true
        },
        index: {
          type: Number,
          required: true
        },
        progress: {
          type: Number,
          default: undefined
        }
    },
    methods: {
      createText
    },
    computed: {
      createScreen(){
        return createText(this.trial)
      }
    }
};  
</script>