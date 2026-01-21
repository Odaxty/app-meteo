<script setup>
  import { onMounted, ref } from 'vue'
  import WeatherCard from './components/WeatherCard.vue'
  import SearchBar from './components/SearchBar.vue'
  import Favories from './components/Favories.vue'
  import WelcomeView from './components/WelcomeView.vue'

  const meteo = ref(null) 
  const favoris = ref([]) 
  const ongletActif = ref('meteo') 
  const chargement = ref(false)
  const messageErreur = ref(null)
  const messageValide = ref(null)
  const afficherIntro = ref(true)

  const entrerDansLApp = () => {
      afficherIntro.value = false
  }


  const ajoutFavori = () => {
    if (meteo.value === null) return
    const nomVille = meteo.value.location.name
    if (favoris.value.includes(nomVille)) return
    favoris.value.push(nomVille)
    localStorage.setItem('mes-villes', JSON.stringify(favoris.value))
  }

  const dellfav = () => {
      localStorage.removeItem('mes-villes')
      favoris.value = []
  }

  const dellSpe = (villesupp) => {
    favoris.value = favoris.value.filter(v => v != villesupp)
    localStorage.setItem('mes-villes', JSON.stringify(favoris.value))
  }

  const rechercherMeteo = async (villeRecue) => {
    try {
      messageErreur.value=null
      messageErreur.value=null
      chargement.value = true
      const response = await fetch(
        `https://api.weatherapi.com/v1/forecast.json?key=${import.meta.env.VITE_API_KEY}&q=${villeRecue}&days=6&aqi=no&alerts=no`      
      )
      if (!response.ok) {
        throw new Error("Ville introuvable")
      }

      const data = await response.json()
      meteo.value = data
      ongletActif.value = 'meteo'
      messageValide.value= `Ville trouvée : ${data.location.name} !`
      setTimeout(() => {
          messageValide.value = null
      }, 3000)

    } catch (error) {
      console.error("Erreur :", error)
      messageErreur.value = "Oups ! Ville introuvable. Vérifiez l'orthographe."
      setTimeout(() => {
          messageErreur.value = null
      }, 3000)
    } finally {
        chargement.value = false
    }
  }

  onMounted(() => {
    const sauvegarde = localStorage.getItem('mes-villes')
    if (sauvegarde){
      favoris.value = JSON.parse(sauvegarde)
    }
  })

  const rechercherPosition = () => {
    if (!navigator.geolocation) {
      alert("Votre navigateur ne supporte pas la géolocalisation.")
      return
    }
    chargement.value = true

    navigator.geolocation.getCurrentPosition(
      (position) => {
        const lat = position.coords.latitude
        const lon = position.coords.longitude
        
        const q = `${lat},${lon}`
        
        rechercherMeteo(q)
      },
      (error) => {
        console.error("Erreur géolocalisation :", error)
        chargement.value = false 
        alert("Impossible de vous géolocaliser. Vérifiez vos autorisations.")
      }
    )
  }
</script>

<template>
  <Transition name="fade" mode="out-in">
    <WelcomeView v-if="afficherIntro" @starts="entrerDansLApp"/>
<main v-else class="w-full bg-gray-100 min-h-screen font-sans p-4 pb-28 md:p-8 md:flex md:flex-col md:items-center md:justify-center">    
  <div 
        v-if="messageErreur" 
        class="fixed top-6  transform -translate-x-1/2 z-50 bg-red-500 text-white px-6 py-3 rounded-full shadow-2xl flex items-center gap-3 animate-bounce"
    >
        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
        
        <span class="font-bold text-sm">{{ messageErreur }}</span>
    </div>

    <div 
        v-if="messageValide" 
        class="fixed top-6  transform -translate-x-1/2 z-50 bg-green-500 text-white px-6 py-3 rounded-full shadow-2xl flex items-center gap-3 animate-bounce"
    >
        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
        
        <span class="font-bold text-sm">{{ messageValide }}</span>
    </div>
    <div class="grid grid-cols-1 md:grid-cols-3 gap-6 w-full max-w-6xl h-auto">

      <div 
        class="flex-col gap-4 md:gap-6 md:col-span-2"
        :class="ongletActif === 'meteo' ? 'flex' : 'hidden md:flex'"
      >
        <div class="bg-white rounded-3xl shadow-sm p-2 shrink-0">
           <SearchBar @search="rechercherMeteo" @locate="rechercherPosition" :loading="chargement"/>
        </div>

        <div class="rounded-3xl shadow-lg relative overflow-hidden bg-gray-200 min-h-[500px]">
            <WeatherCard :meteo="meteo" :favoris="favoris" :loading="chargement" @ad="ajoutFavori"/>
        </div>
      </div>

      <div 
        class="bg-white rounded-3xl shadow-lg p-6 flex-col md:col-span-1"
        :class="ongletActif === 'favoris' ? 'flex min-h-[500px]' : 'hidden md:flex'"
      >
         <Favories 
            :favoris="favoris" 
            @suppall="dellfav" 
            @suppspe="dellSpe" 
            @choisir="rechercherMeteo"
         />
      </div>

    </div>

    <nav class="fixed bottom-6 left-1/2 transform -translate-x-1/2 w-[90%] max-w-xs bg-white/90 backdrop-blur-md shadow-2xl rounded-full p-2 flex justify-between md:hidden z-50 border border-white/50">
        
        <button 
            @click="ongletActif = 'meteo'"
            class="flex-1 py-3 rounded-full text-sm font-bold transition-all duration-300 flex items-center justify-center gap-2"
            :class="ongletActif === 'meteo' ? 'bg-blue-500 text-white shadow-md' : 'text-gray-500 hover:bg-gray-100'"
        >
            <span>☁️</span> Météo
        </button>

        <button 
            @click="ongletActif = 'favoris'"
            class="flex-1 py-3 rounded-full text-sm font-bold transition-all duration-300 flex items-center justify-center gap-2"
            :class="ongletActif === 'favoris' ? 'bg-blue-500 text-white shadow-md' : 'text-gray-500 hover:bg-gray-100'"
        >
            <span>⭐</span> Villes
        </button>

    </nav>

    <p class="mt-10 text-center font-bold text-gray-400 text-sm">
      By @Théo CHAUVIERE
    </p>

  </main>
  </Transition>
</template>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>