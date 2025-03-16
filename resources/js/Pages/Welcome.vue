<script setup>
import ApplicationLogo from '@/Components/ApplicationLogo.vue';
import { getRandomButtonStyle } from '@/Hooks/useButtons';
import { Head, Link } from '@inertiajs/vue3';
import { onMounted, ref } from 'vue';
import Loading from "vue-loading-overlay";
import "vue-loading-overlay/dist/css/index.css";

defineProps({
    canLogin: {
        type: Boolean,
    },
    canRegister: {
        type: Boolean,
    },
    laravelVersion: {
        type: String,
        required: true,
    },
    phpVersion: {
        type: String,
        required: true,
    },
});

const isLoading = ref(false);
const search = ref(null);
const types = ref(null);
const selectedType = ref(null);
const pokemons = ref(null);
const pokemonDetails = ref(null);

const getTypes = async () => {
    const response = await fetch('https://pokeapi.co/api/v2/type');
    types.value = await response.json();
};

const getPokemons = async (type) => {
    try {
        selectedType.value = type.name;
        isLoading.value = true;
        pokemonDetails.value = null;
        const url = type.url;
        const response = await fetch(url);
        const data = await response.json();
        const pokemonsWithDetails = await Promise.all(
            data.pokemon.map(async (item) => {
                const pokemonDetailResponse = await fetch(item.pokemon.url);
                const pokemonDetails = await pokemonDetailResponse.json();
                return {
                    ...item.pokemon,
                    details: pokemonDetails
                };
            })
        );

        pokemons.value = pokemonsWithDetails;
        isLoading.value = false;
    } catch (error) {
        isLoading.value = false;
    }
};


const getDetails = async (url) => {
    try {
        isLoading.value = true;
        const response = await fetch(url);
        const data = await response.json();
        pokemonDetails.value = data;
        pokemons.value = null;
        isLoading.value = false;
    } catch (error) {
        pokemons.value = null;
        pokemonDetails.value = null;
        isLoading.value = false;
    }


};

onMounted(() => {
    getTypes();
    const scrollToTopButton = document.getElementById("scrollToTop");

    // Mostrar el botón cuando se haga scroll hacia abajo
    window.onscroll = function () {
        if (document.body.scrollTop > 100 || document.documentElement.scrollTop > 100) {
            scrollToTopButton.classList.remove("hidden");
        } else {
            scrollToTopButton.classList.add("hidden");
        }
    };

    // Función para subir al inicio de la página
    scrollToTopButton?.addEventListener("click", function () {
        window.scrollTo({ top: 0, behavior: "smooth" });
    });
});
</script>
<template>
    <div class="vl-parent">
        <loading v-model:active="isLoading" :can-cancel="false" :is-full-page="true" />
    </div>

    <Head title="Welcome" />
    <div class="bg-black text-black/50 dark:bg-black dark:text-white/50 min-h-screen">
        <img id="background" class="absolute -left-20 top-0 max-w-screen-md"
            src="https://laravel.com/assets/img/welcome/background.svg" />
        <div class="relative flex flex-col items-center justify-center w-full">
            <div class="relative w-full max-w-2xl px-6 lg:max-w-7xl">
                <header class="py-10">
                    <div class="flex justify-center">
                        <ApplicationLogo class="h-28 w-auto text-white lg:h-36" />
                    </div>
                </header>

                <main class="mt-6 mb-48">
                    <!-- search -->
                    <form class="flex items-center max-w-4xl mx-auto"
                        @submit.prevent="getDetails(`https://pokeapi.co/api/v2/pokemon/${search}`)">
                        <label for="voice-search" class="sr-only">Search</label>
                        <div class="relative w-full">
                            <div class="absolute inset-y-0 start-0 flex items-center ps-3 pointer-events-none">
                                <img src="/img/pokeball.png" alt="Search icon" class="w-5 h-5" />
                            </div>
                            <input v-model="search" type="text" id="voice-search"
                                class="bg-gray-50 border border-gray-300 text-gray-900 text-base rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full ps-10 p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
                                placeholder="Search Name, Id Type" required />
                            <button type="button" class="absolute inset-y-0 end-0 flex items-center pe-3">
                                <svg class="w-4 h-4 text-gray-500 dark:text-gray-400 hover:text-gray-900 dark:hover:text-white"
                                    aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none"
                                    viewBox="0 0 16 20">
                                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"
                                        stroke-width="2"
                                        d="M15 7v3a5.006 5.006 0 0 1-5 5H6a5.006 5.006 0 0 1-5-5V7m7 9v3m-3 0h6M7 1h2a3 3 0 0 1 3 3v5a3 3 0 0 1-3 3H7a3 3 0 0 1-3-3V4a3 3 0 0 1 3-3Z" />
                                </svg>
                            </button>
                        </div>
                        <button type="submit" @click="getDetails(`https://pokeapi.co/api/v2/pokemon/${search}`)"
                            class="inline-flex items-center py-2.5 px-3 ms-2 text-base font-medium text-white bg-blue-700 rounded-lg border border-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800">
                            <svg class="w-4 h-4 me-2" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none"
                                viewBox="0 0 20 20">
                                <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"
                                    stroke-width="2" d="m19 19-4-4m0-7A7 7 0 1 1 1 8a7 7 0 0 1 14 0Z" />
                            </svg>Search
                        </button>
                    </form>

                    <!-- buttons -->
                    <div class="flex flex-wrap justify-start gap-0 my-6">
                        <button v-for="type in types?.results" :key="type.name"
                            :disabled="selectedType == type.name ? true : false" :class="[
                                selectedType == type.name ? 'relative inline-flex items-center justify-center p-0.5 mb-2 me-2 overflow-hidden text-sm font-medium cursor-not-allowed' : getRandomButtonStyle()
                            ]" @click="getPokemons(type)">
                            <span
                                class="relative px-5 py-2.5 transition-all ease-in duration-75 bg-white dark:bg-gray-900 rounded-md group-hover:bg-transparent group-hover:dark:bg-transparent">
                                {{ type.name }}
                            </span>
                        </button>
                    </div>

                    <!-- pokemons -->
                    <div v-if="pokemonDetails"
                        class="flex flex-col md:flex-row items-start bg-white border border-gray-200 rounded-lg shadow-sm hover:bg-gray-100 dark:border-gray-700 dark:bg-gray-950 dark:hover:bg-gray-900 w-full max-w-full">
                        <div class="w-full md:w-1/2 justify-center flex">
                            <img class="object-cover rounded-t-lg md:rounded-none md:rounded-l-lg w-44 md:w-96"
                                :src="pokemonDetails.sprites.other.dream_world.front_default || pokemonDetails.sprites.front_default"
                                alt="">
                        </div>
                        <div class="flex flex-col justify-between p-6 w-full md:w-1/2 mx-4">
                            <div class="justify-center flex items-start gap-3">
                                <h5 class="text-5xl font-bold text-center text-gray-900 dark:text-white">
                                    {{ pokemonDetails.name }}
                                </h5>
                                <p class="text-2xl text-slate-700 dark:text-slate-300">#{{ pokemonDetails?.id }}</p>
                            </div>
                            <div class="flex flex-col gap-4 text-base text-gray-500 dark:text-white/70">
                                <div>
                                    <strong>Abilities:</strong>
                                    <ul class="list-disc pl-4">
                                        <li v-for="ability in pokemonDetails.abilities" :key="ability.ability.name">
                                            {{ ability.ability.name }}
                                        </li>
                                    </ul>
                                </div>
                                <div>
                                    <strong>weight:</strong>
                                    <p>{{ pokemonDetails.weight / 10 }} kg.</p>
                                </div>
                                <div>
                                    <strong>height:</strong>
                                    <p>{{ pokemonDetails.height / 10 }} m.</p>
                                </div>
                            </div>
                        </div>
                    </div>

                    <div class="grid grid-cols-1 gap-2 md:grid-cols-2 lg:grid-cols-3 mb-5"
                        v-else-if="pokemons?.length > 0">
                        <button v-for="item in pokemons" :key="item.name" @click="getDetails(item.url)"
                            class="flex flex-col items-center bg-gradient-to-br from-black to-yellow-500/5 text-white border border-gray-200 rounded-lg shadow-sm md:flex-row md:max-w-xl hover:bg-gray-700 dark:border-gray-700 dark:bg-gray-800 dark:hover:bg-black">
                            <img class="object-cover w-auto rounded-t-lg h-56 md:h-auto md:w-48 md:rounded-none md:rounded-s-lg"
                                :src="item.details.sprites.other.dream_world.front_default || item.details.sprites.front_default"
                                alt="">
                            <div class="flex flex-col justify-end p-4 leading-normal">
                                <h5 class="mb-2 text-xl font-bold tracking-tight text-gray-900 dark:text-white">
                                    {{ item.name }}
                                </h5>
                                <div class="mb-3" v-if="item.details && item.details.abilities">
                                    <div class="text-base text-gray-500 dark:text-white/70">
                                        <p class="text-slate-700 dark:text-slate-300">#{{ item.details.id }}</p>
                                        <strong>Abilities:</strong>
                                        <ul>
                                            <li v-for="ability in item.details.abilities" :key="ability.ability.name">
                                                {{ ability.ability.name }}
                                            </li>
                                        </ul>
                                    </div>
                                </div>
                            </div>
                        </button>
                    </div>
                    <div v-else class="text-center py-24 text-gray-500 dark:text-slate-400">
                        <p>
                            There is nothing here
                        </p>
                    </div>
                </main>

                <!-- footer -->
                <footer class="pt-5 pb-2 text-center text-sm text-black bg-black dark:bg-gray-800">
                    <p>
                        Pokedex by <span class="font-bold">Javier Villagran</span>
                    </p>
                </footer>

                <button id="scrollToTop"
                    class="animate-bounce animate-once animate-duration-500 hidden fixed bottom-20 right-10 bg-yellow-500 text-white p-3 rounded-full shadow-lg hover:bg-yellow-600 transition-all duration-200">
                    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" width="32" height="32"
                        stroke-width="1.5" stroke-linejoin="round" stroke-linecap="round" stroke="currentColor">
                        <path d="M12 5v6m0 3v1.5m0 3v.5"></path>
                        <path d="M16 9l-4 -4"></path>
                        <path d="M8 9l4 -4"></path>
                    </svg>
                </button>
            </div>
        </div>
    </div>
</template>

<style scoped>
/* Footer */
footer {
    background-color: transparent;
    color: white;
    position: fixed;
    bottom: 0;
    left: 0;
    width: 100%;
    text-align: center;
    z-index: 10;
}

#scrollToTop {
    transition: all 0.3s ease-in-out;
}

#scrollToTop.hidden {
    display: none;
}
</style>
