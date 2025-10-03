<script lang="ts">
    import { onMount } from "svelte";

    const CYCLE_LENGTH = 28;
    const PERIOD_WINDOW = 4;
    const OVULATION_DAY = 14; // Typically day 14 of cycle
    const OVULATION_WINDOW = 3; // 3-day ovulation window

    type PeriodDay = {
        date: string;
        status: "none" | "upcoming" | "active" | "completed" | "ovulation";
        month: "prev" | "current";
    };

    let days: PeriodDay[] = [];
    let periodStart: string | null = null;
    let daysPassed = 0;
    let showModal = false;
    let showOverlay = true;

    function generateCalendar() {
        const today = new Date();
        const currentMonth = today.getMonth();
        const year = today.getFullYear();

        // === Full previous month ===
        const prevMonth = currentMonth === 0 ? 11 : currentMonth - 1;
        const prevYear = currentMonth === 0 ? year - 1 : year;
        const prevMonthLastDay = new Date(prevYear, prevMonth + 1, 0).getDate();

        for (let d = 1; d <= prevMonthLastDay; d++) {
            days.push({
                date: `${prevYear}-${prevMonth + 1}-${d}`,
                status: "none",
                month: "prev",
            });
        }

        // === Full current month ===
        const lastDay = new Date(year, currentMonth + 1, 0).getDate();
        for (let d = 1; d <= lastDay; d++) {
            days.push({
                date: `${year}-${currentMonth + 1}-${d}`,
                status: "none",
                month: "current",
            });
        }
    }

    function calcDaysPassed() {
        if (!periodStart) {
            daysPassed = 0;
            return;
        }
        const start = new Date(periodStart);
        const today = new Date();
        const diff = Math.floor(
            (today.getTime() - start.getTime()) / (1000 * 60 * 60 * 24),
        );
        daysPassed = diff % CYCLE_LENGTH;
    }

    function markPeriod() {
        if (!periodStart) return;

        days = days.map((day) => ({ ...day, status: "none" }));

        const start = new Date(periodStart);

        // Completed cycle (past 4 days)
        for (let i = 0; i < PERIOD_WINDOW; i++) {
            const d = new Date(start);
            d.setDate(start.getDate() + i);
            const key = `${d.getFullYear()}-${d.getMonth() + 1}-${d.getDate()}`;
            days = days.map((day) =>
                day.date === key ? { ...day, status: "completed" } : day,
            );
        }

        // Mark ovulation window (days 12-14, center around day 14)
        for (
            let i = OVULATION_DAY - 1;
            i < OVULATION_DAY + OVULATION_WINDOW - 1;
            i++
        ) {
            const d = new Date(start);
            d.setDate(start.getDate() + i);
            const key = `${d.getFullYear()}-${d.getMonth() + 1}-${d.getDate()}`;
            days = days.map((day) =>
                day.date === key ? { ...day, status: "ovulation" } : day,
            );
        }

        // Predict next cycle (assume 28-day cycle)
        const next = new Date(start);
        next.setDate(start.getDate() + CYCLE_LENGTH);
        for (let i = 0; i < PERIOD_WINDOW; i++) {
            const d = new Date(next);
            d.setDate(next.getDate() + i);
            const key = `${d.getFullYear()}-${d.getMonth() + 1}-${d.getDate()}`;
            days = days.map((day) =>
                day.date === key ? { ...day, status: "upcoming" } : day,
            );
        }
    }

    function selectDate(date: string) {
        periodStart = date;
        localStorage.setItem("periodStart", periodStart);
        calcDaysPassed();
        markPeriod();
        showModal = false;
        // Don't hide overlay, only close button can do that
    }

    function closeOverlay() {
        showOverlay = false;
    }

    onMount(() => {
        generateCalendar();
        const stored = localStorage.getItem("periodStart");

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
            on:click={closeOverlay}
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
                <div
                    class={`absolute w-2 h-2 rounded-full
                    ${
                        periodStart
                            ? i < daysPassed
                                ? "bg-blue-500"
                                : i >= 0 && i < PERIOD_WINDOW
                                  ? "bg-red-500"
                                  : i >= OVULATION_DAY - 1 &&
                                      i < OVULATION_DAY + OVULATION_WINDOW - 1
                                    ? "bg-purple-500"
                                    : "bg-gray-400"
                            : "bg-gray-400"
                    }`}
                    style="transform: rotate({(360 / CYCLE_LENGTH) *
                        i}deg) translateY(-7.5rem)"
                ></div>
            {/each}

            <!-- Center circle -->
            <div
                class="w-52 h-52 rounded-full border-2 border-gray-400 flex flex-col space-y-2 items-center justify-center bg-white shadow"
            >
                <p class="text-base text-gray-500">Edit / Log</p>
                <p class="text-base font-bold text-red-500">Your Date</p>
                <button
                    class="mt-2 text-base px-2.5 py-2 bg-red-500 text-white rounded-full w-32"
                    on:click={() => (showModal = true)}
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
            <p class="text-center text-gray-600 font-funky">
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
                        class="w-10 h-10 rounded-full hover:bg-red-100 text-sm font-medium font-inter transition-colors"
                        on:click={() => selectDate(day.date)}
                    >
                        {new Date(day.date).getDate()}
                    </button>
                {/each}
            </div>
            <div class="flex justify-end mt-4">
                <button
                    class="px-4 py-2 bg-gray-200 rounded-full hover:bg-gray-300"
                    on:click={() => (showModal = false)}>Close</button
                >
            </div>
        </div>
    </div>
{/if}

<!-- === Calendar === -->
<div class="relative z-10">
    <div class="flex justify-center min-h-screen">
        <div class="grid grid-cols-7 gap-2 p-4 lg:hidden">
            {#each days as day, i}
                {#if day.status === "upcoming"}
                    <!-- Wrapper for gradient border -->
                    <div
                        class="w-10 h-10 rounded-full p-[2px] bg-gradient-to-br from-red-400 to-red-600"
                    >
                        <div
                            class="flex items-center justify-center w-full h-full rounded-full bg-white font-inter text-red-500"
                        >
                            {new Date(day.date).getDate()}
                        </div>
                    </div>
                {:else}
                    <div
                        class={`flex items-center justify-center w-10 h-10 rounded-full relative font-inter
            ${day.status === "completed" ? "period-completed text-white" : ""}
            ${day.status === "ovulation" ? "ovulation text-white" : ""}
            ${day.status === "active" ? "border border-red-500 text-red-500" : ""}`}
                    >
                        {new Date(day.date).getDate()}
                    </div>
                {/if}

                <!-- separator -->
                {#if i < days.length - 1 && days[i].month === "prev" && days[i + 1].month === "current"}
                    <div
                        class="col-span-7 border-b-2 border-gray-300 my-2"
                    ></div>
                {/if}
            {/each}
        </div>
    </div>
</div>
