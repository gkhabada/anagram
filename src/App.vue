<template lang="pug">
  #app.my-5
    button(@click="gameStart").btn.btn-primary Начать
    .content.py-3(v-if="current")
      img(:src="current.img_src ? current.img_src : require('./assets/placeholder.png')",)
      h1.mt-5 {{current.name.toUpperCase()}}
      p
        span(v-for="letter in anagram").m-2.text-primary {{letter}}


</template>

<script>
import 'bootstrap/dist/css/bootstrap.min.css'
import axios from 'axios'

export default {
  name: 'App',
  data () {
    return {
      items: [],
      current: null,
      startTime: new Date(),
      anagram: [],
    }
  },
  methods: {
    gameStart () {
      this.getWord()
    },
    getAnagram (word) {
      this.anagram = [];

      let splitWord = word.split('')
      word.split('').forEach((letter, index) => {
        let brr = this.getRandom(0, splitWord.length - index)
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
    checkResult () {

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
}

</style>
