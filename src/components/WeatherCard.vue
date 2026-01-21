<script setup>
    import { ref, computed } from 'vue'

    const props = defineProps({
        meteo: Object,
        favoris: Array,
        loading : Boolean
    })
    
    defineEmits(['ad'])

    const afficherRadar = ref(false)

    const estDejaFavori = computed(() => {
        if (!props.meteo || !props.favoris) return false
        return props.favoris.includes(props.meteo.location.name)
    })

    const radarUrl = computed(() => {
        if (!props.meteo) return ''
        const lat = props.meteo.location.lat
        const lon = props.meteo.location.lon
        return `https://embed.windy.com/embed2.html?lat=${lat}&lon=${lon}&detailLat=${lat}&detailLon=${lon}&width=650&height=450&zoom=9&level=surface&overlay=rain&product=ecmwf&menu=&message=true&marker=&calendar=now&pressure=&type=map&location=coordinates&detail=&metricWind=km%2Fh&metricTemp=%C2%B0C&radarRange=-1`
    })

    const getNomJour = (dateStr) => {
        const date = new Date(dateStr) // transforme en Mer. ou Jeu. ou etc
        return date.toLocaleDateString('fr-FR', { weekday: 'short' }) 
    }

      const meteoBackground = computed(() => {
      if (!props.meteo) return 'bg-gray-200'

      const code = props.meteo.current.condition.text.toLowerCase()
      const isDay = props.meteo.current.is_day === 1 

      if (!isDay) {
          return 'bg-gradient-to-br from-gray-900 via-blue-900 to-slate-900 text-white'
      }

      if (code.includes('rain') || code.includes('drizzle') || code.includes('storm')) {
          return 'bg-gradient-to-br from-slate-600 to-blue-800 text-white' // Pluie : Gris/Bleu
      }
      
      if (code.includes('snow') || code.includes('ice') || code.includes('blizzard')) {
          return 'bg-gradient-to-br from-blue-100 to-slate-300 text-gray-800' // Neige : Blanc froid
      }

      if (code.includes('sun') || code.includes('clear')) {
          return 'bg-gradient-to-br from-orange-400 to-yellow-300 text-white' // Soleil : Orange/Jaune
      }

      if (code.includes('cloud') || code.includes('overcast') || code.includes('mist') || code.includes('fog')) {
          return 'bg-gradient-to-br from-blue-400 to-gray-400 text-white' // Nuageux : Bleu gris
      }

      return 'bg-gradient-to-br from-blue-400 to-blue-600 text-white' 
  })

  const getHeuresRestantes = () => {
    if (!props.meteo) return []
    
        const localTimeStr = props.meteo.location.localtime // récupération des bonnes heures
        
        const heureActuelleVille = parseInt(localTimeStr.split(' ')[1].split(':')[0])

        const toutesLesHeures = props.meteo.forecast.forecastday[0].hour
        const heuresdemain = props.meteo.forecast.forecastday[1].hour

        const toutesHeuresDispo = [...toutesLesHeures,...heuresdemain]

        const prochaineHeures = toutesHeuresDispo.slice(heureActuelleVille)

        return prochaineHeures.slice(1,12)
    }
</script>

<template>
    <div class="w-full h-full relative overflow-hidden custom-scroll rounded-3xl">
        
        <div 
            v-if="meteo" 
            class="w-full h-full p-6 flex flex-col animate-fade-in transition-all duration-1000"
            :class="meteoBackground"
        >
            
            <div class="flex justify-between items-start mb-4 z-10">
                <div>
                    <h2 class="text-3xl font-bold leading-tight">
                        {{ meteo.location?.name }}
                    </h2>
                    <p class="text-sm font-medium">
                        {{ meteo.location?.country }}
                    </p>
                    <p class="text-xs mt-1">
                        {{ meteo.location?.localtime }}
                    </p>
                </div>

                <div class="flex items-center gap-2">
                    <button 
                        @click="afficherRadar = !afficherRadar"
                        class="bg-white/20 hover:bg-white/30 backdrop-blur-md px-3 py-2 rounded-xl text-xs font-bold transition duration-300 border border-white/10"
                    >
                        {{ afficherRadar ? 'Voir Infos' : 'Voir Carte' }}
                    </button>

                    <div v-if="estDejaFavori" class="button-check" title="Déjà ajouté">
                        <svg xmlns="http://www.w3.org/2000/svg" class="svgIcon" viewBox="0 0 448 512">
                            <path d="M438.6 105.4c12.5 12.5 12.5 32.8 0 45.3l-256 256c-12.5 12.5-32.8 12.5-45.3 0l-128-128c-12.5-12.5-12.5-32.8 0-45.3s32.8-12.5 45.3 0L160 338.7 393.4 105.4c12.5-12.5 32.8-12.5 45.3 0z"/>
                        </svg>
                    </div>
                    <button v-else title="Ajouter aux favoris" class="button-add group" @click="$emit('ad')">
                        <svg xmlns="http://www.w3.org/2000/svg" width="50px" height="50px" viewBox="0 0 24 24" class="svgIcon stroke-blue-400 fill-none group-hover:fill-blue-800">
                            <path d="M12 22C17.5 22 22 17.5 22 12C22 6.5 17.5 2 12 2C6.5 2 2 6.5 2 12C2 17.5 6.5 22 12 22Z" stroke-width="1.5"></path>
                            <path d="M8 12H16" stroke-width="1.5"></path>
                            <path d="M12 16V8" stroke-width="1.5"></path>
                        </svg>
                    </button>
                </div>
            </div>

            <div v-if="!afficherRadar" class="flex flex-col h-full animate-fade-in">
                
                <div class="flex flex-col items-center justify-center py-2 mb-4">
                    <div class="flex items-center justify-center">
                        <span class="text-7xl font-black tracking-tighter drop-shadow-lg">{{ meteo.current?.temp_c }}°</span>
                        <img :src="meteo.current?.condition?.icon" alt="Météo" class="w-20 h-20 ml-4 drop-shadow-md">
                    </div>
                    <p class="text-xl font-medium opacity-90 capitalize mt-2 bg-white/20 px-4 py-1 rounded-full">{{ meteo.current?.condition?.text }}</p>
                </div>

                <div class="grid grid-cols-2 gap-3 mb-4">
                    <div class="bg-white/20 backdrop-blur-md p-2 rounded-2xl flex flex-col items-center justify-center shadow-sm">
                        <span class="text-[10px] uppercase opacity-70 mb-1">Ressenti</span><span class="text-lg font-bold">{{ meteo.current?.feelslike_c }}°</span>
                    </div>
                    <div class="bg-white/20 backdrop-blur-md p-2 rounded-2xl flex flex-col items-center justify-center shadow-sm">
                        <span class="text-[10px] uppercase opacity-70 mb-1">Vent</span><span class="text-lg font-bold">{{ meteo.current?.wind_kph }} <span class="text-[10px] font-normal">km/h</span></span>
                    </div>
                    <div class="bg-white/20 backdrop-blur-md p-2 rounded-2xl flex flex-col items-center justify-center shadow-sm">
                        <span class="text-[10px] uppercase opacity-70 mb-1">Humidité</span><span class="text-lg font-bold">{{ meteo.current?.humidity }}%</span>
                    </div>
                    <div class="col-span-1 bg-white/20 backdrop-blur-md p-2 rounded-2xl flex flex-col items-center justify-center shadow-sm">
                        <span class="text-[10px] uppercase opacity-70 mb-1">Précipitation</span><span class="text-lg font-bold">{{ meteo.current?.precip_mm }} mm</span>
                    </div>
                </div>
                <div class="mb-4">
                    <p class="text-xs font-bold opacity-60 mb-2 uppercase tracking-wider">Aujourd'hui</p>
                    
                    <div class="flex overflow-x-auto gap-3 pb-2 custom-scroll">
                        
                        <div 
                            v-for="heure in getHeuresRestantes()" 
                            :key="heure.time_epoch"
                            class="flex-shrink-0 flex flex-col items-center bg-white/10 rounded-xl p-2 min-w-[60px]"
                        >
                            <span class="text-xs font-semibold opacity-80">
                                {{ heure.time.split(' ')[1] }}
                            </span>
                            
                            <img :src="heure.condition.icon" class="w-8 h-8 my-1" alt="icon">
                            
                            <span class="text-sm font-bold">{{ Math.round(heure.temp_c) }}°</span>
                            
                            <div v-if="heure.chance_of_rain >= 0" class="text-[9px] text-white-400 flex items-center mt-1">
                                💧 {{ heure.chance_of_rain   }}%
                            </div>
                        </div>

                    </div>
                </div>
                <div class="mt-auto">
                    <p class="text-xs font-bold opacity-60 mb-2 uppercase tracking-wider">Prochains jours</p>
                    
                    <div class="flex justify-between gap-2">
                        <div 
                            v-for="day in meteo.forecast.forecastday.slice(1, 6)" 
                            :key="day.date"
                            class="flex-1 bg-white/10 rounded-xl p-2 flex flex-col items-center justify-center hover:bg-white/20 transition cursor-default"
                        >
                            <span class="text-xs font-semibold capitalize mb-1">{{ getNomJour(day.date) }}</span>
                            
                            <img :src="day.day.condition.icon" class="w-8 h-8 mb-1" alt="icon">
                            
                            <div class="flex flex-col items-center leading-none">
                                <span class="text-sm font-bold">{{ day.day.maxtemp_c }}°</span>
                                <span class="text-[10px] opacity-60">{{ day.day.mintemp_c }}°</span>
                            </div>
                        </div>
                    </div>
                </div>

            </div>

            <div v-else class="flex-1 w-full h-full rounded-2xl overflow-hidden shadow-inner border border-white/20 animate-fade-in relative z-0">
                 <iframe :src="radarUrl" width="100%" height="100%" frameborder="0" class="w-full h-full"></iframe>
            </div>
        </div>

        <div v-else class="w-full h-full flex flex-col items-center justify-center text-gray-400 p-6 text-center animate-fade-in">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-16 w-16 mb-4 animate-bounce text-gray-300" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 11l5-5m0 0l5 5m-5-5v12" />
            </svg>
            <h2 class="text-xl font-bold text-gray-500">Veuillez entrer un nom de ville</h2>
            <p class="text-sm mt-2 opacity-60">Utilisez la barre de recherche ci-dessus pour commencer</p>
        </div>

    </div>
</template>

<style scoped>
.animate-fade-in { animation: fadeIn 0.3s ease-in-out; }
@keyframes fadeIn { from { opacity: 0; transform: translateY(5px); } to { opacity: 1; transform: translateY(0); } }

.button-add { width: 50px; height: 50px; border-radius: 50%; background-color: transparent; border: none; display: flex; align-items: center; justify-content: center; cursor: pointer; transition-duration: .3s; }
.button-add:hover { transform: scale(1.1); }
.button-check { width: 50px; height: 50px; border-radius: 50%; background-color: #22c55e; border: none; display: flex; align-items: center; justify-content: center; box-shadow: 0px 4px 10px rgba(34, 197, 94, 0.4); cursor: default; transition-duration: .3s; animation: popIn 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
.button-check .svgIcon { width: 20px; height: 20px; }
.button-check .svgIcon path { fill: white; }
@keyframes popIn { 0% { transform: scale(0); opacity: 0; } 100% { transform: scale(1); opacity: 1; } }
.button-add svg { width: 50px; height: 50px; }
</style>