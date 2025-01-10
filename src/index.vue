<script setup> 
import { ref, onMounted, isProxy, toRaw} from "vue";

const selectedFilm = ref("")
const selectedFilmRaw = ref("")
const selectedFilmTitle = ref("")
const selectedSeason = ref("")
const selectedVoice = ref("")
const selectedEpisode = ref("")
const debouncingSearchInputTimer = ref(null)
const searchReqestResponce = ref([])
const cdn = import.meta.env.VITE_API_CDN

const getFilmData = async url => {
  const fetchRaw = await fetch(cdn + '/get', {
    method: 'POST',
    body: url
  });

  const res = await fetchRaw.json();
   //https://uakino.me/filmy/genre_adventure/209-garr-potter-kelih-vognyu.html
  selectedFilmRaw.value = res

  if (res.video) {
    selectedEpisode.value = res.video    
  } else {
    const key = Object.keys(res.seasons)[0]
    const key2 = Object.keys(res.seasons[key].list)[0]

    selectedEpisode.value = res.seasons[key]?.list[key2][0].src;
  }
}

const selectFilm = (url, title) => {
  selectedFilm.value = url
  selectedFilmTitle.value = title
  
  getFilmData(url)
}

const debouncingSearchInput = e => {
  clearInterval(debouncingSearchInputTimer.value)
  debouncingSearchInputTimer.value = setTimeout(() => handleSearchRequest(e.target.value), 500)
}

const handleSearchRequest = async text => {
  const fetchRaw = await fetch(cdn + '/search', {
    method: 'POST',
    body: text
  });
  const res = await fetchRaw.json();
  searchReqestResponce.value = res
}


const clearSelectedFilm = () => selectedFilm.value = '';
const selectSeason = (key) => {
  selectedSeason.value = key
  selectedVoice.value = ''
};
const selectVoice = (key) => selectedVoice.value = key;
const selectEpisode = (key) => selectedEpisode.value = key;
const getDescription = () => selectedFilmRaw.value?.description?.slice(0, 255) || "";

const copyToClipboard = async (text) => {
  try {
    // Використовуємо сучасний API, якщо доступний
    if (navigator.clipboard && typeof navigator.clipboard.writeText === 'function') {
      await navigator.clipboard.writeText(text);
    } else {
      // Резервний метод для старіших браузерів
      const textArea = document.createElement('textarea');
      textArea.value = text;
      textArea.style.position = 'fixed'; // уникнення прокручування
      textArea.style.opacity = '0';
      document.body.appendChild(textArea);
      textArea.focus();
      textArea.select();

      const success = document.execCommand('copy');
      if (!success) {
        throw new Error('Не вдалося виконати команду копіювання.');
      }

      document.body.removeChild(textArea);
    }
    isCopied.value = true;
    setTimeout(() => (isCopied.value = false), 2000); // Скидаємо стан через 2 секунди
  } catch (err) {
    // console.error('Clipboard error:', err);
  }
}

// defineExpose({ inputText, fileInput })
</script>

<template>
  <div class="w-screen min-h-screen h-full bg-zinc-800 text-zinc-200 font-semibold flex flex-col items-center justify-end special-p">
    <div :class="`${selectedFilm !== '' ? 'hide-screen':''} w-full w48rem flex flex-col h-full justify-end bg-zinc-900 rounded-md special-pb`">
      <div class="flex gap-2 flex-col-reverse w-full overflow-y-auto">
        <div @click="selectFilm(res.url, res.title)" class="flex flex-row flex-nowrap transition-all hover:bg-emerald-900 hover:text-emerald-200 h-14 min-h-14 p-4 duration-700 ease-in-out cursor-pointer bg-cover bg-center text-emerald-500" v-for="(res, i) in searchReqestResponce" :key="res.id" >
          <div class="text-zinc-300">[</div>
          <div class="text-yellow-500">{{ i }}</div>
          <div class="text-zinc-300 text-nowrap">] ></div>
          <div class="pl-2 truncate">{{res.title}}</div>          
        </div>
      </div>
      
      <div class="p-4 mt-4 w-full">
        <div class="flex flex-row flex-nowrap">
          <div class="pr-2 text-zinc-300">></div>
          <div class="text-yellow-500">findfilm</div>
          <div class="text-zinc-300">(</div>
        </div>
        <div class="flex flex-row flex-nowrap">
          <div class="pl-8 text-zinc-300">"</div>
          <input @input="debouncingSearchInput" placeholder="Search film here..." class="bg-transparent w-full px-1 text-amber-600" type="text">
          <div class="pr-8 text-zinc-300">",</div>
        </div>
        <div class="text-zinc-300">)</div>
      </div>
    </div>

    <div :class="`${selectedFilm == '' ? 'opacity-0 collapse':'hide-screen-r'} w-full w48rem flex flex-col h-full justify-start bg-zinc-900 rounded-md mb-4 gap-2 p-4 special-pb`">
      <div class="flex flex-row flex-nowrap">
        <div class="pr-2 text-zinc-300 select-none">></div>
        <div class="text-zinc-500 select-none pr-2">video.title:</div>
        <div class="text-amber-600 select-none">"</div>
        <div class="text-amber-600 cursor-copy" @click="copyToClipboard(selectedFilmTitle)">{{ selectedFilmTitle }}</div>
        <div class="text-amber-600 select-none">"</div>
      </div>
      <div class="flex flex-row flex-nowrap">
        <div class="text-zinc-300 select-none pr-2">></div>
        <div class="text-zinc-500 select-none pr-2">video.desc:</div>
      </div>
      <div class="text-zinc-600 pl-4">{{ getDescription() }}...</div>
      <div class="flex flex-row flex-nowrap">
        <div class="text-zinc-300 select-none pr-2">></div>
        <div class="text-zinc-500 select-none pr-2">video.src:</div>
      </div>
      <div class="pl-4">
        <iframe style="height: calc(100vw * 9 / 16 - 1rem);max-height: 482px;" class="bg-red-200 w-full rounded-md" :src="selectedEpisode" frameborder="0" allowfullscreen></iframe>
      </div>

      <div v-if="!selectedFilmRaw?.video" class="flex flex-row flex-wrap items-center">
        <div class="flex flex-row flex-nowrap">
          <div class="pr-2 text-zinc-300 select-none">></div>
          <div class="text-zinc-500 select-none">seasons:</div>
        </div>
        <div v-for="(key, index) in selectedFilmRaw.seasons" :key="key.id" @click="selectSeason(key)" class="select-none cursor-pointer text-zinc-400 hover:underline underline-offset-4 hover:text-emerald-500 p-1 transition-all duration-700">
          {{ index }}
        </div>
      </div>
      
      <div v-if="selectedSeason" class="flex flex-row flex-wrap items-center">
        <div class="flex flex-row flex-nowrap">
          <div class="pr-2 text-zinc-300 select-none">></div>
          <div class="text-zinc-500 select-none">voice:</div>
        </div>
        <div v-for="(key, index) in selectedSeason.list" :key="key.id" @click="selectVoice(key)" class="select-none cursor-pointer text-zinc-400 hover:underline underline-offset-4 hover:text-emerald-500 p-1 transition-all duration-700">
          {{ index }}
        </div>
      </div>
      
      <div v-if="selectedVoice" class="flex flex-row flex-wrap items-center">
        <div class="flex flex-row flex-nowrap">
          <div class="pr-2 text-zinc-300 select-none">></div>
          <div class="text-zinc-500 select-none">episodes:</div>
        </div>
        <div v-for="key of selectedVoice" :key="key.id" @click="selectEpisode(key.src)" class="select-none cursor-pointer text-zinc-400 hover:underline underline-offset-4 hover:text-emerald-500 p-1 transition-all duration-700">
          {{ key.name }}
        </div>
      </div>
      
      <div @click="clearSelectedFilm" class="flex flex-row flex-nowrap cursor-pointer mt-auto">
        <div class="pr-2 text-zinc-300 select-none">></div>
        <div class="select-none text-emerald-400">search.reload</div>
        <div class="text-zinc-500 select-none">()</div>
      </div>
    </div>
  </div>
</template>

<style>
  .w48rem {
    max-width: 62rem;
  }
  .hide-screen {
    animation: sweep 2s both;
  }
  .hide-screen-r {
    animation: sweep .5s both reverse;
  }

  .special-p {
    padding-left: 1rem;
    padding-right: 1rem;
    padding-top: 1rem;
  }

  .special-pb {
    margin-bottom: 1rem;
  }

  div.poster_grayscale  {
    filter: saturate(0%);
    animation: sweep-poster .2s steps(25) both reverse;
  }

  div.poster_grayscale:hover {
    animation: sweep-poster .2s steps(25) both;
  }

  @media (max-width: 1024px) {
    .special-p {
      padding: 0px;
    }
    .special-pb {
      margin-bottom: 0px;
    }
  }

  @keyframes sweep {
    0%   {height: 100%; overflow: auto; clip: rect(10px, 20px, 30px, 40px);}
    100% {height: 0px;margin-top: -1rem; overflow: auto; clip: rect(10px, 20px, 30px, 40px);}
  }

  @keyframes sweep2 {
    0%   {height: 100%; opacity: 1;}
    99%  {height: 0px;}
    100% {visibility: collapse; opacity: 0;}
  }

  @keyframes sweep-poster {
    0%  {filter: saturate(0%);}
    100%    {filter: saturate(100%);}
  }
</style>