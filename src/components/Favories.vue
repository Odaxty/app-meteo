<script setup>
    defineProps({
        favoris: Array 
    })
    defineEmits(['suppspe', 'suppall', 'choisir'])
</script>

<template>
    <div class="flex flex-col h-full w-full">
        <h3 class="text-xl font-bold text-gray-800 mb-6 px-2">Mes Villes</h3>

        <ul class="flex-1 overflow-y-auto custom-scroll space-y-3 pr-2">
            
            <li v-for="ville in favoris" :key="ville" 
                class="group flex justify-between items-center bg-gray-50 rounded-2xl hover:bg-blue-50 transition-colors border border-transparent hover:border-blue-100 overflow-hidden pl-0 py-0 pr-3">
                <button 
                    @click="$emit('choisir', ville)" 
                    class="w-full h-full p-3 font-semibold text-gray-700 group-hover:text-blue-600 text-left flex-1 truncate outline-none"
                > 
                    {{ ville }}
                </button>
                
                <button class="button-delete shrink-0" @click="$emit('suppspe', ville)">
                    <svg viewBox="0 0 448 512" class="svgIcon"><path d="M135.2 17.7L128 32H32C14.3 32 0 46.3 0 64S14.3 96 32 96H416c17.7 0 32-14.3 32-32s-14.3-32-32-32H320l-7.2-14.3C307.4 6.8 296.3 0 284.2 0H163.8c-12.1 0-23.2 6.8-28.6 17.7zM416 128H32L53.2 467c1.6 25.3 22.6 45 47.9 45H346.9c25.3 0 46.3-19.7 47.9-45L416 128z"></path></svg>
                </button>

            </li>

            <li v-if="favoris.length === 0" class="text-center text-gray-400 mt-10 italic">
                Aucun favori pour l'instant
            </li>

        </ul>

        <div v-if="favoris.length > 0" class="mt-6 border-t pt-4">
            <button 
                @click="$emit('suppall')"
                class="w-full py-3 rounded-xl text-sm font-bold text-red-500 hover:bg-red-50 transition-colors uppercase tracking-wider"
            >
                Tout supprimer
            </button>
        </div>
    </div>
</template>

<style scoped>
.custom-scroll::-webkit-scrollbar {
  display: none;
}
.custom-scroll {
  -ms-overflow-style: none;  
  scrollbar-width: none;  
}

.button-delete {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background-color: rgb(20, 20, 20);
  border: none;
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0px 0px 20px rgba(0, 0, 0, 0.164);
  cursor: pointer;
  transition-duration: .3s;
  overflow: hidden;
  position: relative;
  flex-shrink: 0; 
}

.svgIcon {
  width: 12px;
  transition-duration: .3s;
}

.svgIcon path {
  fill: white;
}

.button-delete:hover {
  width: 90px;
  border-radius: 50px;
  transition-duration: .3s;
  background-color: rgb(255, 69, 69);
  align-items: center;
}

.button-delete:hover .svgIcon {
  width: 50px;
  transition-duration: .3s;
  transform: translateY(60%);
}

.button-delete::before {
  position: absolute;
  top: -20px;
  content: "Delete";
  color: white;
  transition-duration: .3s;
  font-size: 2px;
}

.button-delete:hover::before {
  font-size: 13px;
  opacity: 1;
  transform: translateY(30px);
  transition-duration: .3s;
}
</style>