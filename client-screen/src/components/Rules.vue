<template>
  <div id="rules">
    <div v-if="step == 'board'">
      <div id="board-title">
        Active Rules
      </div>
      <div id="board">
        <div v-for="rule in rules">
          <div class="rule" :class="{removing: rule.removing}">
            {{ rule.text }}
          </div>
        </div>
      </div>
    </div>
    <div v-if="step == 'new rule'">
      <div id="new-rule" :class="{'flash-text': newRuleStep == 'new rule'}">
        <div v-if="newRuleStep == 'new rule'">
          <div class="flash-text">
            NEW RULE
          </div>
        </div>
        <div v-if="newRuleStep == 'read rule'">
          <TypeAndSay :timeout="40" :doneReading="doneReading">
            {{ newRuleText }}
          </TypeAndSay>
        </div>
      </div>
    </div>
    <div v-if="step == 'closer'">
      <div id="closer" :class="{'flash-text': closerStep == 'closer title'}">
        <div v-if="closerStep == 'closer title'">
          <div class="flash-text">
            CLOSER
          </div>
        </div>
        <div v-if="closerStep == 'read closer'">
          <TypeAndSay :timeout="40" :doneReading="closerFunc">
            {{ closerText }}
          </TypeAndSay>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

import TypeAndSay from './TypeAndSay.vue'
import Strategies from '../utils/Strategies.js'

require('Howler')
var alert01 = new Howl({src:'http://localhost:8082/static/sounds/alert01.ogg'})
var alert02 = new Howl({src:'http://localhost:8082/static/sounds/alert02.ogg'})
var sixBoops = new Howl({src:'http://localhost:8082/static/sounds/sixBoops.ogg'})
var endSong = new Howl({src: 'http://localhost:8082/static/sounds/Who Likes to Party.mp3'})
var heartOfCourage = new Howl({src: 'http://localhost:8082/static/sounds/heartOfCourage.mp3'})

var data = {
  rules: [],
  step: 'board',
  newRuleStep: 'new rule',
  newRuleText: '',
  doneReading() {
    setTimeout(() => {
      console.log('doneSpeaking');
      data.newRuleStep = 'new rule'
      data.step = 'board'
      alert02.play();
    }, 500);
  },
  closerStep: 'closer title',
  closerText: '',
  lastImproviser: {},
  closerFunc() {}
}

function addRule(rule) {
  data.step = 'new rule'
  alert01.play();
  data.rules.push(rule)

  setTimeout(() => {
    data.newRuleText = rule.text
  }, 1500)
  setTimeout(() => {
    data.newRuleStep = 'read rule'
  }, 1400)
}

function removeRule(rule) {
  sixBoops.play();
  setTimeout(() => {
    rule.removing = true;
  }, 0)
  setTimeout(() => {
    rule.removing = false;
  }, 420)
  setTimeout(() => {
    rule.removing = true;
  }, 840)
  setTimeout(() => {
    rule.removing = false;
  }, 1260)
  setTimeout(() => {
    rule.removing = true;
  }, 1680)
  setTimeout(() => {
    data.rules.splice(data.rules.indexOf(rule), 1);
  }, 3100)
}

function removeRandomRule() {
  removeRule(random(data.rules));
}

function random(arr) {
  return arr[parseInt(Math.random() * arr.length)];
}

function randInd(arr) {
  return parseInt(Math.random() * arr.length);
}

function selectRule(rules) {
  let r = Math.random()
  for (var i = 0; i < rules.length; i++) {
    if (rules[i].probCeil > r)
      return i;
  }
}

function generateRule() {
  var names = data.names
  var improvisers = data.improvisers
  var ruleData = data.ruleGens
  // console.log(ruleData)

  var improviser = () => {
    var choice = random(improvisers);
    data.lastImproviser = choice
    return choice.name
  }

  var improviserOrEveryone = () => {
    var choice = random(improvisers.concat('everyone'));
    if (choice === 'everyone') {
      data.lastImproviser = {}
      return choice
    } else {
      data.lastImproviser = choice
      return choice.name
    }
  }

  var uniqueImproviser = () => {
    var avoid = data.lastImproviser ? data.lastImproviser.name : null
    var count = 0;
    var choice = null
    do {
      choice = improviser()
      count++
    } while (count < 20 && choice === avoid)
    return choice
  }

  var getWord = category => {
    return random(ruleData.words[category]);
  }

  var parseRuleGen = (ruleGen) => {
    return eval(ruleGen.template);
  }

  var genderedPronoun = (malePronoun, gender) => {
    var pronouns = {
      he: {
        male: 'he',
        female: 'she',
        generic: 'they'
      },
      him: {
        male: 'him',
        female: 'her',
        generic: 'them'
      },
      his: {
        male: 'his',
        female: 'her',
        generic: 'their'
      }
    }
    return pronouns[malePronoun][gender]
  }

  var him = () => {
    var gender = data.lastImproviser ? data.lastImproviser.gender : 'generic'
    return genderedPronoun('him', gender || 'generic')
  }
  var he = () => {
    var gender = data.lastImproviser ? data.lastImproviser.gender : 'generic'
    return genderedPronoun('he', gender || 'generic')
  }
  var his = () => {
    var gender = data.lastImproviser ? data.lastImproviser.gender : 'generic'
    return genderedPronoun('his', gender || 'generic')
  }

  let selectedRuleIdx = selectRule(ruleData.rules);
  let selectedRule = ruleData.rules[selectedRuleIdx]
  // console.log('selected rule', selectedRule);
  let rule = parseRuleGen(selectedRule);

  data.ruleGens.rules.splice(selectedRuleIdx, 1)
  console.log(rule)
  return {text: rule, removing: false}
}

var closers = [
  {
    message: 'The show is over. Congratulations.',
    func() {
      endSong.play();
    }
  }
]

function closeShow() {
  let closer = random(closers);
  data.closerFunc = closer.func;

  data.step = 'closer';
  alert01.play();

  setTimeout(() => {
    data.closerText = closer.message
  }, 1500)
  setTimeout(() => {
    data.closerStep = 'read closer'
  }, 1400)

}

var closing = false;

function doAction(action) {
  console.log('doAction', action)
  if (closing) return;
  switch (action) {
    case 'default':
      break;
    case 'addRule':
      addRule(generateRule())
      break;
    case 'removeRule':
      removeRandomRule();
      break;
    case 'end show':
      closing = true;
      closeShow()
  }
}

export default {
  name: 'rules',
  props: ['ruleGens', 'names', 'improvisers'],
  data() {
    return data
  },
  components: {
    TypeAndSay
  },
  methods: {
    newRule() {
      sixBoops.play();
      removeRandomRule();
    }
  },
  mounted() {
    data.ruleGens = this.ruleGens;
    data.names = this.names;
    data.improvisers = this.improvisers;
    var strategy = new Strategies.Flip15(10 * 60 * 1000);
    // var strategy = new Strategies.Every15(8 * 60 * 1000);
    // var strategy = new Strategies.JustClose(8 * 60 * 1000);
    setInterval(() => {
      doAction(strategy.getAction())
    }, 1000)

    // var i =0
    // while (i < 100) {
    //   generateRule()
    //   i++
    // }

  }
}
</script>

<style scoped lang="scss">
  $green: #3cff12;

  .title {
    border-bottom: 10px solid $green;
    padding: 15px;
  }

  #rules {
    font-size: 60px;
    padding: 30px;
  }

  #board-title {
    position: absolute;
    top: 30px;
    left: 0;
    height: 8vh;

    font-size: 10vh;
    padding-bottom: 5px;
    margin-left: 30px;
    border-bottom: 1vh solid $green;
  }

  #board {
    position: absolute;
    padding-top: 30px;
    top: 10vh;
    width: 100vw;
    left: 0;
    height: 90vh;
    display: flex;
    flex-wrap: wrap;
    align-content: flex-start;
  }

  .rule {
    border: 10px solid $green;
    padding: 15px;
    margin-top: 30px;
    margin-left: 30px;
    font-size: 50px;
  }

  .removing {
    color: black;
    background-color: $green;
  }

  #new-rule {
    font-size: 120px;
  }

  .flash-text {
    display: flex;
    width: 100vw;
    height: 100vh;
    justify-content: center;
    align-items: center;
    position: absolute;
    top: 0;
    right: 0;
    font-size: 240px;
    animation: flash-text 1s infinite linear;
  }

  @keyframes flash-text{
    1% {background-color: black; color: $green;}

    24% {background-color: black; color: $green;}
    25% {background-color: $green; color: black;}

    49% {background-color: $green; color: black;}
    50% {background-color: black; color: $green;}

    74% {background-color: black; color: $green;}
    75% {background-color: $green; color: black;}

    99% {background-color: $green; color: black;}
    100% {background-color: black; color: $green;}
  }

</style>
