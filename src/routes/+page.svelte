<script lang="ts">
    import { onMount } from "svelte";

    const CYCLE_LENGTH = 28;
    const PERIOD_WINDOW = 4;
    const OVULATION_DAY = 14;
    const OVULATION_WINDOW = 3;

    type PeriodDay = {
        date: Date;
        status: "none" | "upcoming" | "active" | "completed" | "ovulation";
        month: "prev" | "current" | "next";
    };

    let days: PeriodDay[] = [];
    let periodStart: string | null = null;
    let daysPassed = 0;
    let showModal = false;
    let showOverlay = true;

    function generateCalendar() {
        days = [];
        const today = new Date();
        const currentMonth = today.getMonth();
        const year = today.getFullYear();

        // === Full previous month ===
        const prevMonth = currentMonth === 0 ? 11 : currentMonth - 1;
        const prevYear = currentMonth === 0 ? year - 1 : year;
        const prevMonthLastDay = new Date(prevYear, prevMonth + 1, 0).getDate();

        for (let d = 1; d <= prevMonthLastDay; d++) {
            days.push({
                date: new Date(prevYear, prevMonth, d),
                status: "none",
                month: "prev",
            });
        }

        // === Full current month ===
        const lastDay = new Date(year, currentMonth + 1, 0).getDate();
        for (let d = 1; d <= lastDay; d++) {
            days.push({
                date: new Date(year, currentMonth, d),
                status: "none",
                month: "current",
            });
        }

        // === Two future months ===
        for (let monthOffset = 1; monthOffset <= 3; monthOffset++) {
            const futureMonth = (currentMonth + monthOffset) % 12;
            const futureYear =
                currentMonth + monthOffset > 11 ? year + 1 : year;
            const futureLastDay = new Date(
                futureYear,
                futureMonth + 1,
                0,
            ).getDate();

            for (let d = 1; d <= futureLastDay; d++) {
                days.push({
                    date: new Date(futureYear, futureMonth, d),
                    status: "none",
                    month: "next",
                });
            }
        }
    }

    function calcDaysPassed() {
        if (!periodStart) {
            daysPassed = 0;
            return;
        }
        const start = new Date(periodStart);
        const today = new Date();
        today.setHours(0, 0, 0, 0);
        start.setHours(0, 0, 0, 0);

        const diff = Math.floor(
            (today.getTime() - start.getTime()) / (1000 * 60 * 60 * 24),
        );
        daysPassed = diff % CYCLE_LENGTH;
    }

    function markPeriod() {
        if (!periodStart) return;

        days = days.map((day) => ({ ...day, status: "none" }));

        const start = new Date(periodStart);
        const today = new Date();
        today.setHours(0, 0, 0, 0);

        // Mark all cycles (past and future)
        const firstDay = new Date(days[0].date);
        const lastDay = new Date(days[days.length - 1].date);

        // Calculate how many cycles to check
        const totalDays = Math.ceil(
            (lastDay.getTime() - start.getTime()) / (1000 * 60 * 60 * 24),
        );
        const numCycles = Math.ceil(totalDays / CYCLE_LENGTH) + 1;

        for (let cycle = 0; cycle < numCycles; cycle++) {
            const cycleStart = new Date(start);
            cycleStart.setDate(start.getDate() + cycle * CYCLE_LENGTH);

            // Mark period days for this cycle
            for (let i = 0; i < PERIOD_WINDOW; i++) {
                const d = new Date(cycleStart);
                d.setDate(cycleStart.getDate() + i);
                d.setHours(0, 0, 0, 0);

                const isPast = d.getTime() < today.getTime();
                const isToday = d.getTime() === today.getTime();

                days = days.map((day) => {
                    const dayDate = new Date(day.date);
                    dayDate.setHours(0, 0, 0, 0);
                    if (dayDate.getTime() === d.getTime()) {
                        if (isPast) {
                            return { ...day, status: "completed" };
                        } else if (isToday) {
                            return { ...day, status: "active" };
                        } else {
                            return { ...day, status: "upcoming" };
                        }
                    }
                    return day;
                });
            }

            // Mark ovulation days for this cycle
            for (
                let i = OVULATION_DAY - 1;
                i < OVULATION_DAY + OVULATION_WINDOW - 1;
                i++
            ) {
                const d = new Date(cycleStart);
                d.setDate(cycleStart.getDate() + i);
                d.setHours(0, 0, 0, 0);

                days = days.map((day) => {
                    const dayDate = new Date(day.date);
                    dayDate.setHours(0, 0, 0, 0);
                    if (
                        dayDate.getTime() === d.getTime() &&
                        day.status === "none"
                    ) {
                        return { ...day, status: "ovulation" };
                    }
                    return day;
                });
            }
        }
    }

    function selectDate(date: Date) {
        periodStart = date.toISOString();
        localStorage.setItem("period", periodStart);
        calcDaysPassed();
        markPeriod();
        showModal = false;
    }

    function closeOverlay() {
        showOverlay = false;
    }

    onMount(() => {
        generateCalendar();
        const stored = localStorage.getItem("period");

        if (stored) {
            periodStart = stored;
            calcDaysPassed();
            markPeriod();
        }
    });
</script>

<!-- === Overlay with blur + dots === -->
{#if showOverlay}
    <div
        class="fixed inset-0 flex items-center justify-center backdrop-blur-sm z-20"
    >
        <!-- Close button -->
        <!-- svelte-ignore a11y_consider_explicit_label -->
        <button
            class="absolute top-20 right-4 w-10 h-10 rounded-full bg-white shadow-lg flex items-center justify-center hover:bg-gray-100 z-30"
            onclick={closeOverlay}
        >
            <svg
                xmlns="http://www.w3.org/2000/svg"
                class="h-6 w-6 text-gray-600"
                fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor"
            >
                <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M6 18L18 6M6 6l12 12"
                />
            </svg>
        </button>

        <div class="relative w-72 h-72 flex items-center justify-center">
            {#each Array(CYCLE_LENGTH) as _, i}
                {@const isCurrentDay = periodStart && i === daysPassed}
                {@const isPeriod = i < PERIOD_WINDOW}
                {@const isOvulation =
                    i >= OVULATION_DAY - 1 &&
                    i < OVULATION_DAY + OVULATION_WINDOW - 1}

                <div
                    class="absolute"
                    style="transform: rotate({(360 / CYCLE_LENGTH) *
                        i}deg) translateY(-7.5rem)"
                >
                    {#if isCurrentDay}
                        <div class="relative">
                            <div
                                class={`w-2 h-2 rounded-full ${periodStart ? (isPeriod ? "bg-red-500" : isOvulation ? "bg-purple-500" : "bg-gray-400") : "bg-gray-400"}`}
                            ></div>
                            <div
                                class="absolute -top-4 left-1/2 -translate-x-1/2"
                            >
                                <div
                                    class="w-0 h-0 border-l-4 border-r-4 border-b-8 border-transparent border-b-gray-800"
                                ></div>
                            </div>
                        </div>
                    {:else}
                        <div
                            class={`w-2 h-2 rounded-full ${periodStart ? (isPeriod ? "bg-red-500" : isOvulation ? "bg-purple-500" : "bg-gray-400") : "bg-gray-400"}`}
                        ></div>
                    {/if}
                </div>
            {/each}

            <!-- Center circle -->
            <div
                class="w-52 h-52 rounded-full border-2 border-gray-400 flex flex-col space-y-2 items-center justify-center bg-white shadow"
            >
                <p class="text-base text-gray-500">Edit / Log</p>
                <p class="text-base font-bold text-red-500">Your Date</p>
                <button
                    class="mt-2 text-base px-2.5 py-2 bg-red-500 text-white rounded-full w-32"
                    onclick={() => (showModal = true)}
                >
                    Edit
                </button>
                {#if periodStart}
                    <p class="text-sm text-gray-600">
                        {daysPassed}
                        {daysPassed === 1 ? "day" : "days"} since last period
                    </p>
                {:else}
                    <p class="text-sm text-gray-600">No period logged yet</p>
                {/if}
            </div>
        </div>

        <div class="absolute bottom-4">
            <p class="text-center text-black font-funky">
                Made In Love ❤️ For My Queen Chismdi Blessing (Babe M 😘)
            </p>
        </div>
    </div>
{/if}

<!-- === Edit Modal === -->
{#if showModal}
    <div
        class="fixed inset-0 bg-black bg-opacity-40 flex items-center justify-center z-30"
    >
        <div
            class="bg-white border p-6 rounded-xl w-96 max-h-96 overflow-y-auto"
        >
            <h2 class="text-lg font-bold mb-4 text-center">
                When did you see your last period?
            </h2>
            <div class="grid grid-cols-7 gap-2">
                {#each days.filter((d) => d.month === "prev") as day}
                    <button
                        class="w-10 h-10 rounded-full hover:bg-red-100 flex items-center justify-center"
                        onclick={() => selectDate(day.date)}
                    >
                        {day.date.getDate()}
                    </button>
                {/each}
            </div>
            <div class="flex justify-end mt-4">
                <button
                    class="px-4 py-2 bg-gray-200 rounded-full hover:bg-gray-300"
                    onclick={() => (showModal = false)}>Close</button
                >
            </div>
        </div>
    </div>
{/if}

<!-- === Calendar === -->
<div class="relative z-10 my-20">
    <div class="flex justify-center min-h-screen mb-14">
        <div class="flex flex-col gap-8 p-4 lg:hidden">
            {#each Array.from(days.reduce((acc, day) => {
                    const key = `${day.date.getFullYear()}-${day.date.getMonth()}`;
                    if (!acc.has(key)) acc.set(key, []);
                    acc.get(key).push(day);
                    return acc;
                }, new Map())) as [key, monthDays]}
                {@const firstDay = monthDays[0].date}
                <div class="flex flex-col gap-3">
                    <!-- 🗓 Month Name -->
                    <h2
                        class="text-center text-lg font-bold text-gray-700 font-inter mb-4"
                    >
                        {firstDay.toLocaleString("default", {
                            month: "long",
                            year: "numeric",
                        })}
                    </h2>

                    <!-- Month Days Grid -->
                    <div class="grid grid-cols-7 gap-3">
                        {#each monthDays as day}
                            {#if day.status === "upcoming"}
                                <div
                                    class="w-10 h-10 rounded-full border-2 border-dashed border-red-500 flex items-center justify-center font-inter text-gray-700"
                                >
                                    {day.date.getDate()}
                                </div>
                            {:else if day.status === "completed"}
                                <div
                                    class="flex items-center justify-center w-10 h-10 rounded-full relative font-inter bg-gradient-to-br from-red-300 to-red-600"
                                >
                                    <svg
                                        xmlns="http://www.w3.org/2000/svg"
                                        class="h-5 w-5 text-white absolute"
                                        fill="none"
                                        viewBox="0 0 24 24"
                                        stroke="currentColor"
                                    >
                                        <path
                                            stroke-linecap="round"
                                            stroke-linejoin="round"
                                            stroke-width="3"
                                            d="M5 13l4 4L19 7"
                                        />
                                    </svg>
                                    <span
                                        class="text-xs text-white font-bold mt-6"
                                    >
                                        {day.date.getDate()}
                                    </span>
                                </div>
                            {:else if day.status === "active"}
                                <div
                                    class="w-10 h-10 rounded-full p-[2px] bg-gradient-to-br from-pink-300 to-pink-400"
                                >
                                    <div
                                        class="flex items-center justify-center w-full h-full rounded-full bg-pink-100 border-2 border-red-500 font-inter text-red-600 font-bold"
                                    >
                                        {day.date.getDate()}
                                    </div>
                                </div>
                            {:else if day.status === "ovulation"}
                                <div
                                    class="flex items-center justify-center w-10 h-10 rounded-full font-inter bg-purple-500 text-white"
                                >
                                    {day.date.getDate()}
                                </div>
                            {:else}
                                <div
                                    class="flex items-center justify-center w-10 h-10 rounded-full relative font-inter"
                                >
                                    {day.date.getDate()}
                                </div>
                            {/if}
                        {/each}
                    </div>
                    <div class="border-b-2 border-gray-300 my-2"></div>
                </div>
            {/each}
        </div>
    </div>
</div>
