<script lang="ts">
    import { createEventDispatcher } from "svelte";
    import type { Note } from "$lib/notes";

    import Piano from "./Piano.svelte";

    const dispatch = createEventDispatcher<{
        initialNoteChosen: Note;
    }>();

    let hoverNote = "";
</script>

<section class="h-screen w-screen flex flex-col items-center justify-center">
    <h1 class="text-5xl font-bold bg-gradient-to-r from-white to-white/70 bg-clip-text text-transparent mb-12">
        Choose a key to start
    </h1>
    <div class="p-10 flex items-center justify-center">
        <div
            class="bg-zinc-900/80 border-[6px] p-10 pr-20 flex items-center justify-center rounded-xl
                   shadow-2xl backdrop-blur-sm relative"
            style="border-image: linear-gradient(45deg, #ffffff, #a1a1aa, #27272a, #a1a1aa, #ffffff) 1;"
        >
            <div class="absolute inset-0 bg-white/5 blur-3xl rounded-full -z-10"></div>
            <Piano
                {hoverNote}
                playNote={(key) => {
                    dispatch("initialNoteChosen", key);
                }}
            />
        </div>
    </div>
    <p class="mt-8 text-zinc-400 text-lg">
        Click any key to begin
    </p>
</section>

<style>
    /* Add subtle floating animation to the piano container */
    div {
        animation: float 6s ease-in-out infinite;
    }

    @keyframes float {
        0%, 100% { transform: translateY(0); }
        50% { transform: translateY(-10px); }
    }
</style>
