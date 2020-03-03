<template lang="pug">
  #app.my-5
    h1.h4 Игра, в которой из анаграмы перетаскиванием ячеек нужно собрать целое слово
    .content.py-3(v-if="current")
      img(:src="current.img_src ? current.img_src : require('./assets/placeholder.png')",)
      // h1.my-5 {{current.name}}
      draggable(:list="anagram", @end="checkWord").d-flex.flex-wrap.my-5
        div(v-for="letter in anagram").draggable__item {{letter}}

    table.table.table-hover(v-if="showResult")
      thead
        tr
          th(@click="sortFunc('num')", scope="col") #
            img(src="./assets/sort.svg")
          th(@click="sortFunc('word')", scope="col") Слово
            img(src="./assets/sort.svg")
          th(@click="sortFunc('time')", scope="col") Время
            img(src="./assets/sort.svg")
      tbody
        tr(v-for="res in sortTable")
          th {{res.num}}
          td {{res.word}}
          td {{res.time}}

    .modal(v-if="success")
      .modal-dialog
        .modal-content
          .modal-header
            h5.modal-title Правильно
            button.close
          .modal-body
            p Вы выйграли! Повторить?
          .modal-footer
            button(@click="gameNext()").btn.btn-primary Конечно!
            button(@click="gameEnd()").btn.btn-secondary Я уже устал.

</template>

<script>
import axios from 'axios'
import draggable from 'vuedraggable'
import 'bootstrap/dist/css/bootstrap.min.css'

export default {
  name: 'App',
  components: {
    draggable,
  },
  data () {
    return {
      results: [],
      anagram: [],
      current: null,
      startTime: null,
      success: false,
      showResult: false,
      sort: {
        field: null,
        direction: true
      }
    }
  },
  mounted () {
    // инициализация игры при открытии сайта
    this.getWord()
  },
  methods: {
    getAnagram (word) {
      // получение анаграму из слова
      this.anagram = []
      let splitWord = word.split('')
      word.split('').forEach((letter, index) => {
        let brr = this.getRandom(0, splitWord.length - index - 1)
        this.anagram.push(splitWord.splice(brr, 1)[0])
      })
    },
    getRandom (min, max) {
      // получение рандомного числа
      max++
      return Math.floor(Math.random() * (max - min)) + min
    },
    getWord () {
      // запрос на получение слова
      let random = this.getRandom(2, 1368)
      axios.get(`https://apidir.pfdo.ru/v1/directory-program-activities/${random}`).
        then(({ data }) => {
          if (data.data) {
            // если данное слово раньше попадалась
            this.results.forEach(i => {
              if (i.id === data.data.id) {
                this.getWord()
              }
            })
            this.current = data.data
            this.current.name = this.current.name.toUpperCase()
            // определяем время начала
            this.startTime = new Date()
            // получает анаграму для вывода
            this.getAnagram(this.current.name)
          } else {
            this.getWord()
          }

        }).
        catch(err => {
          console.log('error', err)
        })
    },
    checkWord () {
      // проверка слова при изменение позиции
      if (this.anagram.join('') === this.current.name) {

        // вычисление минут и секунд
        let t = (+new Date - this.startTime) / 1000
        let m = t / 60 < 10 ? '0' + Math.floor(t / 60) : Math.floor(t / 60)
        let s = t % 60 < 10 ? '0' + Math.floor(t % 60) : Math.floor(t % 60)

        this.success = true
        this.results.push({
          id: this.current.id,
          num: this.results.length + 1,
          word: this.current.name,
          time: `${m} : ${s}`,
        })
      }
    },
    gameNext () {
      // продолжение игры
      this.getWord()
      this.success = false
    },
    gameEnd () {
      // конец игры и вывод результата
      this.current = null
      this.showResult = true
      this.success = false
    },
    sortFunc (dir) {
      // меня направление сортировки
      if(this.sort.field === dir) {
        this.sort.direction = !this.sort.direction
      } else {
        this.sort.direction = true
      }
      this.sort.field = dir
    },
  },
  computed: {
    sortTable () {
      // сортировка результата

      let results = this.results
      if(this.sort.field) {
        return results.sort((a, b) => {
          if (this.sort.direction) {
            return (a[this.sort.field] > b[this.sort.field]) ? 1 : -1;
          } else {
            return (a[this.sort.field] < b[this.sort.field]) ? 1 : -1;
          }
        })
      }

      return results
    },
  },
}
</script>

<style lang="scss">

#app {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.content {
  flex: 1;
  text-align: center;
  display: flex;
  flex-direction: column;
  justify-content: space-between;

  img {
    margin: 0 auto;
    width: 400px;
    height: 400px;
    object-fit: cover;
    object-position: center;
  }

  .draggable__item {
    cursor: pointer;
    padding: 5px 10px;
    font-size: 25px;
    border: 1px solid #007bff;
    margin: 5px;
    min-width: 40px;
  }
}

.table {
  max-width: 600px;

  thead tr {
    cursor: pointer;

    img {
      height: 15px;
      margin-left: 10px;
    }
  }
}

.modal {
  display: block;
  background-color: #00000077;
}

</style>
