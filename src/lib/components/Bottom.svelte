<script>
    import { Home, Gift, Settings } from "@lucide/svelte/icons";
    import { page } from "$app/stores";
</script>

<slot />

<!-- Bottom Navigation -->
<div
    class="flex justify-evenly items-center align-middle space-x-3 py-1 border-t shadow-1 bg-gradient-to-br from-pink-200 via-purple-50 to-red-100 fixed bottom-0 left-0 right-0"
>
    <a
        href="/"
        data-sveltekit-preload-data="hover"
        class="flex flex-col items-center justify-center px-4 py-0 rounded-lg transition-colors {$page
            .url.pathname === '/'
            ? 'text-red-500'
            : 'text-gray-600 hover:text-red-400'}"
    >
        <Home size={24} />
        <span class="text-xs mt-1">Home</span>
    </a>

    <a
        href="/gift"
        data-sveltekit-preload-data="hover"
        class="flex flex-col items-center justify-center px-4 py-2 rounded-lg transition-colors {$page
            .url.pathname === '/gift'
            ? 'text-red-500'
            : 'text-gray-600 hover:text-red-400'}"
    >
        <Gift size={24} />
        <span class="text-xs mt-1">Gift</span>
    </a>

    <a
        href="/settings"
        data-sveltekit-preload-data="hover"
        class="flex flex-col items-center justify-center px-4 py-2 rounded-lg transition-colors {$page
            .url.pathname === '/settings'
            ? 'text-red-500'
            : 'text-gray-600 hover:text-red-400'}"
    >
        <Settings size={24} />
        <span class="text-xs mt-1">Settings</span>
    </a>
</div>
