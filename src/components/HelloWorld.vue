<template>
  <div class="container">
    <h1 class="title">Базовый алгоритм для расчета рейтинга игроков</h1>

    <article class="article">
      <p class="article__title">Расчет производится по простой формуле:</p>
      <p>W = 150, KO = 100, L= 50, LKO = 100</p>
      <p>(W + KO) - (L + LKO) = PTS</p>
    </article>

    <h2 class="subtitle">Рейтинг игроков</h2>

    <ul class="playerlist" v-click-outside="disabledTrue">
      <li>
        <span>Nickname</span>
        <span>Win</span>
        <span>KO</span>
        <span>Looses</span>
        <span>PTS</span>
      </li>

      <li v-for="(player, index) in sortPlayersArr" :key="player.name">
        <span>
          <input
            type="text"
            v-model="sortPlayersArr[index].name"
            :disabled="inpDisabledArr[index].isDisabled"
          />
        </span>
        <span>
          <input
            type="text"
            :index="index"
            fieldType="win"
            v-model="sortPlayersArr[index].win"
            @input="resort"
            :disabled="inpDisabledArr[index].isDisabled"
          />
        </span>
        <span>
          <input
            type="text"
            :index="index"
            fieldType="ko"
            v-model="sortPlayersArr[index].ko"
            @input="resort"
            :disabled="inpDisabledArr[index].isDisabled"
          />
        </span>
        <span>
          <input
            type="text"
            :index="index"
            fieldType="loose"
            v-model="sortPlayersArr[index].loose"
            @input="resort"
            :disabled="inpDisabledArr[index].isDisabled"
          />
        </span>
        <span>{{ player.pts }}</span>
        <div class="playerlist__action">
          <button class="btn" :counter="index" @click="showFields">
            <img :src="penImage" />
          </button>
          <button class="btn" :counter="index" @click="spliceArr">
            <img :src="deleteImage" />
          </button>
        </div>
      </li>
    </ul>

    <div class="addnewplayer">
      <h2 class="subtitle">Добавить игрока</h2>

      <ul class="addplayer">
        <li>
          <div class="addplayer__title">
            <label for="nickinp">Nickname</label>
          </div>
          <input
            class="field"
            type="text"
            id="nickinp"
            v-model="formFields.name"
            :class="{ error: !validateFields.name }"
          />
        </li>
        <li>
          <div class="addplayer__title">
            <label for="wininp">Win</label>
          </div>
          <input
            class="field"
            type="text"
            id="wininp"
            v-model="formFields.win"
            :class="{ error: !validateFields.win }"
            @input="formValidate"
            keyTitle="win"
          />
        </li>
        <li>
          <div class="addplayer__title">
            <label for="koinp">KO</label>
          </div>
          <input
            class="field"
            type="text"
            id="koinp"
            v-model="formFields.ko"
            :class="{ error: !validateFields.ko }"
            @input="formValidate"
            keyTitle="ko"
          />
        </li>
        <li>
          <div class="addplayer__title">
            <label for="looseinp">Looses</label>
          </div>
          <input
            class="field"
            type="text"
            id="looseinp"
            v-model="formFields.loose"
            :class="{ error: !validateFields.loose }"
            @input="formValidate"
            keyTitle="loose"
          />
        </li>
        <li>
          <div class="addplayer__title">
            <label for="looseko">Loose KO</label>
          </div>
          <input
            class="field"
            type="text"
            id="looseko"
            v-model="formFields.lko"
            :class="{ error: !validateFields.lko }"
            @input="formValidate"
            keyTitle="lko"
          />
        </li>
      </ul>

      <button class="btn" @click="addPlayer">Добавить</button>
      <br />
      <br />
      <p>{{ sortPlayersArr }}</p>
    </div>
  </div>
</template>

<script>
import ClickOutside from 'vue-click-outside'
import { dataBase } from '@/database/database';
import penImage from "@/assets/pen.svg";
import deleteImage from "@/assets/delete.svg";

export default {
  name: 'HelloWorld',
  data() {
    return {
      penImage,
      deleteImage,
      dataPlayer: dataBase,
      sortPlayersArr: [],
      isShowFields: false,
      inpDisabledArr: [],
      isHasError: false,
      validateFields: {
        name: true,
        win: true,
        ko: true,
        loose: true,
        lko: true
      },
      formFields: {
        name: '',
        win: '',
        ko: '',
        loose: '',
        lko: ''
      }
    }
  },
  mounted() {
    this.sortPlayersArr = this.sortPlayers(this.dataPlayer);
  },
  directives: {
    ClickOutside
  },
  methods: {
    disabledTrue() {
      if (this.isShowFields) {
        for (let i = 0; i < this.inpDisabledArr.length; i++) {
          this.inpDisabledArr[i].isDisabled = true;
        }
        this.isShowFields = false;
      }
    },
    showFields(event) {
      let getIndex = event.currentTarget.getAttribute('counter');
      this.inpDisabledArr[getIndex].isDisabled = false;
      this.isShowFields = true;
    },
    spliceArr(event) {
      let index = event.currentTarget.getAttribute('counter');
      this.sortPlayersArr.splice(index, 1);
    },
    formValidate(event) {
      let valueIndex = event.currentTarget.getAttribute('keyTitle');
      let checkNumber = this.formFields[valueIndex].match(/[0-9a-zA-Z]/g);
      if (!checkNumber) {
        this.formFields[valueIndex] = '';
      } else {
        this.formFields[valueIndex] = Number(checkNumber.join(''));
      }
    },
    addPlayer() {
      for (let key in this.formFields) {
        if (this.formFields[key] == '') {
          this.isHasError = true;
          this.validateFields[key] = false;
        } else {
          this.isHasError = false;
          this.validateFields[key] = true;
        }
      }
      if (!this.isHasError) {
        let clone = {};
        for (let key in this.formFields) {
          clone[key] = this.formFields[key];
        }
        this.dataPlayer.push(clone);
        this.sortPlayersArr = this.sortPlayers(this.dataPlayer);
        for (let key in this.formFields) {
          this.formFields[key] = '';
        }
      }
    },
    resort(event) {
      let valueIndex = Number(event.currentTarget.getAttribute('index'));
      let fieldType = event.currentTarget.getAttribute('fieldType');
      let checkNumber = this.sortPlayersArr[valueIndex][fieldType].match(/[0-9a-zA-Z]/g);
      if (!checkNumber) {
        this.sortPlayersArr[valueIndex][fieldType] = 0;
      } else {
        let currentArr = Number(checkNumber.join(''));
        this.sortPlayersArr[valueIndex][fieldType] = currentArr;
      }
      this.sortPlayersArr = this.sortPlayers(this.sortPlayersArr);
    },
    sortPlayers(arr) {
      let newSortArr = [];
      for (let i = 0; i < arr.length; i++) {
        let resultPts = ((arr[i].win * 150) + (arr[i].ko * 100)) - ((arr[i].loose * 50) + (arr[i].lko * 100));
        arr[i].pts = resultPts;
        let boolean = {};
        boolean.isDisabled = true;
        this.inpDisabledArr.push(boolean);
        newSortArr.push(this.dataPlayer[i]);
        newSortArr.sort((a, b) => a.pts < b.pts ? 1 : -1);
      }
      return newSortArr;
    }
  }
}
</script>

