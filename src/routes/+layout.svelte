<script>
    import '../app.css';
    let { children } = $props();
</script>

<div class="min-h-screen bg-[#0e1117] text-[#e0e6ed] font-sans selection:bg-[#ff4b4b] selection:text-white flex flex-col">
    
    <!-- Premium Sticky Header -->
    <header class="sticky top-0 z-50 bg-[#1e2430]/80 backdrop-blur-md border-b border-gray-800 shadow-sm">
        <div class="h-1 bg-gradient-to-r from-[#ff4b4b] to-[#e67e22]"></div>
        <div class="max-w-7xl mx-auto px-6 h-16 flex items-center justify-between">
            <div class="flex items-center space-x-3">
                <svg class="w-8 h-8 text-[#ff4b4b]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M15 2v20"></path><path d="M9 2v20"></path><path d="M4 6h16"></path><path d="M4 12h16"></path><path d="M4 18h16"></path><circle cx="9" cy="6" r="2"></circle><circle cx="15" cy="12" r="2"></circle><circle cx="9" cy="18" r="2"></circle>
                </svg>
                <div class="flex flex-col">
                    <span class="text-xl font-black tracking-widest text-white uppercase leading-none">SENTINEL</span>
                    <span class="text-[0.6rem] font-bold tracking-[0.2em] text-[#ff4b4b] uppercase leading-none mt-1">Diagnostic Intelligence</span>
                </div>
            </div>
            
            <div class="hidden sm:flex items-center space-x-6">
                <div class="flex items-center text-xs font-semibold text-gray-400 bg-gray-800/50 px-3 py-1.5 rounded-full border border-gray-700/50">
                    <span class="w-2 h-2 rounded-full bg-green-500 mr-2 pulse-dot"></span>
                    BACKEND Engine Online
                </div>
            </div>
        </div>
    </header>

    <!-- Main Content Area -->
    <main class="flex-grow max-w-7xl w-full mx-auto px-4 sm:px-6 py-8 relative">
        {@render children()}
    </main>

    <!-- Minimal Footer -->
    <footer class="w-full bg-[#1e2430] border-t border-gray-800 py-6 mt-12">
        <div class="max-w-7xl mx-auto px-6 flex flex-col sm:flex-row justify-between items-center text-gray-500 text-sm">
            <p>&copy; {new Date().getFullYear()} SENTINEL Platform v2.1 VISHWAJEET VIJAY PAWAR</p>
            <p class="mt-2 sm:mt-0 opacity-70">Powered by CatBoost & SHAP XAI Models</p>
        </div>
    </footer>

</div>

<style>
    :global(body) {
        margin: 0;
        background-color: #0e1117;
    }
</style>
