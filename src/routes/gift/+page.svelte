<script lang="ts">
    let isOpen = false;
    let typedText = "";

    const message =
        "You are my everything, Chismdi Blessing. Every moment with you is a gift. I love you more than words can say! 💕";
    const emojis = [
        "🌸",
        "💖",
        "🌺",
        "💝",
        "🌷",
        "💗",
        "🌹",
        "💕",
        "✨",
        "🎀",
        "💐",
        "🦋",
    ];

    const leaves = Array.from({ length: 80 }, (_, i) => ({
        id: i,
        left: Math.random() * 100,
        delay: Math.random() * 3,
        duration: 4 + Math.random() * 3,
        emoji: emojis[Math.floor(Math.random() * emojis.length)],
    }));

    function handleGiftClick() {
        isOpen = true;

        // Typing animation
        let index = 0;
        const interval = setInterval(() => {
            if (index < message.length) {
                typedText = message.substring(0, index + 1);
                index++;
            } else {
                clearInterval(interval);
            }
        }, 50);
    }
</script>

<div
    class="min-h-screen bg-gradient-to-br from-pink-100 via-purple-100 to-red-100 flex items-center justify-center overflow-hidden relative"
>
    <!-- Falling celebration leaves -->
    {#if isOpen}
        <div class="absolute inset-0 pointer-events-none z-50">
            {#each leaves as leaf}
                <div
                    class="absolute text-5xl animate-fall"
                    style="left: {leaf.left}%; top: -100px; animation-delay: {leaf.delay}s; animation-duration: {leaf.duration}s;"
                >
                    {leaf.emoji}
                </div>
            {/each}
        </div>
    {/if}

    <!-- Main content -->
    <div class="text-center z-10 px-4 w-full">
        {#if !isOpen}
            <!-- Big centered gift box -->
            <div class="flex flex-col items-center justify-center">
                <button
                    on:click={handleGiftClick}
                    class="gift-shake cursor-pointer transform transition-all duration-300 hover:scale-110"
                >
                    <div class="relative w-64 h-64 md:w-80 md:h-80">
                        <!-- Gift box -->
                        <div
                            class="w-full h-full bg-gradient-to-br from-red-400 via-red-500 to-red-600 rounded-3xl shadow-2xl relative"
                        >
                            <!-- Ribbon vertical -->
                            <div
                                class="absolute top-0 left-1/2 transform -translate-x-1/2 w-16 h-full bg-gradient-to-b from-yellow-300 via-yellow-400 to-yellow-500 shadow-lg"
                            ></div>
                            <!-- Ribbon horizontal -->
                            <div
                                class="absolute top-1/2 left-0 transform -translate-y-1/2 w-full h-16 bg-gradient-to-r from-yellow-300 via-yellow-400 to-yellow-500 shadow-lg"
                            ></div>
                            <!-- Bow -->
                            <div
                                class="absolute -top-12 left-1/2 transform -translate-x-1/2 text-9xl drop-shadow-2xl"
                            >
                                🎀
                            </div>

                            <!-- Sparkles around box -->
                            <div
                                class="absolute -top-4 -right-4 w-8 h-8 bg-yellow-300 rounded-full animate-ping"
                            ></div>
                            <div
                                class="absolute -bottom-4 -left-4 w-6 h-6 bg-pink-400 rounded-full animate-ping"
                                style="animation-delay: 0.5s"
                            ></div>
                            <div
                                class="absolute top-1/4 -left-6 w-5 h-5 bg-purple-300 rounded-full animate-ping"
                                style="animation-delay: 1s"
                            ></div>
                            <div
                                class="absolute bottom-1/4 -right-6 w-7 h-7 bg-red-300 rounded-full animate-ping"
                                style="animation-delay: 1.5s"
                            ></div>
                        </div>
                    </div>
                </button>

                <p
                    class="mt-10 text-3xl md:text-4xl font-bold bg-gradient-to-r from-pink-500 via-purple-500 to-red-500 bg-clip-text text-transparent animate-bounce font-funky"
                >
                    Click to Open Your message!
                </p>
            </div>
        {:else}
            <!-- Message after opening -->
            <div class="animate-fadeIn max-w-3xl mx-auto">
                <div
                    class="bg-white/90 backdrop-blur-md rounded-3xl p-8 md:p-12 shadow-2xl"
                >
                    <h1
                        class="text-4xl md:text-5xl font-bold bg-gradient-to-r from-pink-500 via-purple-500 to-red-500 bg-clip-text text-transparent mb-8 font-funky"
                    >
                        For You, My Love 💖
                    </h1>
                    <p
                        class="text-2xl md:text-3xl font-funky text-slate-700 leading-relaxed min-h-[250px] font-medium"
                    >
                        {typedText}<span class="animate-pulse text-purple-500"
                            >|</span
                        >
                    </p>
                </div>
            </div>
        {/if}
    </div>
</div>

<style>
    @keyframes fall {
        0% {
            transform: translateY(-100px) rotate(0deg);
            opacity: 1;
        }
        100% {
            transform: translateY(100vh) rotate(720deg);
            opacity: 0;
        }
    }

    @keyframes fadeIn {
        from {
            opacity: 0;
            transform: scale(0.5);
        }
        to {
            opacity: 1;
            transform: scale(1);
        }
    }

    @keyframes shake {
        0%,
        100% {
            transform: rotate(0deg);
        }
        25% {
            transform: rotate(-5deg);
        }
        75% {
            transform: rotate(5deg);
        }
    }

    .animate-fall {
        animation: fall linear forwards;
    }

    .animate-fadeIn {
        animation: fadeIn 0.6s ease-out;
    }

    .gift-shake:hover {
        animation: shake 0.5s ease-in-out infinite;
    }
</style>
