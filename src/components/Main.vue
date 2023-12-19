<script lang="ts" setup>
import {ref} from 'vue';

//constants
const PARAGON_WEIGHT = 100;
const FAT_CAMP = ref<FatFuck[]>([]);
const FATTIES_COUNTER = 12;
const isFatter = ref<boolean>(Math.random() < 0.5);
const weightDiff = ref<1 | -1>();
const imposter = ref<number>();

//statics
const attempts = ref<number>(3);

//dynamics
const leftSide = ref<FatFuck[]>([]);
const rightSide = ref<FatFuck[]>([]);
const activeSide = ref<boolean>(false);
const weighInResult = ref<0 | 1 | 2>(0);

//results
const answerTarget = ref<number>();
const answerSide = ref<boolean>(false);
const answerResult = ref<boolean>(false);
const complete = ref<boolean>(false);

interface FatFuck {
  _diff: boolean,
  _weight: number,
  _id: number;
  _used: boolean;
}

class Fatty {
  _weight: number;
  _diff: boolean;
  _id: number = 0;
  _used: boolean = false;

  constructor(weight: number) {
    this._weight = weight;
    this._diff = this._weight !== PARAGON_WEIGHT;
  }
}

//decides which item will have different weight
function getRandomInt(max: number) {
  return Math.floor(Math.random() * max);
}

//set item on chosen side
const setFatty = (id: number) => {
  if (!activeSide.value) {
    leftSide.value.push(FAT_CAMP.value[id])
  } else {
    rightSide.value.push(FAT_CAMP.value[id])
  }
  FAT_CAMP.value[id]._used = true;
}

//remove item from the chosen side
const removeFatty = (id: number) => {
  let bufferArr: FatFuck[] = [];
  if (!activeSide.value) {
    bufferArr = leftSide.value;
    leftSide.value = bufferArr.filter(item => item._id !== id);
    FAT_CAMP.value[id - 1]._used = false;
  } else {
    bufferArr = rightSide.value;
    rightSide.value = bufferArr.filter(item => item._id !== id);
    FAT_CAMP.value[id - 1]._used = false;
  }
}

//calculate weight difference between sides
const weighIn = () => {
  let leftWeight = 0;
  let rightWeight = 0;
  leftSide.value.forEach(item => {
    leftWeight += item._weight;
  })
  rightSide.value.forEach(item => {
    rightWeight += item._weight;
  })
  attempts.value -= 1;
  leftSide.value = [];
  rightSide.value = [];
  FAT_CAMP.value.forEach(item => {
    item._used = false;
  })
  if (leftWeight === rightWeight) weighInResult.value = 0;
  if (leftWeight > rightWeight) weighInResult.value = 1;
  if (rightWeight > leftWeight) weighInResult.value = 2;
}

//calculate result depending on user's answer
const calculateResult = () => {
  if (!imposter.value) return;
  if (imposter.value + 1 === answerTarget.value) {
    if (weightDiff.value === -1 && !answerSide.value) {
      answerResult.value = true;
    } else answerResult.value = (weightDiff.value === 1 && answerSide.value);
  } else answerResult.value = false;
  complete.value = true;
}

onMounted(() => {
  imposter.value = getRandomInt(FATTIES_COUNTER);
  weightDiff.value = isFatter.value ? 1 : -1;
  for (let i = 0; i < FATTIES_COUNTER; i++) {
    let bufferWeight = i === imposter.value ? PARAGON_WEIGHT + weightDiff.value : PARAGON_WEIGHT;
    FAT_CAMP.value?.push(new Fatty(bufferWeight));
    FAT_CAMP.value[i]._id = i + 1;
  }
})
</script>

<template>
  <div class="wrapper">
    <div class="row">
      <div class="main">
        <h1 class="main-title">
          SeeSaw puzzle
        </h1>
        <div class="main-subtitle">
          <p class="main-subtitle-text">
            Взвешиваний осталось: {{ attempts }}
          </p>
        </div>
        <div class="main-result">
          <template v-if="!complete">
            <p v-if="weighInResult === 0">
              Стороны равны
            </p>
            <p v-if="weighInResult === 1">
              Левая сторона тяжелее
            </p>
            <p v-if="weighInResult === 2">
              Правая сторона тяжелее
            </p>
          </template>
          <template v-if="complete">
            <p v-if="answerResult">
              Верно!
            </p>
            <p v-if="!answerResult">
              Неверно.
            </p>
          </template>
        </div>
        <div class="main-picture">
          <img src="@/public/scales.svg" alt="scales">
        </div>
        <div class="main-box">
          <button
              v-for="(item, index) in FAT_CAMP"
              :key="index"
              class="main-box-item"
              :disabled="item._used"
              @click="setFatty(index)"
          >
            {{ item._id }}
          </button>
        </div>
        <div class="main-sides">
          <div class="main-sides-item" :class="{'active': !activeSide}" @click="activeSide = false">
            <div v-for="(item, index) in leftSide" :key="index" class="main-sides-fatty" @click="removeFatty(item._id)">
              <p>
                {{ item._id }}
              </p>
            </div>
          </div>
          <div class="main-sides-item" :class="{'active': activeSide}" @click="activeSide = true">
            <div v-for="(item, index) in rightSide" :key="index" class="main-sides-fatty"
                 @click="removeFatty(item._id)">
              <p>
                {{ item._id }}
              </p>
            </div>
          </div>
        </div>
        <div class="main-controls">
          <button v-if="attempts" class="main-controls-weigh" @click="weighIn()">
            Взвесить
          </button>
          <div v-if="!attempts" class="main-controls-answer">
            <div class="main-controls-answer--input">
              <label for="answer-number">
                Номер участника
              </label>
              <input v-model="answerTarget" id="answer-number" type="number">
            </div>
            <div class="main-controls-answer--radio">
              <input type="radio" id="lighter" :value="false" v-model="answerSide">
              <label for="lighter">Легче</label>
              <input type="radio" id="heavier" :value="true" v-model="answerSide">
              <label for="heavier">Тяжелее</label>
            </div>
            
            <button class="main-controls-answer--button" @click="calculateResult()">
              Дать ответ
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.wrapper {
  display: flex;
  padding: 15px 30px;

  .row {
    display: flex;
    flex-flow: row wrap;
    flex: 1;

    .main {
      display: flex;
      flex-flow: column;
      flex: 1;
      text-align: center;

      &-title {
        margin: 0;
      }

      &-result {
        margin-top: 10px;
        font-size: 18px;
      }

      &-picture {
        max-width: 100px;
        display: flex;
        margin: 30px auto;

        img {
          width: 100%;
        }
      }

      &-box {
        display: flex;
        flex-flow: row wrap;
        gap: 18px;
        justify-content: center;

        &-item {
          border: 1px solid #6d6d6d;
          border-radius: 24px;
          padding: 6px 12px;
          display: flex;
          align-items: center;
          justify-content: center;
          cursor: pointer;
          transition: color, background-color 0.4s;

          &:hover {
            background: #4c4c4c;
            color: #ffffff;
          }

          &:disabled {
            pointer-events: none;
          }
        }
      }

      &-sides {
        padding: 30px;
        max-width: 960px;
        width: 100%;
        margin: 0 auto;
        display: grid;
        grid-template-columns: 1fr 1fr;
        grid-template-rows: 250px;

        &-item {
          border: 1px solid #c2c2c2;
          transition: border-color 0.4s;
          padding: 12px;
          display: flex;
          flex-flow: row wrap;
          align-items: flex-start;
          align-content: flex-start;
          gap: 12px;

          .main-sides-fatty {
            pointer-events: none;
          }

          &.active {
            border-color: #000000;

            .main-sides-fatty {
              pointer-events: all;
            }
          }
        }

        &-fatty {
          border: 1px solid #6d6d6d;
          border-radius: 24px;
          padding: 6px 12px;
          display: flex;
          align-items: center;
          justify-content: center;
          cursor: pointer;
          transition: color, background-color 0.4s;

          &:hover {
            background: #4c4c4c;
            color: #ffffff;
          }
        }
      }

      &-controls {
        margin: 0 auto;

        &-answer {
          display: flex;
          flex-flow: column;
          gap: 6px;

          &--input {
            display: flex;
            gap: 8px;
            align-items: center;
          }

          &--radio{
            padding: 15px 0;
          }
        }
      }
    }
  }
}
</style>