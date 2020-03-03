<template lang="pug">
  #app.my-5
    .content.py-3(v-if="current")
      img(:src="current.img_src ? current.img_src : require('./assets/placeholder.png')",)
      h1.my-5 {{current.name.toUpperCase()}}
      draggable(:list="anagram", @end="checkWord")
        span(v-for="letter in anagram").p-2.mx-2.text-white.bg-secondary.h4 {{letter}}

    table.table(v-if="!current")
      thead
        tr
          th(@click="sortTable()", scope="col") #
          th(@click="sortTable()", scope="col") Слово
          th(@click="sortTable()", scope="col") Время
      tbody
        tr(v-for="res in results")
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
      current: null,
      startTime: new Date(),
      anagram: [],
      success: false,
    }
  },
  mounted () {
    this.getWord()
  },
  methods: {
    getAnagram (word) {
      this.anagram = []

      let splitWord = word.split('')
      word.split('').forEach((letter, index) => {
        let brr = this.getRandom(0, splitWord.length - index - 1)
        this.anagram.push(splitWord.splice(brr, 1)[0])

      })
    },
    getRandom (min, max) {
      max++
      return Math.floor(Math.random() * (max - min)) + min
    },
    getWord () {
      let random = this.getRandom(2, 1368)
      axios.get(`https://apidir.pfdo.ru/v1/directory-program-activities/${random}`).
        then(({ data }) => {
          if (data.data) {
            // нужна проверка если одинаковое слово придет
            this.current = data.data

            console.log(this.current)

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
      if (this.anagram.join('') === this.current.name) {

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
    gameNext() {
      this.getWord();
      this.success = false
    },
    gameEnd () {
      this.current = null;
      this.success = false
    },
    sortTable () {

    }
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
}

.modal {
  display: block;
  background-color: #00000077;
}

</style>
