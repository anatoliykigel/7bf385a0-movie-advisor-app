<script setup>
import { ref, reactive, computed, onMounted } from 'vue'

import SelectControlGroup from './components/SelectControlGroup.vue'

const movies = reactive({
  list: [],
  randomOne: null,
  spinningProcess: false,
  isFirstSpin: true,
  showMotivationTitle: true
})

/*
 *  Genre Type Reactivities
 */
const genreTypesList = computed(() => [
  'all genres',
  ...new Set(movies.list.map((item) => item.genre_type))
])

const genreTypeSelected = ref(genreTypesList.value[0])

const isConcreteGenreTypeSelected = computed(
  () => genreTypeSelected.value !== genreTypesList.value[0]
)

/*
 *  Rating Score Reactivities
 */
const ratingScoresList = computed(() =>
  ['any score', ...new Set(movies.list.map((item) => Math.floor(item.rating_score)))].sort(
    (a, b) => b - a
  )
)

const ratingScoreSelected = ref(ratingScoresList.value[0])

const isConcreteRatingScoreSelected = computed(
  () => ratingScoreSelected.value !== ratingScoresList.value[0]
)

/*
 *  Year Reactivities
 */
const yearsList = computed(() =>
  ['any year', ...new Set(movies.list.map((item) => Math.floor(item.year)))].sort((a, b) => b - a)
)

const yearSelected = ref(yearsList.value[0])

const isConcreteYearSelected = computed(() => yearSelected.value !== yearsList.value[0])

/*
 *  Suitable Movies Reactivities
 */
const suitableMovies = computed(() =>
  movies.list
    .filter(
      (item) => !isConcreteGenreTypeSelected.value || item.genre_type === genreTypeSelected.value
    )
    .filter(
      (item) =>
        !isConcreteRatingScoreSelected.value || item.rating_score >= ratingScoreSelected.value
    )
    .filter((item) => !isConcreteYearSelected.value || item.year >= yearSelected.value)
)

const randomSuitableMovie = () => {
  let prevRandomOne = movies.randomOne

  movies.spinningProcess = true
  movies.randomOne = null
  movies.showMotivationTitle = false

  let moviesToSelect = [...suitableMovies.value]

  if (prevRandomOne && moviesToSelect.length >= 2) {
    /* Prevent Randomize the Same Movie Two Times */
    moviesToSelect = moviesToSelect.filter((item) => item.id != prevRandomOne.id)
  }

  let randomMovie = moviesToSelect[Math.floor(Math.random() * moviesToSelect.length)]

  if (randomMovie) {
    let __preloadImage = new Image()
    __preloadImage.src = randomMovie.image_path
  }

  setTimeout(() => {
    movies.randomOne = randomMovie
    movies.spinningProcess = false

    if (movies.isFirstSpin) setTimeout(() => (movies.isFirstSpin = false), 500)
  }, 1000)
}

onMounted(async () => {
  let data = await fetch('/api/movies_list.json')

  setTimeout(async () => {
    /* Long Data Loading Imitation */
    movies.list = await data.json()
  }, 1000)
})
</script>

<template>
  <Transition>
    <main class="main-form" v-if="movies.list.length">
      <section class="main-form__control-section">
        <SelectControlGroup
          groupTitle="Genre"
          :itemsList="genreTypesList"
          :itemModel="genreTypeSelected"
          @update:itemModel="(newValue) => (genreTypeSelected = newValue)"
        />

        <SelectControlGroup
          groupTitle="IMDB"
          endItemsWith=" and more"
          :itemsList="ratingScoresList"
          :itemModel="ratingScoreSelected"
          @update:itemModel="(newValue) => (ratingScoreSelected = newValue)"
        />

        <SelectControlGroup
          groupTitle="YEAR"
          endItemsWith=" and up"
          :itemsList="yearsList"
          :itemModel="yearSelected"
          @update:itemModel="(newValue) => (yearSelected = newValue)"
        />

        <div class="space"></div>

        <button class="btn-active" :disabled="movies.spinningProcess" @click="randomSuitableMovie">
          {{ movies.isFirstSpin ? 'SPIN' : 'SPIN AGAIN' }}
          <i
            class="fa-solid fa-spin fa-circle-notch transition"
            :style="{
              opacity: movies.spinningProcess ? 1 : 0
            }"
          ></i>
        </button>
      </section>

      <section class="main-form__info-section">
        <Transition>
          <h3 class="main-form__info-section__motivation-title" v-if="movies.showMotivationTitle">
            Let fate take the wheel... <br />Have a spin.
          </h3>
        </Transition>
        <Transition>
          <h3
            class="main-form__info-section__motivation-title"
            v-if="movies.randomOne === undefined"
          >
            Nothing found... <br />Please try another criterias
          </h3>
        </Transition>
        <Transition name="slide-fade">
          <div v-if="movies.randomOne" class="main-form__info-section__movie-wrapper">
            <img :src="movies.randomOne.image_path" alt="" />
            <div class="main-form__info-section__movie-wrapper__details">
              <h2>{{ movies.randomOne.title }}</h2>

              <p class="summary">
                {{ movies.randomOne.year }}
                &nbsp; IMDB: {{ movies.randomOne.rating_score }}/10 &nbsp;
                {{ movies.randomOne.genre_type.toUpperCase() }}
              </p>

              <p class="description">
                {{ movies.randomOne.description }}
              </p>

              <div class="space"></div>

              <div class="see-btn-group">
                <button class="btn-secondary">
                  <i class="fa-solid fa-plus"></i>
                  &nbsp;Want To See
                </button>

                <button class="btn-secondary">
                  <i class="fa-solid fa-check"></i>
                  &nbsp;Seen It
                </button>
              </div>

              <button class="btn-active">Watch Now</button>
            </div>
          </div>
        </Transition>
      </section>
    </main>
  </Transition>

  <div v-if="!movies.list.length" class="data-load-spinner">
    <i class="fa-solid fa-3x fa-spin fa-circle-notch"></i>
  </div>
</template>

<style lang="scss"></style>
