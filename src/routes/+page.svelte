<script lang="ts">
  import { onMount } from "svelte";
  import { fade, fly } from "svelte/transition";
  import { quintOut } from "svelte/easing";

  let movies: any[] = [];
  let searchQuery = "";
  let selectedMovie: any = null;
  let isPlaying = false;
  let savedMovies: Set<number> = new Set();

  const API_KEY = "06d4705a4cc85a5bdc8fdf430ea9fa60";
  const BASE_URL = "https://api.themoviedb.org/3";
  const EMBED_URL = "https://www.2embed.cc/embed/";

  onMount(fetchPopularMovies);

  async function fetchPopularMovies() {
    const url = `${BASE_URL}/movie/popular?api_key=${API_KEY}`;
    movies = await fetchData(url);
  }

  async function searchMovies() {
    if (searchQuery.trim() === "") return fetchPopularMovies();
    const url = `${BASE_URL}/search/movie?api_key=${API_KEY}&query=${encodeURIComponent(searchQuery)}`;
    movies = await fetchData(url);
  }

  async function fetchData(url: string) {
    try {
      const response = await fetch(url);
      if (!response.ok) throw new Error("API request failed");
      const data = await response.json();
      return data.results;
    } catch (error) {
      console.error("Error fetching data:", error);
      return [];
    }
  }

  function getImageUrl(path: string | null) {
    return path
      ? `https://image.tmdb.org/t/p/w500${path}`
      : "https://placehold.co/400x600?text=No+Image";
  }

  function getEmbedUrl(movieId: number) {
    return `${EMBED_URL}${movieId}`;
  }

  function toggleSave(event: Event, movieId: number) {
    event.stopPropagation();
    savedMovies.has(movieId)
      ? savedMovies.delete(movieId)
      : savedMovies.add(movieId);
    savedMovies = savedMovies;
  }

  function handleKeydown(event: KeyboardEvent) {
    if (event.key === "Escape") {
      if (isPlaying) isPlaying = false;
      else if (selectedMovie) selectedMovie = null;
    }
  }
</script>

<svelte:window on:keydown={handleKeydown} />

<main class="min-h-screen bg-gray-50">
  <header class="sticky top-0 z-50 bg-white shadow-md">
    <div
      class="container mx-auto px-4 py-4 flex flex-col sm:flex-row justify-between items-center"
    >
      <h1 class="text-3xl font-bold text-gray-800">WATCHA</h1>
      <div class="flex items-center space-x-4 mt-4 sm:mt-0">
        <input
          type="text"
          bind:value={searchQuery}
          on:input={searchMovies}
          placeholder="Search movies..."
          class="w-full sm:w-64 bg-gray-100 rounded-full px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-300"
        />
        <button class="text-gray-600 hover:text-gray-800" aria-label="GitHub">
          <svg class="w-6 h-6" fill="currentColor" viewBox="0 0 24 24">
            <path
              d="M12 0c-6.626 0-12 5.373-12 12 0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23.957-.266 1.983-.399 3.003-.404 1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576 4.765-1.589 8.199-6.086 8.199-11.386 0-6.627-5.373-12-12-12z"
            />
          </svg>
        </button>
      </div>
    </div>
  </header>

  <div class="container mx-auto p-4 sm:p-8">
    <div
      class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 xl:grid-cols-6 gap-4"
    >
      {#each movies as movie (movie.id)}
        <div
          class="relative overflow-hidden rounded-lg shadow-md transition-all duration-300 transform hover:scale-105 cursor-pointer bg-white"
          on:click={() => (selectedMovie = movie)}
          on:keydown={(e) => e.key === "Enter" && (selectedMovie = movie)}
          role="button"
          tabindex="0"
          in:fly={{ y: 50, duration: 300, delay: 300, easing: quintOut }}
        >
          <img
            src={getImageUrl(movie.poster_path)}
            alt={movie.title}
            class="w-full h-full object-cover"
          />
          <div
            class="absolute inset-0 bg-gradient-to-t from-black via-black/70 to-transparent opacity-0 hover:opacity-100 transition-opacity duration-300 flex flex-col justify-end p-2 sm:p-4"
          >
            <h2 class="text-white text-sm font-semibold truncate mb-2">
              {movie.title}
            </h2>
            <div class="flex items-center justify-between text-xs text-white">
              <span class="flex items-center">
                <svg
                  class="w-3 h-3 text-yellow-400 mr-1"
                  fill="currentColor"
                  viewBox="0 0 20 20"
                >
                  <path
                    d="M9.049 2.927c.3-.921 1.603-.921 1.902 0l1.07 3.292a1 1 0 00.95.69h3.462c.969 0 1.371 1.24.588 1.81l-2.8 2.034a1 1 0 00-.364 1.118l1.07 3.292c.3.921-.755 1.688-1.54 1.118l-2.8-2.034a1 1 0 00-1.175 0l-2.8 2.034c-.784.57-1.838-.197-1.539-1.118l1.07-3.292a1 1 0 00-.364-1.118L2.98 8.72c-.783-.57-.38-1.81.588-1.81h3.461a1 1 0 00.951-.69l1.07-3.292z"
                  />
                </svg>
                {movie.vote_average.toFixed(1)}
              </span>
              <button
                class="p-1 rounded-full bg-white/20 text-white hover:bg-white/40 transition-colors duration-200"
                on:click={(e) => toggleSave(e, movie.id)}
              >
                <svg
                  class="w-4 h-4"
                  fill={savedMovies.has(movie.id) ? "currentColor" : "none"}
                  stroke="currentColor"
                  viewBox="0 0 24 24"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z"
                  />
                </svg>
              </button>
            </div>
          </div>
        </div>
      {/each}
    </div>
  </div>
</main>

{#if selectedMovie}
  <div
    class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50"
    on:click={() => (selectedMovie = null)}
    on:keydown={(e) => e.key === "Escape" && (selectedMovie = null)}
    role="dialog"
    tabindex="-1"
    transition:fade={{ duration: 200 }}
  >
    <div
      class="bg-white rounded-lg p-6 max-w-sm sm:max-w-lg w-full"
      on:click|stopPropagation
      role="document"
      transition:fly={{ y: 50, duration: 300, easing: quintOut }}
    >
      <img
        src={getImageUrl(selectedMovie.backdrop_path)}
        alt={selectedMovie.title}
        class="w-full h-48 object-cover rounded-lg mb-4"
      />
      <h2 class="text-2xl font-bold mb-2 text-gray-800">
        {selectedMovie.title}
      </h2>
      <p class="mb-4 text-sm text-gray-600">{selectedMovie.overview}</p>
      <div class="flex justify-between items-center mb-4">
        <span class="text-yellow-500 font-bold"
          >{selectedMovie.vote_average.toFixed(1)}/10</span
        >
        <span class="text-gray-500"
          >Release: {new Date(
            selectedMovie.release_date
          ).toLocaleDateString()}</span
        >
      </div>
      <button
        class="w-full bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded transition-colors duration-300"
        on:click={() => (isPlaying = true)}
      >
        Play
      </button>
    </div>
  </div>
{/if}

{#if isPlaying}
  <div class="fixed inset-0 z-50">
    <div class="relative w-full h-full">
      <iframe
        src={getEmbedUrl(selectedMovie.id)}
        title={selectedMovie.title}
        class="absolute inset-0 w-full h-full"
        frameborder="0"
        allowfullscreen
      ></iframe>
      <div
        class="absolute top-0 left-0 p-4 flex items-center space-x-4 bg-black bg-opacity-60 rounded-br-lg"
      >
        <h3 class="text-white text-lg font-bold">{selectedMovie.title}</h3>
        <button
          class="text-white bg-black bg-opacity-40 hover:bg-opacity-60 px-4 py-2 rounded transition"
          on:click={() => (isPlaying = false)}
        >
          Close
        </button>
      </div>
    </div>
  </div>
{/if}

<style>
  :global(body) {
    font-family: Arial, sans-serif;
  }
  :global(*) {
    transition: all 0.3s ease;
  }
  :global(button) {
    cursor: pointer;
  }
  /* Responsive adjustments */
  @media (max-width: 640px) {
    h1 {
      font-size: 2rem; /* Responsive title size */
    }
  }
</style>
