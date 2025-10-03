<script lang="ts">
    import { goto } from "$app/navigation";

    function navigateToHome() {
        goto("/");
    }
</script>

<!-- svelte-ignore a11y_no_static_element_interactions -->
<div
    class="flex justify-center items-center align-middle bg-gradient-to-br from-red-200 via-purple-50 to-pink-100 border-b shadow-xl py-3 w-full"
>
    <!-- svelte-ignore a11y_click_events_have_key_events -->
    <div
        class="flex space-x-2 items-center cursor-pointer"
        onclick={navigateToHome}
    >
        <svg
            xmlns="http://www.w3.org/2000/svg"
            viewBox="0 0 24 24"
            class="w-10 h-10"
        >
            <defs>
                <linearGradient
                    id="dropGradient"
                    x1="0%"
                    y1="0%"
                    x2="0%"
                    y2="100%"
                >
                    <stop
                        offset="0%"
                        style="stop-color:#f87171;stop-opacity:1"
                    />
                    <stop
                        offset="100%"
                        style="stop-color:#dc2626;stop-opacity:1"
                    />
                </linearGradient>
            </defs>
            <path
                fill="url(#dropGradient)"
                d="M12 2c-1 2-8 9-8 13.5C4 19.4 7.6 23 12 23s8-3.6 8-7.5C20 11 13 4 12 2z"
            />
        </svg>
        <p
            class="font-funky bg-gradient-to-r from-red-300 to-red-600 bg-clip-text text-transparent"
        >
            Chycndi Tracker
        </p>
    </div>
</div>
