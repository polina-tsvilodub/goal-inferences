<template>
  <Experiment title="answer-explanation-experiment">

    <InstructionScreen :title="'Welcome'">
      Thank you for taking part in our experiment! 
      <br />
      You are participating in a study conducted by cognitive scientists at the University of Tübingen. 
      Your participation in this research is voluntary.
      You may decline to answer any or all of the following questions.
      You may decline further participation, at any time, without adverse consequences.
      <br />
      <br />
      Your anonymity is assured; the researchers who have requested your
      participation will not receive any personal information about you.
      <br />
      <br />
      By pressing the button 'Next' you confirm that you are at least 18 years old and agree to participate in this study. 
    </InstructionScreen>
    <InstructionScreen :title="'Instructions'">
      In the following, you will see short descriptions of scenes that you could observe in everyday life. In each scene, one
      person asks a question and another person answers it. Your task is to explain <b>why, intuitively, you think the answerer provided the specific answer</b> they did.
      <br />
      <br />
      Notice that there will also be simple <b>attention checking</b> trials. 
      You will recognize them immediately when you read the important text on each trial carefully -- those trials contain instructions for you to type a certain word in the textbox. 
      Please spell the words exactly as stated in the instructions. 
      <br />
      <br />
      Please respond naturally and reasonably. <br />
      Please avoid jokes, insults or otherwise making the dialogues into
      something else than simple, harmless exchanges of information.
    </InstructionScreen>

    <template v-for="(trial, i) in trials">
     <template v-if="!trial.type">
      <FreetypingScreen :key=i :trial=trial :trial_type="'main'" :index=i :progress="i / trials.length" />     
     </template>
     <template v-else>
      <FreetypingScreen :key=i :trial=trial :trial_type="'filler'" :index=i :progress="i / trials.length" />     
     </template>
    </template>

    <PostTestScreen />

    <SubmitResultsScreen />

  </Experiment>
</template>

<script>
import _ from 'lodash';
import items from '../trials/items.csv';
import trialsAll from '../trials/cs2_human_competitors.csv';
import fillersAll from '../trials/fillers_split_cogsci.csv';
import FreetypingScreen from './FreeTypingScreen';

var group = _.sample(['odd', 'even']);

const n_vignettes = 4;
const n_fillers = 1;

const selected_items = 
  group == 'odd'
    ? items.filter((element, index) => {
        return index % 2 === 0;
      })
    : items.filter((element, index) => {
        return index % 2 != 0;
      });
// select itemName only
const sampled_items = _.sampleSize(selected_items, n_vignettes).map(
  (element) => {
    return element.itemName;
  }
)
console.log(sampled_items)

// select one trial per sampled item
const trials = _.shuffle(
  _.map(sampled_items, (itemName) => {
    return _.sample(
      trialsAll.filter((trial) => {
        return trial.itemName === itemName;
      })
    );
  })
);
console.log(trials.length)


const fillers =
  group == 'odd'
    ? fillersAll.filter((element, index) => {
        return index % 2 === 0;
      })
    : fillersAll.filter((element, index) => {
        return index % 2 != 0;
      });

// Disable selecting text
// Supported by the following browsers:
// https://caniuse.com/mdn-api_document_selectstart_event
document.onselectstart = () => false;

// Disable context menu
// Supported by the following browsers:
// https://caniuse.com/mdn-api_element_contextmenu_event
document.oncontextmenu = () => false;

export default {
  name: 'App',
  components: { FreetypingScreen },
  data() {
    return {
      trials: _.shuffle(_.concat( trials, _.sampleSize(fillers, n_fillers)))
    };
  },
  computed: {
    // Expose lodash to template code
    _() {
      return _;
    }
  }
};
</script>
<style>
body {
  /*
  Disable selecting text via css
  Supported by the following browsers
  https://caniuse.com/mdn-css_properties_user-select
  */
  user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  -webkit-user-select: none;
}
</style>