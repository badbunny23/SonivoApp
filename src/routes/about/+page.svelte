<script lang="ts">
    import IconMusic from '~icons/mdi/music-note';
    import IconPiano from '~icons/mdi/piano';
    import IconCode from '~icons/mdi/code-braces';
    import IconBrain from '~icons/mdi/brain';
    import { goto } from '$app/navigation';
</script>

<svelte:head>
    <title>About | Sonivo</title>
</svelte:head>

<main class="min-h-screen bg-zinc-950">
    <!-- Navigation Bar -->
    <nav class="fixed w-full top-0 bg-zinc-800 border-b border-white/5 z-50">
        <div class="max-w-7xl mx-auto px-4">
            <div class="flex items-center justify-between h-16">
                <a href="/" class="text-2xl font-bold bg-gradient-to-r from-white to-white/70 bg-clip-text text-transparent">
                    Sonivo
                </a>
                <div class="flex items-center gap-8">
                    <a href="/" class="text-white/70 hover:text-white text-sm font-medium transition-colors">Home</a>
                    <a href="/about" class="text-white/90 hover:text-white text-sm font-medium transition-colors">About</a>
                    <button
                        class="bg-white text-zinc-900 rounded-full px-6 py-2 
                               text-sm font-medium transition-all duration-300
                               hover:bg-white/90 hover:scale-[1.02]"
                        on:click={() => {
                            goto('/?start=true', { replaceState: true });
                        }}
                    >
                        Start Creating
                    </button>
                </div>
            </div>
        </div>
    </nav>

    <!-- Hero Section -->
    <section class="pt-32 pb-16 px-4 sm:px-6 lg:px-8">
        <div class="max-w-7xl mx-auto">
            <h1 class="text-5xl md:text-6xl font-bold tracking-tight mb-8">
                <span class="bg-gradient-to-r from-white to-white/70 bg-clip-text text-transparent">
                    Behind the Scenes
                </span>
            </h1>
            <p class="text-lg text-zinc-400 max-w-2xl">
                Sonivo combines music theory, modern web technologies, and intuitive design to create 
                an accessible platform for music composition. Here's our story and how it all works.
            </p>
        </div>
    </section>

    <!-- Background Section -->
    <section class="py-16 bg-white/5">
        <div class="max-w-7xl mx-auto px-4">
            <div class="grid md:grid-cols-2 gap-16 items-center">
                <div>
                    <h2 class="text-3xl font-bold text-white/90 mb-6">The Background</h2>
                    <div class="space-y-4 text-zinc-400">
                        <p>
                            Music composition has traditionally been accessible only to those with years 
                            of formal training. We wanted to change that by creating a tool that makes 
                            composing music intuitive and enjoyable for everyone.
                        </p>
                        <p>
                            Our goal was to create a simple platform that makes music composition 
                            accessible and fun, helping users learn basic music concepts along the way.
                        </p>
                    </div>
                </div>
                <div class="grid grid-cols-2 gap-4">
                    <img 
                        src="/piano.jpg" 
                        alt="Piano keys" 
                        class="aspect-square object-cover rounded-lg"
                    />
                    <img 
                        src="/graph.png" 
                        alt="Music visualization graph" 
                        class="aspect-square object-cover rounded-lg"
                    />
                    <img 
                        src="/hi.png" 
                        alt="Interface preview" 
                        class="aspect-square object-cover rounded-lg"
                    />
                    <img 
                        src="/mm.png" 
                        alt="Music visualization" 
                        class="aspect-square object-cover rounded-lg"
                    />
                </div>
            </div>
        </div>
    </section>

    <!-- Technical Deep Dive -->
    <section class="py-16 px-4 sm:px-6 lg:px-8">
        <div class="max-w-7xl mx-auto">
            <h2 class="text-3xl font-bold text-white/90 mb-12 text-center">Technical Part</h2>
            <div class="grid sm:grid-cols-2 lg:grid-cols-4 gap-8">
                <div class="bg-white/5 rounded-xl p-6 backdrop-blur-sm">
                    <div class="w-12 h-12 rounded-lg bg-white/10 flex items-center justify-center mb-4">
                        <IconMusic class="text-2xl text-white/80" />
                    </div>
                    <h3 class="text-lg font-semibold text-white/90 mb-3">Choose Your Key</h3>
                    <p class="text-zinc-400">
                        Every time you select a key, we generate a new unique set of notes that work well together.
                    </p>
                </div>
                <div class="bg-white/5 rounded-xl p-6 backdrop-blur-sm">
                    <div class="w-12 h-12 rounded-lg bg-white/10 flex items-center justify-center mb-4">
                        <IconPiano class="text-2xl text-white/80" />
                    </div>
                    <h3 class="text-lg font-semibold text-white/90 mb-3">Sound Synthesis</h3>
                    <p class="text-zinc-400">
                        Built on Tone.js for high-quality audio synthesis and precise timing control 
                        in the browser.
                    </p>
                </div>
                <div class="bg-white/5 rounded-xl p-6 backdrop-blur-sm">
                    <div class="w-12 h-12 rounded-lg bg-white/10 flex items-center justify-center mb-4">
                        <IconCode class="text-2xl text-white/80" />
                    </div>
                    <h3 class="text-lg font-semibold text-white/90 mb-3">Modern Stack</h3>
                    <p class="text-zinc-400">
                        Developed using SvelteKit, TypeScript, and TailwindCSS for a fast, 
                        responsive experience.
                    </p>
                </div>
                <div class="bg-white/5 rounded-xl p-6 backdrop-blur-sm">
                    <div class="w-12 h-12 rounded-lg bg-white/10 flex items-center justify-center mb-4">
                        <IconBrain class="text-2xl text-white/80" />
                    </div>
                    <h3 class="text-lg font-semibold text-white/90 mb-3">Smart Algorithms</h3>
                    <p class="text-zinc-400">
                        Intelligent note suggestion system based on music theory principles and 
                        pattern recognition.
                    </p>
                </div>
            </div>
        </div>
    </section>

    <!-- Footer -->
    <footer class="py-12 mt-20 border-t border-white/5">
        <div class="max-w-7xl mx-auto px-4 text-center">
            <p class="text-zinc-500">© 2024 Sonivo. Developed by Het, Kabir, Nathan</p>
        </div>
    </footer>
</main> 