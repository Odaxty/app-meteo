<script setup>
    import { ref, onMounted } from 'vue'

    defineEmits(['starts'])

    const deferredPrompt = ref(null)
    const afficherBoutonInstall = ref(false)

    onMounted(() => {
        window.addEventListener('beforeinstallprompt', (e) => {
            e.preventDefault()
            deferredPrompt.value = e
            afficherBoutonInstall.value = true
        })
    })

    const installerPWA = async () => {
        if (!deferredPrompt.value) return
        deferredPrompt.value.prompt()
        const { outcome } = await deferredPrompt.value.userChoice
        deferredPrompt.value = null
        afficherBoutonInstall.value = false
    }
</script>

<template>
    <div class="min-h-screen w-full bg-gray-100 flex flex-col items-center justify-center p-6 text-gray-800 relative overflow-hidden font-sans">
        
        <div class="absolute top-0 left-0 w-96 h-96 bg-blue-300 rounded-full mix-blend-multiply filter blur-3xl opacity-30 animate-blob"></div>
        <div class="absolute bottom-0 right-0 w-96 h-96 bg-indigo-300 rounded-full mix-blend-multiply filter blur-3xl opacity-30 animate-blob animation-delay-2000"></div>

        <div class="z-10 text-center max-w-5xl w-full">
            
            <h1 class="text-6xl md:text-8xl font-black tracking-tighter mb-4 drop-shadow-sm animate-fade-in-up text-gray-900">
                Météo<span class="text-blue-500">Bento</span>.
            </h1>
            
            <p class="text-xl md:text-2xl text-gray-500 mb-10 font-medium animate-fade-in-up delay-100">
                L'élégance de la météo, réinventée.
            </p>

            <div class="flex flex-col items-center mb-16 animate-fade-in-up delay-200">
                <div class="flex flex-col md:flex-row gap-4 justify-center items-center">
                    
                    <button 
                        @click="$emit('starts')"
                        class="group relative inline-flex items-center justify-center px-8 py-4 text-lg font-bold text-white transition-all duration-200 bg-blue-600 rounded-full hover:bg-blue-700 hover:shadow-lg hover:-translate-y-1 shadow-blue-500/30 shadow-md animate-bounce-slow"
                    >
                        Accéder à l'application 
                        <svg class="w-5 h-5 ml-2 transition-transform group-hover:translate-x-1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
                            <path fill-rule="evenodd" d="M10.293 3.293a1 1 0 011.414 0l6 6a1 1 0 010 1.414l-6 6a1 1 0 01-1.414-1.414L14.586 11H3a1 1 0 110-2h11.586l-4.293-4.293a1 1 0 010-1.414z" clip-rule="evenodd" />
                        </svg>
                    </button>

                    <button 
                        v-if="afficherBoutonInstall"
                        @click="installerPWA"
                        class="group relative inline-flex items-center justify-center px-6 py-4 text-base font-bold text-gray-700 transition-all duration-200 bg-white border border-gray-200 rounded-full hover:bg-gray-50 hover:text-blue-600 hover:shadow-md hover:-translate-y-1 shadow-sm"
                    >
                        📱 Installer l'app
                    </button>
                </div>

                <p class="text-sm text-gray-400 mt-6 md:hidden animate-pulse bg-white/50 px-4 py-2 rounded-full backdrop-blur-sm">
                    Sur iPhone ? Appuyez sur <span class="font-bold text-gray-600">Partager</span> puis <span class="font-bold text-gray-600">"Sur l'écran d'accueil"</span>.
                </p>
            </div>
            
            <div class="grid grid-cols-1 md:grid-cols-3 gap-6 animate-fade-in-up delay-200">
                
                <div class="bg-white p-8 rounded-3xl shadow-lg hover:shadow-xl transition duration-300 border border-gray-100 flex flex-col items-center">
                    <div class="bg-blue-50 w-16 h-16 rounded-2xl flex items-center justify-center text-4xl mb-4 text-blue-500">🌍</div>
                    <h3 class="text-xl font-bold mb-2 text-gray-800">Monde Entier</h3>
                    <p class="text-sm text-gray-500 leading-relaxed">Accédez aux prévisions de n'importe quelle ville instantanément.</p>
                </div>

                <div class="bg-white p-8 rounded-3xl shadow-lg hover:shadow-xl transition duration-300 border border-gray-100 flex flex-col items-center">
                    <div class="bg-yellow-50 w-16 h-16 rounded-2xl flex items-center justify-center text-4xl mb-4 text-yellow-500">⭐</div>
                    <h3 class="text-xl font-bold mb-2 text-gray-800">Favoris</h3>
                    <p class="text-sm text-gray-500 leading-relaxed">Ajoutez vos villes préférées pour y accéder en un clic.</p>
                </div>

                <div class="bg-white p-8 rounded-3xl shadow-lg hover:shadow-xl transition duration-300 border border-gray-100 flex flex-col items-center">
                    <div class="bg-purple-50 w-16 h-16 rounded-2xl flex items-center justify-center text-4xl mb-4 text-purple-500">⚡</div>
                    <h3 class="text-xl font-bold mb-2 text-gray-800">Temps Réel</h3>
                    <p class="text-sm text-gray-500 leading-relaxed">Données actualisées en direct avec géolocalisation précise.</p>
                </div>

            </div>

        </div>
        
        <div class="absolute bottom-6 text-gray-400 text-sm font-semibold tracking-wide opacity-50">
            By @Théo CHAUVIERE
        </div>

    </div>
</template>

<style scoped>
.animate-fade-in-up {
    animation: fadeInUp 0.8s cubic-bezier(0.16, 1, 0.3, 1) forwards;
    opacity: 0;
    transform: translateY(30px);
}
.delay-100 { animation-delay: 0.1s; }
.delay-200 { animation-delay: 0.2s; }

@keyframes fadeInUp {
    to { opacity: 1; transform: translateY(0); }
}

.animate-blob {
    animation: blob 10s infinite;
}
.animation-delay-2000 {
    animation-delay: 4s;
}

@keyframes blob {
    0% { transform: translate(0px, 0px) scale(1); }
    33% { transform: translate(30px, -50px) scale(1.1); }
    66% { transform: translate(-20px, 20px) scale(0.9); }
    100% { transform: translate(0px, 0px) scale(1); }
}

.animate-bounce-slow {
    animation: bounce 3s infinite;
}
</style>