<script setup>
import { computed, ref, onMounted, onBeforeUnmount } from 'vue';
import { getRecentTracks } from '@/services/lastfmService';


const discordStatusColor = computed(() => "#5865F2"); // Default to Discord purple
const spotify = ref(null);
const discordStatus = ref("Offline");
const discordUser = ref(null);
const vscodeActivity = ref(null);
const isLoading = ref(false);

const vscodeStatus = computed(() => {
  if (!vscodeActivity.value) return null;

  if (vscodeActivity.value.details && vscodeActivity.value.details.toLowerCase().includes('idling')) {
    return 'idling';
  }
  return {
    name: vscodeActivity.value.name,
    details: vscodeActivity.value.details,
    state: vscodeActivity.value.state
  };
});

const profilePhotoUrl = ref("https://media.discordapp.net/attachments/1056288920184307812/1411440208368504994/tot.jpg?ex=68b4a9a0&is=68b35820&hm=b4fd97d99f00138caba4f682e4e62cc5000dce96fbedbedf4bcadcde7434137e");


const repos = ref([]);

const allTracks = ref([]);
const songsLoading = ref(true);
const songsError = ref(null);
const imageErrors = ref({});
let updateInterval = null;



const handleImageError = (trackName) => {
  imageErrors.value[trackName] = true;
};

const currentTrack = computed(() => {
  return allTracks.value.find(track => track['@attr']?.nowplaying);
});

const consolidatedTracks = computed(() => {
  const tracks = allTracks.value.filter(track => !track['@attr']?.nowplaying);
  const consolidated = [];
  let currentTrack = null;
  let count = 1;

  tracks.forEach((track, index) => {
    const key = `${track.name}-${track.artist['#text']}`;

    if (currentTrack === key) {
      count++;
    } else {
      if (currentTrack) {
        const prevTrack = tracks[index - 1];
        consolidated.push({
          ...prevTrack,
          playcount: count,
          date: prevTrack.date?.['#text']
        });
      }
      currentTrack = key;
      count = 1;
    }

    if (index === tracks.length - 1) {
      consolidated.push({
        ...track,
        playcount: count,
        date: track.date?.['#text']
      });
    }
  });

  return consolidated.slice(0, 10);
});

const formatDate = (dateString) => {
  if (!dateString) return '';
  const date = new Date(dateString);
  return date.toLocaleDateString('en-US', {
    month: 'short',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  });
};



const getLanguageIcon = (language) => {
  const devIcons = {
    'QML': 'https://cdn.jsdelivr.net/gh/devicons/devicon/icons/qt/qt-original.svg',
    'CMake': 'https://cdn.jsdelivr.net/gh/devicons/devicon/icons/cmake/cmake-original.svg',
    'Makefile': 'https://cdn.jsdelivr.net/gh/devicons/devicon/icons/cmake/cmake-original.svg',
    'Objective-C': 'https://cdn.jsdelivr.net/gh/devicons/devicon/icons/objectivec/objectivec-plain.svg',
    'Vim script': 'https://cdn.jsdelivr.net/gh/devicons/devicon/icons/vim/vim-original.svg'
  };

  // Check if we have a devicon first
  if (devIcons[language]) {
    return devIcons[language];
  }
  const languageMap = {
    'JavaScript': 'js',
    'TypeScript': 'ts',
    'Python': 'py',
    'Java': 'java',
    'C++': 'cpp',
    'C#': 'cs',
    'C': 'c',
    'Go': 'go',
    'Rust': 'rust',
    'PHP': 'php',
    'Ruby': 'ruby',
    'Swift': 'swift',
    'Kotlin': 'kotlin',
    'Dart': 'dart',
    'HTML': 'html',
    'CSS': 'css',
    'Vue': 'vue',
    'React': 'react',
    'Svelte': 'svelte',
    'Angular': 'angular',
    'Shell': 'bash',
    'PowerShell': 'powershell',
    'Dockerfile': 'docker'
  };

  const mappedLanguage = languageMap[language] || language.toLowerCase();
  return `https://skillicons.dev/icons?i=${mappedLanguage}`;
};

const fetchSongs = async () => {
  try {
    songsLoading.value = true;
    const newTracks = await getRecentTracks();
    allTracks.value = newTracks;
    imageErrors.value = {};
    songsError.value = null;
  } catch (err) {
    songsError.value = "Failed to load tracks. Please try again later.";
    console.error('Error fetching tracks:', err);
  } finally {
    songsLoading.value = false;
  }
};

const fetchProjects = async () => {
  try {
    const reposResponse = await fetch('https://api.github.com/users/tot1337/repos');
    const reposData = await reposResponse.json();
    repos.value = reposData.sort((a, b) => b.stargazers_count - a.stargazers_count);
  } catch (error) {
    console.error('Error fetching projects:', error);
  }
};

onMounted(() => {
  fetchProjects();
  fetchSongs();
  updateInterval = setInterval(fetchSongs, 30000);
});

onBeforeUnmount(() => {
  if (updateInterval) {
    clearInterval(updateInterval);
  }
});
</script>

<template>
  <div class="w-full max-w-screen-lg mx-auto px-3 sm:px-4 py-4 md:p-6 relative min-h-screen overflow-x-hidden">
    
    <!-- Header Section -->
    <div class="relative mb-6 md:mb-8 mt-2 md:mt-4">
      <!-- Hero Section -->
        <div class="mb-10">
          <!-- Title with Profile -->
          <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-6 mb-8">
            <div class="grid lg:grid-cols-2 gap-4 mb-8">
              <div class="flex flex-wrap items-center gap-4 sm:flex-nowrap sm:justify-start lg:col-span-1">
                <div
                  class="w-16 h-16 md:w-20 md:h-20 rounded-2xl bg-red-dark flex items-center justify-center overflow-hidden flex-shrink-0 ring-2 ring-red-accent/30 shadow-lg">
                  <img v-if="profilePhotoUrl"
                    :src="profilePhotoUrl"
                    alt="Profile Avatar" class="w-full h-full object-cover" />
                  <font-awesome-icon v-else :icon="['fas', 'user']" class="text-2xl md:text-3xl text-red-light" />
                </div>
                <div class="flex-1">
                  <p class="text-base sm:text-lg md:text-xl text-red-text font-medium mb-1">
                    <span class="text-red-light">zeytin</span> <span class="text-red-light">(<span
                        class="text-red-accent">aka tot</span>)</span>
                  </p>
                  <h1 class="font-sans text-2xl sm:text-3xl md:text-4xl font-black text-red-text">
                    <span class="text-red-medium">zeyt</span><span class="text-red-light">.in</span>
                  </h1>
                </div>
              </div>
            </div>
            <!-- Social Links -->
            <div class="flex items-center justify-center sm:justify-end gap-3 sm:gap-4 lg:col-span-1 lg:justify-end">
              <a href="https://github.com/tot1337" target="_blank"
                class="w-12 h-12 glass rounded-xl text-red-light hover:text-red-medium transition-colors flex items-center justify-center">
                <font-awesome-icon :icon="['fab', 'github']" class="text-xl" />
              </a>
              <a href="https://www.instagram.com/192.168.3.6" target="_blank"
                class="w-12 h-12 glass rounded-xl text-red-light hover:text-red-accent transition-colors flex items-center justify-center">
                <font-awesome-icon :icon="['fab', 'instagram']" class="text-xl" />
              </a>
              <a href="https://discord.com/user/470904884946796544" target="_blank"
                class="w-12 h-12 glass rounded-xl text-red-light hover:text-red-light transition-colors flex items-center justify-center">
                <font-awesome-icon :icon="['fab', 'discord']" class="text-xl" />
              </a>
              <a href="https://open.spotify.com/user/31h5p2gqbcol3zt3xvorjnpjtzxm?si=3a2712faffea4cc7" target="_blank"
                class="w-12 h-12 glass rounded-xl text-red-light hover:text-red-accent transition-colors flex items-center justify-center">
                <font-awesome-icon :icon="['fab', 'spotify']" class="text-xl" />
              </a>
            </div>
          </div>

          <!-- Bio & Status Grid -->
          <div class="grid lg:grid-cols-2 gap-4 sm:gap-6">
            <!-- Bio Card -->
            <div class="lg:col-span-1 glass rounded-2xl p-4 hover-glow transition-shadow">
              <p class="text-base md:text-lg text-red-light leading-relaxed">
                <span class="text-red-accent font-semibold">student</span> with a passion for <span
                  class="text-red-accent font-semibold">copying code</span>. when i'm not coding, i enjoy ddnet and music.
              </p>

              
            </div>

            <!-- Tech Stack Card -->
            <div class="lg:col-span-1 glass rounded-2xl p-4 hover-glow transition-shadow">
              <h3 class="text-lg font-bold text-red-medium mb-4">uses/</h3>
              <div class="grid grid-cols-2 gap-2">
                <img src="https://skillicons.dev/icons?i=linux"
                  class="w-8 h-8 hover:scale-110 transition-transform" />
                <img src="https://skillicons.dev/icons?i=git"
                  class="w-8 h-8 hover:scale-110 transition-transform" />
                <img src="https://skillicons.dev/icons?i=vscode"
                  class="w-8 h-8 hover:scale-110 transition-transform" />
                <img src="https://skillicons.dev/icons?i=github"
                  class="w-8 h-8 hover:scale-110 transition-transform" />
              </div>
          </div>
          </div>
        </div>
      </div>
      <!-- Projects & Songs Side by Side -->
      <div class="flex flex-col lg:grid lg:grid-cols-2 gap-4 sm:gap-6 md:gap-8 mb-8 md:mb-12">
        <!-- Projects Section -->
        <div class="relative">
          <h2 class="text-lg md:text-xl font-black text-red-medium mb-3 animate-fade-in">projects/</h2>

          <div class="space-y-2">
            <div v-if="!repos.length" class="text-center text-red-light py-8">
              <font-awesome-icon :icon="['fas', 'code']" class="text-2xl mb-2 opacity-50" />
              <p class="text-xs">Projects could not be retrieved.</p>
            </div>
            <a v-for="repo in repos.slice(0, 6)" :key="repo.id" :href="repo.html_url" target="_blank"
              class="flex items-center gap-3 p-3 glass rounded-lg hover-glow transition-shadow cursor-pointer group">
              <div
                class="w-8 h-8 rounded-md bg-red-dark flex items-center justify-center flex-shrink-0 overflow-hidden">
                <img v-if="repo.language" 
                  :src="getLanguageIcon(repo.language)"
                  :alt="repo.language"
                  class="w-full h-full object-cover transition-transform group-hover:scale-110"
                  @error="$event.target.src = repo.owner.avatar_url">
                <img v-else :src="repo.owner.avatar_url"
                  class="w-full h-full object-cover transition-transform group-hover:scale-110">
              </div>
              <div class="flex-1 min-w-0">
                <p class="font-semibold text-red-text truncate text-sm group-hover:text-red-medium transition-colors"
                  :title="repo.name">
                  {{ repo.name }}
                </p>
                <p class="text-red-light truncate text-xs" :title="repo.description">
                  {{ repo.description || 'No description' }}
                </p>
              </div>
              <div class="flex items-center gap-2 text-xs flex-shrink-0">
                <div class="flex items-center gap-1 text-red-accent">
                  <font-awesome-icon :icon="['fas', 'star']" />
                  {{ repo.stargazers_count }}
                </div>
              </div>
            </a>
          </div>
        </div>

        <!-- Songs Section -->
        <div class="relative">
          <h2 class="text-lg md:text-xl font-black text-red-medium mb-3 animate-fade-in">songs/</h2>

          <div class="space-y-2">
            <!-- Loading State - Show skeleton cards -->
            <template v-if="songsLoading">
              <div v-for="i in 6" :key="`loading-${i}`" class="flex items-center gap-3 p-3 glass rounded-lg animate-pulse">
                <div class="w-8 h-8 rounded-md bg-red-dark flex items-center justify-center flex-shrink-0">
                  <font-awesome-icon :icon="['fas', 'music']" class="text-sm text-red-light opacity-50" />
                </div>
                <div class="flex-1 min-w-0">
                  <div class="h-4 bg-red-dark rounded mb-1 w-3/4"></div>
                  <div class="h-3 bg-red-dark rounded w-1/2"></div>
                </div>
                <div class="w-12 h-3 bg-red-dark rounded"></div>
              </div>
            </template>
            
            <!-- Error State -->
            <div v-else-if="songsError" class="text-center text-red-medium py-8">
              <font-awesome-icon :icon="['fas', 'music']" class="text-2xl mb-2 opacity-50" />
              <p class="text-xs">{{ songsError }}</p>
            </div>
            
            <!-- Empty State -->
            <div v-else-if="!consolidatedTracks.length && !currentTrack" class="text-center text-red-light py-8">
              <font-awesome-icon :icon="['fas', 'music']" class="text-2xl mb-2 opacity-50" />
              <p class="text-xs">No recent tracks found.</p>
            </div>

            <!-- Loaded Content -->
            <template v-else>
              <!-- Now Playing Track (if exists) -->
              <a v-if="currentTrack" :href="currentTrack.url" target="_blank"
                class="flex items-center gap-3 p-3 glass rounded-lg hover-glow transition-all duration-200 cursor-pointer transform hover:scale-105 group border border-red-accent/30">
                <div
                  class="w-8 h-8 rounded-md overflow-hidden bg-red-dark flex items-center justify-center flex-shrink-0">
                  <img :src="currentTrack.image[1]['#text']" alt="track image"
                    class="w-full h-full object-cover transition-transform duration-300 group-hover:scale-110"
                    @error="handleImageError(currentTrack.name)" :class="{ 'hidden': imageErrors[currentTrack.name] }" />
                  <font-awesome-icon v-if="imageErrors[currentTrack.name]" :icon="['fas', 'compact-disc']"
                    class="text-sm text-red-light animate-spin-slow" />
                </div>
                <div class="flex-1 min-w-0">
                  <p class="font-semibold text-red-text truncate text-sm group-hover:text-red-accent transition-colors duration-200"
                    :title="currentTrack.name">
                    {{ currentTrack.name }}
                  </p>
                  <p class="text-red-light truncate text-xs" :title="currentTrack.artist['#text']">{{
                    currentTrack.artist['#text'] }}</p>
                </div>
                <div class="flex items-center gap-1 text-red-accent text-xs flex-shrink-0">
                  <div class="w-1.5 h-1.5 rounded-full bg-red-accent animate-pulse"></div>
                  Now
                </div>
              </a>

              <!-- Recent Tracks -->
              <a v-for="track in consolidatedTracks.slice(0, currentTrack ? 1 : 2)"
                :key="`${track.name}-${track.artist['#text']}-${track.date}`" :href="track.url" target="_blank"
                class="flex items-center gap-3 p-3 glass rounded-lg hover-glow transition-shadow cursor-pointer group">
                <div
                  class="w-8 h-8 rounded-md overflow-hidden bg-red-dark flex items-center justify-center flex-shrink-0">
                  <img :src="track.image[1]['#text']" alt="track image"
                    class="w-full h-full object-cover transition-transform duration-300 group-hover:scale-110"
                    @error="handleImageError(track.name)" :class="{ 'hidden': imageErrors[track.name] }" />
                  <font-awesome-icon v-if="imageErrors[track.name]" :icon="['fas', 'compact-disc']"
                    class="text-sm text-red-light animate-spin-slow" />
                </div>
                <div class="flex-1 min-w-0">
                  <p class="font-semibold text-red-text truncate text-sm group-hover:text-red-medium transition-colors" :title="track.name">
                    {{ track.name }}
                    <span v-if="track.playcount > 1" class="text-red-accent text-xs ml-1">{{ track.playcount
                    }}x</span>
                  </p>
                  <p class="text-red-light truncate text-xs" :title="track.artist['#text']">{{ track.artist['#text']
                  }}</p>
                </div>
                <div class="text-xs text-red-light flex-shrink-0">
                  {{ formatDate(track.date) }}
                </div>
              </a>
            </template>
          </div>
        </div>
      </div>
    </div>
</template>
