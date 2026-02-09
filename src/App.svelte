<script>
  import { onMount } from 'svelte';

  // --- 1. STATE & VARIABLES ---
  const bookmarkletCode = "javascript:(function(){let s=0.8;let a=true;function g(){if(a){window.scrollBy(0,s);requestAnimationFrame(g);}}window.addEventListener('dblclick',()=>a=!a);g();})();";
  
  let deferredPrompt;      
  let showInstallModal = false; 
  let isListening = false;
  let recognition;         
  let speed = 0.8;         

  // --- 2. LIFECYCLE ON MOUNT ---
  onMount(() => {
    // Registrasi Service Worker
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker.register('/sw.js')
        .then(() => console.log("wscroll: Service Worker Active"))
        .catch(err => console.error("SW Registration Failed", err));
    }

    // Tangkap event instalasi PWA
    window.addEventListener('beforeinstallprompt', (e) => {
      e.preventDefault();
      deferredPrompt = e;
    });

    // Inisialisasi Speech Recognition
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    if (SpeechRecognition) {
      recognition = new SpeechRecognition();
      recognition.continuous = true;
      recognition.lang = 'en-US';

      recognition.onresult = (event) => {
        const command = event.results[event.results.length - 1][0].transcript.toLowerCase();
        console.log("Command:", command);
        if (command.includes("stop") || command.includes("pause")) isListening = false;
        if (command.includes("faster")) speed += 0.5;
        if (command.includes("slow")) speed = Math.max(0.1, speed - 0.5);
      };

      recognition.onend = () => { if(isListening) recognition.start(); };
    }
  });

  // --- 3. FUNCTIONS ---
  async function installPWA() {
    if (deferredPrompt) {
      deferredPrompt.prompt();
      const { outcome } = await deferredPrompt.userChoice;
      if (outcome === 'accepted') deferredPrompt = null;
    } else {
      alert("Please use your browser menu to 'Add to Home Screen'.");
    }
  }

  function toggleVoice() {
    // Mengecek apakah sudah dalam mode PWA (Standalone)
    const isPWA = window.matchMedia('(display-mode: standalone)').matches || window.navigator.standalone;

    if (!isPWA) {
      showInstallModal = true; // Jika dibuka di browser biasa, munculkan Modal
      return;
    }

    if (!recognition) {
      alert("Browser tidak mendukung Voice Recognition.");
      return;
    }
    
    isListening = !isListening;
    if (isListening) {
      recognition.start();
    } else {
      recognition.stop();
    }
  }
</script>

<main class="min-h-screen bg-slate-950 text-white flex flex-col items-center justify-center p-6 font-sans">
  <div class="max-w-3xl w-full">
    
    <header class="text-center mb-16">
      <h1 class="text-7xl font-black italic tracking-tighter mb-4 bg-gradient-to-r from-blue-500 to-cyan-400 bg-clip-text text-transparent">
        wscroll.
      </h1>
      <p class="text-slate-400 text-xl font-light tracking-tight">
        The world's lightest & most responsive auto-scroll tool.
      </p>
    </header>

    <div class="grid md:grid-cols-2 gap-8 items-stretch">
      
      <div class="bg-slate-900 border border-slate-800 p-8 rounded-3xl hover:border-blue-500 transition-all group flex flex-col">
        <div class="flex-grow">
          <h2 class="text-2xl font-bold mb-4 text-blue-400">wscroll Lite</h2>

          <div class="mb-8 text-center bg-slate-950/50 py-4 rounded-2xl border border-slate-800/50">
      <span class="text-4xl font-black text-white uppercase tracking-widest">Free</span>
    </div>

    <p class="text-slate-500 text-sm mb-10 leading-relaxed text-left">
      No installation. Simply drag the button below to your browser's bookmark bar and start scrolling.
    </p>
  </div>
        
        <div class="mt-auto">
          <a href={bookmarkletCode} 
             on:click|preventDefault={() => alert('Drag this button to your Bookmark Bar!')}
             class="block w-full py-4 text-center bg-blue-600 hover:bg-blue-500 rounded-2xl font-black shadow-lg shadow-blue-500/20 active:scale-95 transition-all uppercase tracking-widest text-white">
            wscroll
          </a>
          <p class="text-[10px] text-slate-600 mt-4 text-center uppercase tracking-[0.2em] font-bold italic">
            Drag to bookmarks ↑
          </p>
        </div>
      </div>

        
        
      <div class="bg-slate-900 border border-slate-700 p-8 rounded-3xl hover:border-blue-500 transition-all relative overflow-hidden flex flex-col shadow-2xl">
  <div class="absolute -top-2 -right-2 bg-yellow-400 text-black text-[10px] font-black px-5 py-2 rotate-12 shadow-[0_0_15px_rgba(250,204,21,0.6)] z-10">
    PRO
  </div>
  
  <div class="flex-grow">
    <h2 class="text-2xl font-bold mb-6 text-blue-300 text-left">wscroll Elite</h2>
    
    <div class="mb-8 text-center bg-slate-950 py-4 rounded-2xl border border-slate-800">
      <span class="text-4xl font-black text-white">$0.99</span>
      <span class="text-slate-300 text-sm italic">/month</span>
    </div>

    <p class="text-slate-300 text-sm mb-10 leading-relaxed text-left font-medium">
      Hands-free Voice Commands, AI-speed, and standalone PWA experience.
    </p>
  </div>
  
  <div class="mt-auto">
    <button on:click={installPWA}
       class="w-full py-4 text-center bg-blue-600 hover:bg-blue-500 rounded-2xl font-black shadow-lg shadow-blue-500/40 active:scale-95 transition-all uppercase tracking-widest text-white">
      Subscribe Now
    </button>
    <p class="text-[11px] text-slate-400 mt-4 text-center uppercase tracking-[0.1em] font-bold italic">
      7-Day Free Trial Included
    </p>
  </div>
</div>

    <footer class="mt-20 text-center border-t border-slate-900 pt-8">
      <p class="text-slate-500 text-sm font-medium">
        Usage: <span class="text-slate-300">Double Click</span> anywhere to Play/Pause.
      </p>
      <p class="mt-6 text-slate-800 text-[10px] tracking-[0.3em] uppercase font-black">
        wscroll engine &copy; 2026
      </p>
    </footer>

  </div>
</main>

<style>
  /* Optimization for drag and drop */
  a {
    user-select: none;
    -webkit-user-drag: element;
  }
</style>