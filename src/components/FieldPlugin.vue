<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useFieldPlugin } from '@storyblok/field-plugin/vue3'

const plugin = useFieldPlugin()
const pluginName = 'sb-notion-pages'

const notionEpisodes = ref()
const searchInput = ref('')
const isOpen = ref(false)
const selectedEpisode: any = ref(plugin.data?.content)

const notionEpisodesFiltered = computed(() => {
  return (
    notionEpisodes.value?.filter((episode: any) =>
      episode.title.toLowerCase().includes(searchInput.value.toLowerCase()),
    ) ?? []
  )
})

const getEpisodes = async () => {
  const episodesServerLess = await fetch(
    `https://sb-notion-serverless-functions.netlify.app/.netlify/functions/nomedalavida-notes`,
  ).then((res) => res.json())
  notionEpisodes.value = Object.values(episodesServerLess).reduce(
    (acc: any, episodes: any) => {
      return [...acc, ...episodes]
    },
    [],
  )
}

const searchEpisode = () => {
  isOpen.value = true
}

const selectEpisode = (episode: any) => {
  plugin.actions?.setContent({
    plugin: pluginName,
    episode,
  })
  selectedEpisode.value = { episode }
  isOpen.value = false
}

onMounted(() => {
  getEpisodes()
})
</script>

<template>
  <div class="container">
    <form
      @submit.prevent="searchEpisode"
      class="form-field"
    >
      <input
        class="input"
        v-model="searchInput"
        placeholder="Type episode name"
      />
      <button
        class="btn"
        type="submit"
      >
        <svg
          width="20"
          height="20"
          aria-labelledby="title desc"
          role="img"
          xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 19.9 19.7"
        >
          <title id="title">Search Icon</title>
          <desc id="desc">A magnifying glass icon.</desc>
          <g
            class="search-path"
            fill="none"
            stroke="currentColor"
            stroke-width="1.5"
          >
            <path
              stroke-linecap="square"
              d="M18.5 18.3l-5.4-5.4"
            />
            <circle
              cx="8"
              cy="8"
              r="7"
            />
          </g>
        </svg>
      </button>
    </form>
    <p
      v-if="selectedEpisode && selectedEpisode.episode"
      class="episode-selected"
    >
      Selected episode
      <span>{{ selectedEpisode.episode.title }}</span>
    </p>
    <ul
      v-if="isOpen"
      class="episodes-list"
    >
      <li
        v-for="episode in notionEpisodesFiltered"
        class="episode-item"
        @click="selectEpisode(episode)"
      >
        {{ episode.title }}
      </li>
    </ul>
  </div>
</template>

<style scoped>
.form-field {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.episode-selected {
  display: flex;
  flex-direction: column;
  padding: 1rem;
  background: var(--sb_green);
  color: var(--white);
  border-radius: 4px;
}
.episode-selected span {
  font-weight: bold;
  font-size: 1.15rem;
}

.episodes-list {
  position: absolute;
  top: 50px;
  left: 0;
  right: 0;
  max-height: 200px;
  overflow-y: auto;
  padding: 0;
  margin: 0;
  list-style: none;
  background-color: var(--white);
  border: 1px solid var(--sb_green);
  border-top: 0;
  border-radius: 0 0 4px 4px;
}

.episode-item {
  padding: 1rem;
}
.episode-item:hover {
  background-color: var(--light_25);
}
</style>
