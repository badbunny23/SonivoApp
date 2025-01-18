<script lang="ts">
    import { onMount } from "svelte";
    import * as Tone from "tone";
    import { page } from '$app/stores';

    import Piano from "$lib/components/Piano.svelte";
    import PianoRoll from "$lib/components/PianoRoll.svelte";
    import Graph from "$lib/components/Graph.svelte";

    import createSampler from "$lib/sampler";
    import {
        generateIntervals,
        Note,
        Measure,
        generateEmptyMeasure,
        getAllNotes,
    } from "$lib/notes";
    import {
        DOMINANT_CADENCE,
        POSSIBLE_PROGRESSIONS,
        MAJOR_TRIADS_MAP,
    } from "$lib/constants";
    import { delay } from "$lib/utils";
    import type { Chord, NoteName } from "$lib/types";

    // Icons
    import IconBack from "~icons/material-symbols/arrow-back-ios";
    import IconForward from "~icons/material-symbols/arrow-forward-ios";
    import IconWholeNote from "~icons/mdi/music-note-whole";
    import IconHalfNote from "~icons/mdi/music-note-half";
    import IconQuarterNote from "~icons/mdi/music-note-quarter";
    import IconEighthNote from "~icons/mdi/music-note-eighth";
    import ChooseKeyToStart from "$lib/components/ChooseKeyToStart.svelte";
    import IconCursorHand from '~icons/mdi/cursor-pointer';
    import IconPlay from "~icons/material-symbols/play-circle";

    // The current measure and chord (noteChoices) both correspond to the same index
    let melody: Measure[] = [];
    let noteChoices: Measure[] = [{ notes: [] }];
    let measureIndex = 0;
    let inMeasureIndex = 0;

    let isFirstKeyChosen = false;

    const playPianoNote = (key: string) => {
        synth.triggerAttack(key);
    };

    const rand = Math.floor(Math.random() * POSSIBLE_PROGRESSIONS.length);

    const chordProgression = [
        ...POSSIBLE_PROGRESSIONS[rand],
        ...DOMINANT_CADENCE,
    ];

    const setKey = (key: Note) => {
        if (key.note === "") return;

        setNoteChoices(key.note, chordProgression);

        melody[measureIndex] = { notes: generateEmptyMeasure(selectedRhythm) };
        isFirstKeyChosen = true;
    };

    const setNoteChoices = (key: NoteName, chordProgression: Chord[]) => {
        let intervals = generateIntervals(key);

        chordProgression.forEach((element, i) => {
            noteChoices[i] = { notes: [] };

            intervals[element].forEach((note: NoteName) => {
                noteChoices[i].notes.push(new Note(note));
            });
        });
    };

    const rhythms = [
        { name: "Whole Note", rhythm: [1] },
        { name: "Half Notes", rhythm: [1 / 2, 1 / 2] },
        { name: "Quarter Notes", rhythm: [1 / 4, 1 / 4, 1 / 4, 1 / 4] },
        {
            name: "Eighth Notes",
            rhythm: [1 / 8, 1 / 8, 1 / 8, 1 / 8, 1 / 8, 1 / 8, 1 / 8, 1 / 8],
        },
    ];

    let selectedRhythm: number[] = rhythms[2].rhythm;

    const setMeasureNote = (note: NoteName, octave: string) => {
        melody[measureIndex].notes[inMeasureIndex].note = note;
        melody[measureIndex].notes[inMeasureIndex].octave = octave;

        // Either increment the current line or go to the next measure or finish
        if (inMeasureIndex + 1 < melody[measureIndex].notes.length) {
            inMeasureIndex++;
        } else if (measureIndex < noteChoices.length - 1) {
            nextMeasure();
        } else {
            finished = true;
        }
    };

    const deleteMeasureNote = (i: number) => {
        melody[measureIndex].notes[i].note = "";
        melody[measureIndex].notes[i].octave = "";
    };

    const nextMeasure = () => {
        if (finished) return;
        setTimeout(() => {

        measureIndex++;

            synth.triggerRelease();
            
            if (!melody[measureIndex]) {
                melody[measureIndex] = { notes: [] };
                melody[measureIndex].notes = generateEmptyMeasure(selectedRhythm);
            }
            
            inMeasureIndex = 0;
        }, 100);
    };

    const previousMeasure = () => {
        if (measureIndex - 1 < 0 || finished) return;

        measureIndex--;
        inMeasureIndex = 0;
    };

    const onChangeRhythm = () => {
        melody[measureIndex].notes = generateEmptyMeasure(selectedRhythm);
        inMeasureIndex = 0;
    };

    let finished = false;

    const measureArrayToNoteArray = (melody: Measure[]) => {
        const notes: Note[] = [];
        for (let i = 0; i < melody.length; i++) {
            for (let j = 0; j < melody[i].notes.length; j++) {
                notes.push(melody[i].notes[j]);
            }
        }
        return notes;
    };

    $: melodyToNotes = measureArrayToNoteArray(melody);

    let hoverNote: string;
    let rollRow = 0;

    let graphRef;

    let isPlaying = false;

    let synth: Tone.Synth<Tone.SynthOptions>;

    let isRecordingHovers = false;
    let recordedNotes: { note: string, duration: number }[] = [];
    let lastHoverTime: number = 0;

    const toggleHoverRecording = () => {
        if (isRecordingHovers) {
            // Stop recording and play the sequence
            playRecordedSequence();
        } else {
            // Start new recording
            recordedNotes = [];
            lastHoverTime = Date.now();
        }
        isRecordingHovers = !isRecordingHovers;
    };

    const addHoveredNote = (note: string) => {
        if (!isRecordingHovers) return;
        
        const currentTime = Date.now();
        const duration = currentTime - lastHoverTime;
        lastHoverTime = currentTime;
        
        recordedNotes.push({
            note,
            duration: Math.max(0.1, duration / 1000) // Convert to seconds, minimum 0.1s
        });
    };

    let currentPlayingNote = -1;

    const playRecordedSequence = async () => {
        if (recordedNotes.length === 0) return;
        
        let now = Tone.now();
        recordedNotes.forEach(({ note, duration }, i) => {
            synth.triggerAttackRelease(note, 0.1, now + i * 0.3);
            // Highlight the current note
            setTimeout(() => {
                currentPlayingNote = i;
                // Reset after the note finishes
                setTimeout(() => {
                    if (i === recordedNotes.length - 1) {
                        currentPlayingNote = -1;
                    }
                }, 300);
            }, i * 300);
        });
    };

    onMount(() => {
        synth = new Tone.Synth().toDestination();

        recorder = new MediaRecorder(dest.stream);

        sampler = createSampler();
        backgroundSynth = createSampler();
        backgroundSynth.volume.value -= 20;

        const keyMap = {
            '1': 'C4',
            '2': 'E4',
            '3': 'G4',
            '4': 'C5',
        };
        
        window.addEventListener('keydown', (e) => {
            const note = keyMap[e.key];
            if (note) {
                synth.triggerAttackRelease(note, '0.1');
                lastPlayedNote = note;
            }
        });
    });

    const onRelease = () => {
        synth.triggerRelease();
    };

    let flagStart = false;
    let startingTimeNow = -1;

    const dest = Tone.getContext().createMediaStreamDestination();

    let recorder: MediaRecorder;
    const audio: Blob[] = [];

    let sampler: Tone.PolySynth<Tone.Synth<Tone.SynthOptions>>;
    let backgroundSynth: Tone.PolySynth<Tone.Synth<Tone.SynthOptions>>;

    let recordedPrev = false;

    const playbackRecord = async () => {
        isPlaying = true;

        let now = Tone.now();

        sampler.connect(dest);

        backgroundSynth.connect(dest);

        recorder.ondataavailable = (event) => {
            audio.push(event.data);
        };

        recorder.onstop = () => {
            const audioBlob = new Blob(audio, {
                type: "audio/ogg; codecs=opus",
            });

            const downloadUrl = URL.createObjectURL(audioBlob);
            const downloadAnchor = document.createElement("a");

            downloadAnchor.style.display = "none";
            downloadAnchor.href = downloadUrl;
            downloadAnchor.download = "myImpromptuPiece.ogg";

            document.body.appendChild(downloadAnchor);
            downloadAnchor.click();
            document.body.removeChild(downloadAnchor);

            URL.revokeObjectURL(downloadUrl);
        };
        if (recordedPrev == false) {
            recorder.start();
            recordedPrev = true;
        }

        if (flagStart == false) {
            startingTimeNow = now;
        }

        for (let i = 0; i < melodyToNotes.length; i++) {
            if (!isPlaying) break;

            let delayTime = 1000 * melodyToNotes[i].length;

            hoverNote = melodyToNotes[i].name();
            rollRow = i;

            if (melodyToNotes[i].note === "") {
                now += melodyToNotes[i].length;
                await delay(delayTime);
                continue;
            }

            let backingTrack;
            let combineTracks;
            let isBacking = false;

            const noteName = melodyToNotes[i].note;

            if (
                (Math.floor(now * 8.0 + 0.001) -
                    Math.floor(startingTimeNow * 8.0 + 0.001)) %
                    1 ==
                    0 &&
                noteName !== ""
            ) {
                backingTrack = MAJOR_TRIADS_MAP[noteName];

                isBacking = true;
                combineTracks = [
                    backingTrack[0],
                    backingTrack[1],
                    backingTrack[2],
                ];
            }

            sampler.triggerAttackRelease(
                melodyToNotes[i].name(),
                melodyToNotes[i].length,
                now,
            );

            if (isBacking == true && combineTracks) {
                backgroundSynth.triggerAttackRelease(combineTracks, 1, now);
            }

            now += melodyToNotes[i].length;
            await delay(delayTime);
        }

        flagStart = false;
        hoverNote = "";
        isPlaying = false;
    };

    let showKeys = false;

    let showHomePage = true;

    $: if ($page.url.searchParams.get('start') === 'true') {
        showHomePage = false;
    }

    let lastPlayedNote = '';

    function getNotePosition(note: string) {
        // Simple mapping of notes to positions (you can expand this)
        const positions = {
            'C4': 80,
            'E4': 60,
            'G4': 40,
            'C5': 20,
            'C#4': 75,
            'F4': 55,
            'G#4': 35,
        };
        return positions[note] || 50;
    }
</script>

<svelte:window on:mouseup={onRelease} />

{#if showHomePage}
    <main class="min-h-screen bg-zinc-900">
        <!-- Navigation Bar -->
        <nav class="fixed w-full top-0 bg-zinc-800 border-b border-white/5 z-50">
            <div class="max-w-7xl mx-auto px-4">
                <div class="flex items-center justify-between h-16">
                    <!-- Logo -->
                    <div class="flex-shrink-0">
                        <span class="text-2xl font-bold bg-gradient-to-r from-white to-white/70 bg-clip-text text-transparent">
                            Sonivo
                        </span>
                    </div>
                    
                    <!-- Navigation Links -->
                    <div class="hidden md:flex items-center gap-8">
                        <a href="/" class="text-white/90 hover:text-white text-sm font-medium transition-colors">
                            Home
                        </a>
                        <a href="/about" class="text-white/70 hover:text-white text-sm font-medium transition-colors">
                            About
                        </a>
                        <button
                            class="bg-white text-zinc-900 rounded-full px-6 py-2 
                                   text-sm font-medium transition-all duration-300
                                   hover:bg-white/90 hover:scale-[1.02]"
                            on:click={() => {
                                showHomePage = false;
                            }}
                        >
                            Start Creating
                        </button>
                    </div>
                </div>
            </div>
        </nav>

        <!-- Hero Section -->
        <section class="relative pt-32 pb-16 px-4 sm:px-6 lg:px-8 max-w-7xl mx-auto">
            <!-- Background gradient effect -->
            <div class="absolute inset-0 -z-10 overflow-hidden">
                <div class="absolute left-[40%] top-0 h-[500px] w-[500px] rounded-full bg-indigo-500/20 blur-[128px]" />
                <div class="absolute right-[40%] bottom-0 h-[500px] w-[500px] rounded-full bg-purple-500/20 blur-[128px]" />
            </div>

            <div class="grid lg:grid-cols-2 gap-12 items-center">
                <div>
                    <h1 class="text-5xl sm:text-7xl font-bold tracking-tight">
                        <span class="bg-gradient-to-r from-white to-white/70 bg-clip-text text-transparent">
                            Create Music
                        </span>
                        <br />
                        <span class="bg-gradient-to-r from-white/70 to-white/50 bg-clip-text text-transparent">
                            With Ease
                        </span>
                    </h1>
                    <p class="mt-8 text-lg text-zinc-400/90 max-w-lg leading-relaxed">
                        Experience a new way of composing music. Our intuitive interface helps you create 
                        beautiful melodies without any musical background.
                    </p>
                    <div class="mt-12 flex gap-6">
                        <button
                            class="bg-gradient-to-r from-white to-white/90 text-zinc-900 rounded-full px-8 py-3.5 
                                   text-sm font-medium transition-all duration-300
                                   hover:shadow-lg hover:shadow-white/20 hover:scale-[1.02]"
                            on:click={() => {
                                showHomePage = false;
                            }}
                        >
                            Start Creating
                        </button>
                        <a 
                            href="/about"
                            class="text-white/90 border border-white/20 rounded-full px-8 py-3.5 
                                   text-sm font-medium transition-all duration-300
                                   hover:bg-white/10 hover:border-white/40 hover:text-white"
                        >
                            Learn More
                        </a>
                    </div>
                </div>
                
                <!-- Visual Element -->
                <div class="relative">
                    <div class="aspect-square rounded-2xl bg-gradient-to-br from-white/10 via-white/5 to-transparent p-8 backdrop-blur-3xl
                               shadow-2xl shadow-black/20 border border-white/10">
                        <div class="w-full h-full rounded-xl bg-gradient-to-br from-white/5 to-transparent backdrop-blur">
                            <!-- Piano keys visual -->
                            <div class="absolute inset-8 flex">
                                {#each Array(7) as _}
                                    <div class="flex-1 border-r border-white/10 transition-all duration-500 hover:bg-white/10">
                                        <div class="h-2/3 bg-white/5"></div>
                                        <div class="h-1/3"></div>
                                    </div>
                                {/each}
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </section>

        <!-- Features Section -->
        <section class="py-16 px-4 sm:px-6 lg:px-8 max-w-7xl mx-auto">
            <div class="grid md:grid-cols-3 gap-0.5 bg-zinc-800 p-0.5 rounded-2xl">
                {#each [
                    { 
                        title: 'Intuitive Interface',
                        desc: 'Experience music creation like never before with our modern tools and features.'
                    },
                    { 
                        title: 'Real-time Playback',
                        desc: 'Hear your composition instantly as you create, with high-quality sound synthesis.'
                    },
                    { 
                        title: 'Export Your Music',
                        desc: 'Save and share your musical creations with others in high-quality audio format.'
                    }
                ] as { title, desc }, i}
                    <div class="group relative first:rounded-l-2xl last:rounded-r-2xl overflow-hidden">
                        <div class="bg-white/5 backdrop-blur-sm p-8 
                                   transition-all duration-300 hover:bg-white/10
                                   flex flex-col h-full">
                            <h3 class="text-lg font-semibold text-white/90 mb-3">{title}</h3>
                        <p class="text-zinc-400">
                                {desc}
                            </p>
                            
                            <!-- Subtle bottom line -->
                            <div class="absolute bottom-0 left-0 right-0 h-[1px] 
                                      bg-gradient-to-r from-transparent via-white/20 to-transparent"></div>
                        </div>
                        
                        <!-- Subtle divider (except last) -->
                        {#if i !== 2}
                            <div class="absolute right-0 top-8 bottom-8 w-[1px]
                                      bg-gradient-to-b from-transparent via-white/10 to-transparent"></div>
                        {/if}
                    </div>
                {/each}
            </div>
        </section>

        <!-- How it Works Section -->
        <section class="py-24 px-4 sm:px-6 lg:px-8 max-w-7xl mx-auto">
            <div class="flex flex-col items-center gap-4 mb-16">
                <h2 class="text-3xl font-bold text-white/90 text-center">How It Works</h2>
                <p class="text-zinc-400 text-center max-w-2xl mb-4">
                    This interactive piano demonstrates how our music composer organizes different notes. While this is just a demonstration, the actual composer uses these principles to intelligently arrange notes in patterns that create pleasing melodies. Try hovering over the keys to see how different note combinations work together!
                </p>
                <div class="flex items-center gap-3 px-4 py-2 bg-white/5 rounded-full border border-white/10 backdrop-blur-sm">
                    <IconCursorHand class="text-white/70 text-xl animate-bounce" />
                    <span class="text-sm text-zinc-400">
                        Interactive demo - Hover to play notes
                    </span>
                    <div class="h-4 w-[1px] bg-zinc-600"></div>
                    <button
                        class="flex items-center gap-2 px-3 py-1 rounded-full transition-colors
                               {isRecordingHovers ? 'bg-red-500/20 text-red-400' : 'bg-white/5 text-zinc-400'}
                               hover:bg-white/10"
                        on:click={toggleHoverRecording}
                    >
                        <div class="w-2 h-2 rounded-full {isRecordingHovers ? 'bg-red-500 animate-pulse' : 'bg-zinc-400'}" />
                        {isRecordingHovers ? 'Recording...' : 'Record Sequence'}
                    </button>
                </div>
            </div>
            
            <!-- Piano-styled steps container -->
            <div class="relative mx-auto max-w-6xl">
                <!-- Piano top panel -->
                <div class="h-8 bg-zinc-700 rounded-t-xl border-b border-zinc-600 
                            flex items-center justify-center">
                    <div class="w-32 h-2 bg-zinc-500 rounded-full"></div>
                </div>
                
                <!-- Piano body -->
                <div class="relative bg-zinc-800 p-4 rounded-b-xl border-x border-b border-zinc-700
                            shadow-2xl">
                    <!-- Keys container with subtle gradient -->
                    <div class="relative bg-gradient-to-b from-zinc-700 to-zinc-800 p-1 rounded-lg">
                        <!-- White keys -->
                        <div class="grid md:grid-cols-4 gap-1 relative">
                            {#each [
                                { step: 1, title: 'Choose Your Key', desc: 'Select your starting note to begin your musical journey', note: 'C4' },
                                { step: 2, title: 'Select Rhythm', desc: 'Pick from various note durations to create your perfect rhythm', note: 'E4' },
                                { step: 3, title: 'Compose', desc: 'Use our intuitive interface to create your melody', note: 'G4' },
                                { step: 4, title: 'Export', desc: 'Download and share your musical creation', note: 'C5' }
                            ] as { step, title, desc, note }, i}
                                <div class="group relative"
                                     on:mouseenter={() => {
                                         synth.triggerAttackRelease(note, '0.1');
                                         lastPlayedNote = note;
                                         addHoveredNote(note);
                                     }}>
                                    <!-- White key -->
                                    <div class="bg-gradient-to-b from-white to-white/95 backdrop-blur-md rounded-lg p-8
                                              transition-all duration-300 
                                              flex flex-col items-center text-center gap-4
                                              min-h-[280px] relative z-0
                                              shadow-sm group-hover:shadow-xl
                                              transform group-hover:translate-y-1
                                              {i === 0 ? 'group-hover:bg-blue-50 group-hover:from-blue-50 group-hover:to-blue-50/95' : 
                                               i === 1 ? 'group-hover:bg-emerald-50 group-hover:from-emerald-50 group-hover:to-emerald-50/95' :
                                               i === 2 ? 'group-hover:bg-violet-50 group-hover:from-violet-50 group-hover:to-violet-50/95' :
                                               'group-hover:bg-amber-50 group-hover:from-amber-50 group-hover:to-amber-50/95'}">
                                        <!-- Step number -->
                                        <div class="w-14 h-14 rounded-full bg-zinc-900 
                                                  flex items-center justify-center mb-4
                                                  shadow-inner border border-zinc-800">
                                            <span class="text-2xl font-bold text-white">{step}</span>
                                        </div>
                                        
                                        <!-- Content -->
                                        <h3 class="text-lg font-semibold text-zinc-900 mb-2
                                                  transition-colors duration-300
                                                  {i === 0 ? 'group-hover:text-blue-600' : 
                                                   i === 1 ? 'group-hover:text-emerald-600' :
                                                   i === 2 ? 'group-hover:text-violet-600' :
                                                   'group-hover:text-amber-600'}">{title}</h3>
                                        <p class="text-zinc-600 leading-relaxed">{desc}</p>
                                        
                                        <!-- Bottom gradient -->
                                        <div class="absolute bottom-0 inset-x-0 h-12 bg-gradient-to-t 
                                                  from-black/5 to-transparent rounded-b-lg
                                                  opacity-0 group-hover:opacity-100
                                                  transition-all duration-300"></div>
                    </div>
                                    
                                    <!-- Black key overlay -->
                                    {#if i !== 3}
                                        <div class="absolute -right-6 top-0 w-12 h-[65%] 
                                                  bg-gradient-to-b from-zinc-900 to-zinc-800
                                                  rounded-b-xl z-10 hidden md:block
                                                  shadow-lg border-x border-b border-zinc-800
                                                  group-hover:from-zinc-800 group-hover:to-zinc-900
                                                  transition-colors duration-300"
                                             on:mouseenter|stopPropagation={() => {
                                                 const blackNotes = ['C#4', 'F4', 'G#4'];
                                                 synth.triggerAttackRelease(blackNotes[i], '0.1');
                                                 lastPlayedNote = blackNotes[i];
                                             }}>
                                            <!-- Black key shine effect -->
                                            <div class="absolute inset-x-0 top-0 h-8 bg-gradient-to-b 
                                                      from-zinc-800 to-transparent rounded-t-sm"></div>
                </div>
                                    {/if}
                    </div>
                            {/each}
                </div>
                    </div>
                </div>
                
                <!-- Piano bottom shadow -->
                <div class="absolute -bottom-8 inset-x-0 h-8 bg-gradient-to-t from-zinc-900 to-transparent"></div>
                
                <!-- Note display -->
                <div class="mt-12 text-center">
                    {#key lastPlayedNote}
                        {#if lastPlayedNote}
                            <div class="inline-block px-6 py-2 bg-zinc-800/80 backdrop-blur-sm rounded-full 
                                        border border-zinc-700/50 shadow-lg
                                        transition-all duration-300 animate-fade-in">
                                <span class="text-lg font-mono text-white/90">
                                    {lastPlayedNote}
                                </span>
                    </div>
                        {/if}
                    {/key}
                </div>

                <!-- Developer Credits - Moved here -->
               
            </div>
        </section>
    </main>
{:else if !isFirstKeyChosen}
    <ChooseKeyToStart
        on:initialNoteChosen={(e) => {
            synth.triggerAttack(e.detail.name());

            setTimeout(() => {
                synth.triggerRelease();
            }, 200);

            setKey(e.detail);
        }}
    />
{:else}
    <div class="min-h-screen max-w-screen-xl mx-auto flex flex-col">
        <header class="flex gap-8 justify-end w-full py-8">
            <div class="flex items-center text-white gap-2">
                <span
                    class="bg-gradient-to-r from-zinc-300 via-zinc-400 to-zinc-300 bg-clip-text text-transparent inline-block"
                >
                    Show Keys
                </span>
                <label class="inline-flex items-center cursor-pointer">
                    <input
                        type="checkbox"
                        class="sr-only peer"
                        bind:checked={showKeys}
                    />
                    <div
                        class="relative w-12 h-6 bg-zinc-600 rounded-full
                                peer peer-checked:after:translate-x-full
                                rtl:peer-checked:after:-translate-x-full
                                peer-checked:after:border-white after:content-['']
                                after:absolute after:top-[2px] after:start-[2px]
                                after:bg-white after:border-zinc-600 after:border
                                after:rounded-full after:h-5
                                after:w-5 after:transition-all
                                peer-checked:bg-zinc-700"
                    ></div>
                </label>
            </div>
            <div>
                <button
                    class="disabled:cursor-not-allowed bg-transparent text-[#f8f8ff] border-2 border-[#f8f8ff] rounded px-4 py-2 cursor-pointer text-lg transition-colors hover:bg-[#f8f8ff] hover:text-black
                        {!isPlaying && finished
                        ? 'border-green-400 text-green-300 hover:bg-green-100 hover:text-black'
                        : finished
                          ? 'border-red-400 text-red-300 hover:bg-red-100 hover:text-black'
                          : ''}"
                    disabled={isPlaying}
                    on:click={() => {
                        if (!finished) {
                            finished = true;
                        } else {
                            playbackRecord();
                            setTimeout(
                                () => {
                                    recorder.stop();
                                    sampler.disconnect(dest);
                                    backgroundSynth.disconnect(dest);
                                },
                                1000 * (chordProgression.length + 2) + 200,
                            );
                        }
                    }}
                >
                    {!finished ? "Finish" : "Play"}
                </button>
            </div>
        </header>

        <main class="flex gap-8 items-center h-full grow">
            <div class="grow flex justify-center items-center flex-col pb-10">
                <div class="flex">
                    <div>
                        <button
                            disabled={finished}
                            on:click={previousMeasure}
                            class="text-zinc-400 p-4 {!finished && 'hover:text-zinc-300'}"
                        >
                            <IconBack />
                        </button>
                    </div>
                    <div>
                        <!-- PIANO -->
                        <div class="bg-zinc-800 h-fit w-fit p-6 rounded-3xl shadow-xl border border-zinc-700">
                            <!-- ROLL -->
                            {#if !finished}
                                <PianoRoll
                                    notes={melody[measureIndex].notes}
                                    bind:selectedIndex={inMeasureIndex}
                                    deleteNote={deleteMeasureNote}
                                />
                            {:else}
                                <PianoRoll
                                    notes={melodyToNotes}
                                    selectedIndex={rollRow}
                                    canEdit={false}
                                    deleteNote={deleteMeasureNote}
                                />
                            {/if}

                            <!-- KEYS -->
                            <Piano
                                {hoverNote}
                                playNote={(key) => {
                                    playPianoNote(key.name());
                                    if (finished || key.note === "") return;
                                    setMeasureNote(key.note, key.octave);
                                }}
                                selectableNotes={!finished ? noteChoices[measureIndex].notes : getAllNotes()}
                                {showKeys}
                            />
                        </div>
                    </div>
                    <div>
                        <button
                            disabled={finished}
                            on:click={() => {
                                if (measureIndex < noteChoices.length - 1) {
                                    nextMeasure();
                                }
                            }}
                            class="text-zinc-400 p-4 {!finished && 'hover:text-zinc-300'}"
                        >
                            <IconForward />
                        </button>
                    </div>
                </div>
                <!-- RHYTHM BUTTONS -->
                <div
                    class="mt-8 flex flex-row items-center justify-center gap-2"
                >
                    {#each rhythms as { name, rhythm }}
                        <button
                            on:click={() => {
                                selectedRhythm = rhythm;
                                onChangeRhythm();
                            }}
                            class="flex justify-center items-center p-4 rounded-lg border-2 hover:border-white border-transparent {finished &&
                                'invisible'}"
                        >
                            {#if name === "Whole Note"}
                                <IconWholeNote class="text-zinc-400 text-xl" />
                            {:else if name === "Half Notes"}
                                <IconHalfNote class="text-zinc-400 text-xl" />
                            {:else if name === "Quarter Notes"}
                                <IconQuarterNote
                                    class="text-zinc-400 text-xl"
                                />
                            {:else if name === "Eighth Notes"}
                                <IconEighthNote class="text-zinc-400 text-xl" />
                            {/if}
                            <span
                                class="bg-gradient-to-r from-zinc-400 via-zinc-500 to-zinc-400 bg-clip-text text-transparent inline-block"
                            >
                                {name}
                            </span>
                        </button>
                    {/each}
                </div>
            </div>
            <div class="max-h-full box-border grow pb-20">
                <Graph bind:this={graphRef} notes={melodyToNotes} />
            </div>
        </main>
    </div>
{/if}

{#if recordedNotes.length > 0 && !isRecordingHovers}
    <!-- Timeline visualization -->
    <div class="w-full bg-gradient-to-t from-black/80 to-transparent py-4 -mt-12">
        <div class="max-w-2xl mx-auto px-4">
            <div class="bg-zinc-800/90 backdrop-blur-sm rounded-2xl border border-white/10 p-4">
                <!-- Notes timeline -->
                <div class="relative h-12">
                    <div class="absolute inset-0 flex items-center">
                        {#each recordedNotes as note, i}
                            <div 
                                class="flex-1 px-0.5 group"
                                style="transition-delay: {i * 50}ms"
                            >
                                <div class="h-8 relative">
                                    <!-- Note marker -->
                                    <div class="absolute inset-x-0 top-1/2 -translate-y-1/2 
                                                flex flex-col items-center gap-1">
                                        <div class="w-2 h-2 rounded-full transition-all duration-300
                                                    {i === currentPlayingNote ? 'bg-white scale-125' : 'bg-white/40'} 
                                                    group-hover:bg-white/80"></div>
                                        <!-- Note label -->
                                        <div class="opacity-0 group-hover:opacity-100 
                                                    transition-opacity text-xs text-white/70">
                                            {note.note}
                                        </div>
                                    </div>
                                </div>
                            </div>
                        {/each}
                    </div>
                </div>
                
                <!-- Timeline info -->
                <div class="flex justify-between items-center mt-2 text-sm text-zinc-400">
                    <span>{recordedNotes.length} notes recorded</span>
                    <button
                        class="flex items-center gap-2 px-3 py-1.5 rounded-full 
                               bg-white/5 hover:bg-white/10 transition-colors"
                        on:click={playRecordedSequence}
                    >
                        <IconPlay class="text-lg" />
                        <span>Play Sequence</span>
                    </button>
                </div>
            </div>
        </div>
    </div>
{/if}

<style>
    .fade-out {
        animation: fadeOutAnimation 1s ease forwards;
    }

    @keyframes fadeOutAnimation {
        0% {
            opacity: 1;
        }
        100% {
            opacity: 0;
        }
    }

    @keyframes float {
        0%, 100% { 
            transform: translateY(0) rotate(0deg) scale(1); 
            opacity: 0.2;
        }
        50% { 
            transform: translateY(-20px) rotate(10deg) scale(1.1);
            opacity: 0.3;
        }
    }

    @keyframes fade-in {
        from { opacity: 0; transform: translateY(-10px); }
        to { opacity: 1; transform: translateY(0); }
    }
    
    .animate-fade-in {
        animation: fade-in 0.2s ease-out forwards;
    }
</style>
