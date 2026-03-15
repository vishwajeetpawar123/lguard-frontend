<script>
    import '../app.css';
    let { children } = $props();
</script>

<div class="min-h-screen bg-[#0e1117] text-[#e0e6ed] font-sans selection:bg-[#ff4b4b] selection:text-white">
    <main class="max-w-7xl mx-auto px-4 py-8 relative">
        {@render children()}
    </main>
</div>

<style>
    :global(body) {
        margin: 0;
        background-color: #0e1117;
    }
</style>
