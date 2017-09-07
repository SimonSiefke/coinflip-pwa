<template lang="pug">
  main
    p.error(v-if="errorMessage") {{this.errorMessage}}
    .coin(v-if="this.fetched" @click="getSide();clearErrorMessage()")
      span.side {{this.side}}

    i.fa.fa-spinner.fa-spin.side(v-if="!this.fetched")


</template>

<script>
import axios from 'axios'

// Website for random numbers
const site = 'https://www.random.org/integers/?num=1&min=0&max=1&col=1&base=10&format=plain&rnd=new'
let errorTimeout

export default {
  data() {
    return {
      fetched: false,
      side: '',
      errorMessage: ''
    }
  },
  methods: {
    localRandomNumber() {
      const result = Math.random() > .5 ? 1 : 0
      return result
    },
    async getRandomNumber() {
      let fetched = false

      return new Promise((resolve, reject) => {
        axios(site)
          .then(response => {
            if (!fetched) {
              fetched = true
              const result = response.data
              resolve(result)
            }
          }).catch(err => {
            this.displayError('request failed, using local randomness')
            fetched = true
            resolve(this.localRandomNumber())
          })

        // If axios takes too long
        setTimeout(() => {
          if (!fetched) {
            fetched = true
            this.displayError('request is taking too long, used local randomness')
            resolve(this.localRandomNumber())
          }
        }, 5500)
      })
    },
    async randomSide() {
      const result = await this.getRandomNumber() % 2 === 0 ? "100" : "$"
      return result
    },
    async getSide() {
      this.fetched = false
      const side = await this.randomSide()
      this.side = side
      this.fetched = true
      return side
    },
    clearErrorMessage() {
      this.errorMessage = ''
    },
    displayError(message) {
      this.errorMessage = message
      clearTimeout(errorTimeout)
      errorTimeout = setTimeout(() => {
        this.clearErrorMessage()
      }, 6000)
    }
  },
  mounted() {
    this.getSide()
  }
}
</script>


<style lang="stylus">

//////////////
// variables
////////////

$coin-size = 40vmin
$orange = #f7941e
$light-orange = lighten($orange, 25%)
$error-yellow = #f1bd00

//////////////
// functions
////////////

flex()
  display flex
  justify-content center
  align-items center


/////////////
// styling
///////////

body
  background linear-gradient(to right, #d31027, #ea384d)


main
  height 100vh
  flex()

.error
  position absolute
  top 0
  background $error-yellow
  color white
  padding 10px
  font-size 200%
  border-radius 5px
  width 70%
  max-width 700px
  text-align center
  margin-top 30px

  &::selection
    background lime


  // centering horizontally
  left: 50%;
  transform: translate(-50%, 0);

.coin
  width $coin-size
  height $coin-size
  background $orange
  border-radius 50%
  border 'calc(%s/7)' % $coin-size solid $light-orange
  cursor pointer
  flex()

.side
  color $light-orange
  font-size 'calc(%s/1.7)' % $coin-size
  user-select none

i
  cursor default

</style>
