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
              explanation: trial.competitor_response
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
            v-model="$magpie.measurements.answer"
            style="width: 500px; height: 200px"
          />

          <button
            v-if="
              $magpie.measurements.answer &&
              $magpie.measurements.answer.length > 3
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
      console.log(Object.keys(trial))
      if(_.includes(Object.keys(trial), 'mostSimilar')) {
        var itemOrder = _.shuffle(['competitor', 'sameCategory', 'otherCategory', 'mostSimilar'])
      } else {
        var itemOrder = _.shuffle(['competitor', 'sameCategory', 'otherCategory'])
      }
      console.log(itemOrder)
      var vignette_start = trial.vignette_start
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
        var response = trial.competitor_response //trial.answer
        var character = character_response.replace(" replies:", "").toLowerCase().trim();
        console.log(character)
        var slide_text = [vignette_start, context, vignette_continuation, "\"".concat(question).concat("\""), "\\n\\n", character_response, ['"', response.trim(), '"'].join(""), "\\n\\n", "Please explain to Bo why ", [character, "'s"].join(""), " response is reasonable, i.e., which thoughts would justify mentioning ", trial.competitor, " instead of mentioning something else."].join(" ");
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