<script lang="ts">
    import { fade, slide } from 'svelte/transition';
    type PatientData = {
        age: number;
        gender: 'Male' | 'Female';
        tot_bilirubin: number;
        direct_bilirubin: number;
        alkphos: number;
        sgpt: number;
        sgot: number;
        tot_proteins: number;
        albumin: number;
        ag_ratio: number;
    };

    let data: PatientData = {
        age: 45,
        gender: 'Male',
        tot_bilirubin: 0.8,
        direct_bilirubin: 0.2,
        alkphos: 150.0,
        sgpt: 35.0,
        sgot: 40.0,
        tot_proteins: 6.5,
        albumin: 3.5,
        ag_ratio: 1.0
    };

    type PredictionResult = {
        success: boolean;
        prediction_code: number;
        prediction_text: string;
        confidence_score: number;
        color: string;
        clinical_description: string;
        stage_confidence: number[];
        shap_plot_base64: string;
    } | null;

    let isLoading = $state(false);
    let result: PredictionResult = $state(null);
    let errorMessage = $state('');

    async function analyzeData(event: Event) {
        event.preventDefault();
        isLoading = true;
        errorMessage = '';
        result = null;

        try {
            const apiUrl = import.meta.env.VITE_API_URL || 'http://localhost:8000';
            const response = await fetch(`${apiUrl}/predict`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(data)
            });

            if (!response.ok) {
                const errData = await response.json();
                throw new Error(errData.detail || 'Failed to fetch prediction');
            }

            result = await response.json();
            
            // Scroll slightly down to see result smoothly
            setTimeout(() => {
                window.scrollBy({ top: 400, behavior: 'smooth' });
            }, 100);

        } catch (error: any) {
            errorMessage = error.message;
        } finally {
            isLoading = false;
        }
    }
</script>

<div class="mb-10 text-center sm:text-left">
    <h1 class="text-4xl md:text-5xl font-extrabold text-[#ff4b4b] tracking-tight mb-2">🧬 HepaGuard AI V2</h1>
    <p class="text-[#a0aec0] text-lg italic tracking-wide">Production Ensemble Learning & Diagnostic XAI Backend</p>
</div>

<div class="bg-red-500/10 border-l-4 border-[#ff4b4b] p-5 rounded-r-lg mb-10 shadow-sm">
    <p class="text-white"><strong>Welcome Doctor.</strong> Enter the patient's biomarker data below. The AI will classify the patient into <strong>Healthy</strong> or <strong>Liver Disease Stages 1-4</strong> through the FastAPI Backend predicting engine.</p>
</div>

<div class="grid lg:grid-cols-12 gap-10">
    <!-- FORM SECTION -->
    <div class="lg:col-span-5 bg-[#1e2430] rounded-2xl p-6 shadow-xl border border-gray-800 relative z-10 transition-all hover:border-gray-700">
        <h2 class="text-2xl font-bold mb-6 text-white border-b border-gray-700 pb-3 flex items-center">
            <span class="mr-3">📝</span> Patient Clinical Data
        </h2>

        <form onsubmit={analyzeData} class="space-y-4">
            <div class="grid grid-cols-2 gap-4">
                <div class="flex flex-col gap-1">
                    <label class="text-xs font-semibold text-gray-400 uppercase tracking-wider" for="age">Age (Years)</label>
                    <input class="bg-[#2a3140] text-white p-3 rounded-lg border border-transparent focus:border-[#ff4b4b] focus:ring-1 focus:ring-[#ff4b4b] outline-none transition-colors" type="number" id="age" bind:value={data.age} min="1" max="120" required />
                </div>
                <div class="flex flex-col gap-1">
                    <label class="text-xs font-semibold text-gray-400 uppercase tracking-wider" for="gender">Gender</label>
                    <select class="bg-[#2a3140] text-white p-3 rounded-lg border border-transparent focus:border-[#ff4b4b] focus:ring-1 focus:ring-[#ff4b4b] outline-none transition-colors appearance-none" id="gender" bind:value={data.gender} required>
                        <option value="Male">Male</option>
                        <option value="Female">Female</option>
                    </select>
                </div>
            </div>

            <div class="grid grid-cols-2 gap-4 pt-2">
                <div class="flex flex-col gap-1">
                    <label class="text-xs font-semibold text-gray-400 uppercase tracking-wider" for="tot_bil">Total Bilirubin</label>
                    <input class="bg-[#2a3140] p-3 rounded-lg border border-transparent focus:border-[#ff4b4b] focus:ring-1 focus:ring-[#ff4b4b] outline-none transition-colors" type="number" step="0.1" id="tot_bil" bind:value={data.tot_bilirubin} required />
                </div>
                <div class="flex flex-col gap-1">
                    <label class="text-xs font-semibold text-gray-400 uppercase tracking-wider" for="dir_bil">Direct Bilirubin</label>
                    <input class="bg-[#2a3140] p-3 rounded-lg border border-transparent focus:border-[#ff4b4b] focus:ring-1 focus:ring-[#ff4b4b] outline-none transition-colors" type="number" step="0.1" id="dir_bil" bind:value={data.direct_bilirubin} required />
                </div>
            </div>

            <div class="grid grid-cols-2 gap-4 pt-2">
                <div class="flex flex-col gap-1">
                    <label class="text-xs font-semibold text-gray-400 uppercase tracking-wider" for="alkphos">Alk. Phosphatase</label>
                    <input class="bg-[#2a3140] p-3 rounded-lg border border-transparent focus:border-[#ff4b4b] focus:ring-1 focus:ring-[#ff4b4b] outline-none transition-colors" type="number" id="alkphos" bind:value={data.alkphos} required />
                </div>
                <div class="flex flex-col gap-1">
                    <label class="text-xs font-semibold text-gray-400 uppercase tracking-wider" for="sgpt">SGPT (ALT)</label>
                    <input class="bg-[#2a3140] p-3 rounded-lg border border-transparent focus:border-[#ff4b4b] focus:ring-1 focus:ring-[#ff4b4b] outline-none transition-colors" type="number" id="sgpt" bind:value={data.sgpt} required />
                </div>
            </div>

            <div class="grid grid-cols-2 gap-4 pt-2">
                <div class="flex flex-col gap-1">
                    <label class="text-xs font-semibold text-gray-400 uppercase tracking-wider" for="sgot">SGOT (AST)</label>
                    <input class="bg-[#2a3140] p-3 rounded-lg border border-transparent focus:border-[#ff4b4b] focus:ring-1 focus:ring-[#ff4b4b] outline-none transition-colors" type="number" id="sgot" bind:value={data.sgot} required />
                </div>
                <div class="flex flex-col gap-1">
                    <label class="text-xs font-semibold text-gray-400 uppercase tracking-wider" for="tot_proteins">Total Proteins</label>
                    <input class="bg-[#2a3140] p-3 rounded-lg border border-transparent focus:border-[#ff4b4b] focus:ring-1 focus:ring-[#ff4b4b] outline-none transition-colors" type="number" step="0.1" id="tot_proteins" bind:value={data.tot_proteins} required />
                </div>
            </div>

            <div class="grid grid-cols-2 gap-4 pt-2">
                <div class="flex flex-col gap-1">
                    <label class="text-xs font-semibold text-gray-400 uppercase tracking-wider" for="albumin">Albumin</label>
                    <input class="bg-[#2a3140] p-3 rounded-lg border border-transparent focus:border-[#ff4b4b] focus:ring-1 focus:ring-[#ff4b4b] outline-none transition-colors" type="number" step="0.1" id="albumin" bind:value={data.albumin} required />
                </div>
                <div class="flex flex-col gap-1">
                    <label class="text-xs font-semibold text-gray-400 uppercase tracking-wider" for="ag_ratio">A/G Ratio</label>
                    <input class="bg-[#2a3140] p-3 rounded-lg border border-transparent focus:border-[#ff4b4b] focus:ring-1 focus:ring-[#ff4b4b] outline-none transition-colors" type="number" step="0.1" id="ag_ratio" bind:value={data.ag_ratio} required />
                </div>
            </div>

            <div class="pt-6">
                <button 
                    type="submit" 
                    disabled={isLoading}
                    class="w-full bg-gradient-to-r from-[#ff4b4b] to-[#ff7b7b] hover:from-[#e33e3e] hover:to-[#ff6b6b] disabled:opacity-50 disabled:cursor-not-allowed text-white font-bold py-4 px-6 rounded-xl shadow-[0_5px_15px_rgba(255,75,75,0.3)] hover:shadow-[0_8px_20px_rgba(255,75,75,0.5)] hover:-translate-y-1 transition-all duration-300 flex justify-center items-center"
                >
                    {#if isLoading}
                        <svg class="animate-spin -ml-1 mr-3 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                        </svg>
                        ANALYZING WITH FASTAPI...
                    {:else}
                        ANALYZE PATIENT DATA
                    {/if}
                </button>
            </div>
        </form>

        {#if errorMessage}
            <div transition:slide class="mt-4 bg-red-900/50 border border-red-500 text-red-200 p-4 rounded-lg">
                <strong>Error:</strong> {errorMessage}
            </div>
        {/if}
    </div>

    <!-- RESULTS SECTION -->
    <div class="lg:col-span-7 relative">
        {#if !result && !isLoading}
            <!-- Empty State -->
             <div class="h-full min-h-[500px] flex items-center justify-center border-2 border-dashed border-gray-700 rounded-3xl bg-[#131720] text-gray-500 flex-col gap-4">
                <svg class="w-16 h-16 opacity-50" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 002-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10"></path></svg>
                <h2 class="text-xl">Awaiting Fast API Inference...</h2>
             </div>
        {/if}

        {#if isLoading}
            <!-- Loading Skeleton -->
            <div class="h-full min-h-[500px] flex flex-col justify-center items-center rounded-3xl bg-[#131720] border border-gray-800">
               <div class="animate-pulse flex flex-col items-center space-y-6 w-full max-w-md">
                   <div class="rounded-full bg-gray-700 h-24 w-24"></div>
                   <div class="h-4 bg-gray-700 rounded w-3/4"></div>
                   <div class="space-y-3 w-full px-8">
                       <div class="h-2 bg-gray-700 rounded"></div>
                       <div class="h-2 bg-gray-700 rounded w-5/6"></div>
                   </div>
               </div>
            </div>
        {/if}

        {#if result && !isLoading}
            <!-- Populated Results -->
             <div transition:fade={{duration: 400}} class="space-y-6">
                <!-- Staging Card -->
                <div class="bg-[#1e2430] p-8 rounded-2xl shadow-xl border-t-[6px] text-center transform transition duration-500 hover:scale-[1.02]" style="border-top-color: {result.color};">
                    <h2 class="text-2xl font-bold mb-2 tracking-wide" style="color: {result.color};">{result.prediction_text}</h2>
                    <h1 class="text-6xl font-black text-white my-3 drop-shadow-md">{(result.confidence_score * 100).toFixed(1)}%</h1>
                    <p class="text-gray-400 uppercase tracking-widest text-sm font-semibold">AI Confidence for Primary Diagnosis</p>
                    <p class="mt-6 text-gray-300 bg-gray-800/50 p-4 rounded-lg inline-block text-left w-full border border-gray-700/50">{result.clinical_description}</p>
                </div>

                <!-- Confidence Bars -->
                <div class="bg-[#1e2430] p-6 rounded-2xl shadow-xl border border-gray-800">
                    <h3 class="text-xl font-bold text-white mb-6 flex items-center">
                        <span class="mr-2">📊</span> Diagnostic Probability Distribution
                    </h3>
                    <div class="space-y-4">
                        {#each ['Healthy Liver', 'Stage 1: Inflammation', 'Stage 2: Significant', 'Stage 3: Severe', 'Stage 4: Critical'] as label, i}
                            <div class="space-y-1">
                                <div class="flex justify-between text-xs font-semibold uppercase tracking-wider">
                                    <span class={i === result.prediction_code ? 'text-[#ff4b4b]' : 'text-gray-400'}>{label}</span>
                                    <span class="text-white">{(result.stage_confidence[i] * 100).toFixed(1)}%</span>
                                </div>
                                <div class="h-2 w-full bg-gray-800 rounded-full overflow-hidden">
                                    <div 
                                        class="h-full transition-all duration-1000 ease-out" 
                                        style="width: {result.stage_confidence[i] * 100}%; background-color: {i === result.prediction_code ? result.color : '#374151'}"
                                    ></div>
                                </div>
                            </div>
                        {/each}
                    </div>
                </div>

                <!-- SHAP Card -->
                <div class="bg-[#1e2430] p-6 rounded-2xl shadow-xl border border-gray-800">
                    <h3 class="text-xl font-bold text-white mb-2 flex items-center">
                        <span class="mr-2">🧠</span> Deep Medical Insight (CatBoost SHAP)
                    </h3>
                    <p class="text-gray-400 text-sm mb-6">This chart explains the exact factors the CatBoost estimator used to reach the diagnosis. Red variables increase disease risk, blue decrease it.</p>
                    
                    <div class="bg-gray-100 p-2 rounded-xl overflow-hidden shadow-inner flex justify-center mt-2 group relative">
                         <!-- The image comes raw base64 from the API -->
                         <img 
                            src={`data:image/png;base64,${result.shap_plot_base64}`} 
                            alt="SHAP Local Waterfall Plot" 
                            class="w-full mix-blend-multiply opacity-90 group-hover:opacity-100 transition-opacity" 
                        />
                    </div>
                </div>
             </div>
        {/if}
    </div>
</div>
