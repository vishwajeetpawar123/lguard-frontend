<script lang="ts">
    import { fade, slide } from 'svelte/transition';
    import ReportDownload from '$lib/ReportDownload.svelte';
    type PatientData = {
        age: number | null;
        gender: 'Male' | 'Female' | null;
        tot_bilirubin: number | null;
        direct_bilirubin: number | null;
        alkphos: number | null;
        sgpt: number | null;
        sgot: number | null;
        tot_proteins: number | null;
        albumin: number | null;
        ag_ratio: number | null;
    };

    let data: PatientData = $state({
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
    });

    type PredictionResult = {
        success: boolean;
        prediction_code: number;
        prediction_text: string;
        confidence_score: number;
        color: string;
        clinical_description: string;
        stage_confidence: number[];
        shap_plot_base64: string;
        text_explanation: string;
        imputed_data: Record<string, number>;
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
    <h1 class="text-4xl md:text-5xl font-extrabold text-[#ff4b4b] tracking-tight mb-2">🧬 SENTINEL</h1>
    <p class="text-[#a0aec0] text-lg italic tracking-wide">Production Ensemble Learning & Diagnostic XAI Backend</p>
</div>

<div class="bg-gradient-to-r from-red-500/20 to-transparent border-l-4 border-[#ff4b4b] p-6 rounded-r-xl mb-10 shadow-lg backdrop-blur-sm">
    <p class="text-gray-200 leading-relaxed"><strong class="text-white text-lg font-semibold tracking-wide">Welcome, Doctor.</strong><br>Enter the patient's biomarker data below. SENTINEL will classify the patient into <strong>Healthy</strong> or <strong>Liver Disease Stages 1-4</strong> through the FastAPI predicting engine, providing deep SHAP-based medical insights.</p>
</div>

<div class="grid lg:grid-cols-12 gap-10">
    <!-- FORM SECTION -->
    <div class="lg:col-span-5 glass-card rounded-2xl p-8 relative z-10 transition-all hover:border-gray-600">
        <h2 class="text-2xl font-bold mb-6 text-white border-b border-gray-700/50 pb-4 flex items-center">
            <svg class="w-6 h-6 mr-3 text-[#ff4b4b]" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"></path></svg>
            Patient Clinical Data
        </h2>

        <form onsubmit={analyzeData} class="space-y-5">
            <div class="grid grid-cols-2 gap-5">
                <div class="flex flex-col gap-1.5 group">
                    <label class="text-xs font-bold text-gray-400 uppercase tracking-wider flex items-center" for="age">
                        <span class="mr-1.5 opacity-70">👤</span> Age (Years)
                    </label>
                    <input class="w-full bg-[#131720]/80 text-white p-3.5 rounded-xl border border-gray-700/50 focus:border-[#ff4b4b] focus:ring-2 focus:ring-[#ff4b4b]/30 outline-none transition-all duration-300 shadow-inner group-hover:border-gray-600" type="number" id="age" bind:value={data.age} min="1" max="120" />
                </div>
                <div class="flex flex-col gap-1.5 group">
                    <label class="text-xs font-bold text-gray-400 uppercase tracking-wider flex items-center" for="gender">
                        <span class="mr-1.5 opacity-70">⚧️</span> Gender
                    </label>
                    <select class="w-full bg-[#131720]/80 text-white p-3.5 rounded-xl border border-gray-700/50 focus:border-[#ff4b4b] focus:ring-2 focus:ring-[#ff4b4b]/30 outline-none transition-all duration-300 shadow-inner appearance-none group-hover:border-gray-600 cursor-pointer" id="gender" bind:value={data.gender}>
                        <option value="Male">Male</option>
                        <option value="Female">Female</option>
                    </select>
                </div>
            </div>

            <div class="grid grid-cols-2 gap-5">
                <div class="flex flex-col gap-1.5 group" title="Normal range: 0.1–1.2 mg/dL">
                    <label class="text-xs font-bold text-gray-400 uppercase tracking-wider flex items-center cursor-help" for="tot_bil">
                        <span class="mr-1.5 opacity-70">🩸</span> Total Bilirubin
                    </label>
                    <input class="w-full bg-[#131720]/80 text-white p-3.5 rounded-xl border border-gray-700/50 focus:border-[#ff4b4b] focus:ring-2 focus:ring-[#ff4b4b]/30 outline-none transition-all duration-300 shadow-inner group-hover:border-gray-600" type="number" step="0.1" id="tot_bil" bind:value={data.tot_bilirubin} />
                </div>
                <div class="flex flex-col gap-1.5 group" title="Normal range: 0.0–0.3 mg/dL">
                    <label class="text-xs font-bold text-gray-400 uppercase tracking-wider flex items-center cursor-help" for="dir_bil">
                        <span class="mr-1.5 opacity-70">🩸</span> Direct Bilirubin
                    </label>
                    <input class="w-full bg-[#131720]/80 text-white p-3.5 rounded-xl border border-gray-700/50 focus:border-[#ff4b4b] focus:ring-2 focus:ring-[#ff4b4b]/30 outline-none transition-all duration-300 shadow-inner group-hover:border-gray-600" type="number" step="0.1" id="dir_bil" bind:value={data.direct_bilirubin} />
                </div>
            </div>

            <div class="grid grid-cols-2 gap-5">
                <div class="flex flex-col gap-1.5 group" title="Normal range: 44–147 IU/L">
                    <label class="text-xs font-bold text-gray-400 uppercase tracking-wider flex items-center cursor-help" for="alkphos">
                        <span class="mr-1.5 opacity-70">🧪</span> Alk. Phosphatase
                    </label>
                    <input class="w-full bg-[#131720]/80 text-white p-3.5 rounded-xl border border-gray-700/50 focus:border-[#ff4b4b] focus:ring-2 focus:ring-[#ff4b4b]/30 outline-none transition-all duration-300 shadow-inner group-hover:border-gray-600" type="number" id="alkphos" bind:value={data.alkphos} />
                </div>
                <div class="flex flex-col gap-1.5 group" title="Normal range: 7–56 U/L. Assesses liver damage.">
                    <label class="text-xs font-bold text-gray-400 uppercase tracking-wider flex items-center cursor-help" for="sgpt">
                        <span class="mr-1.5 opacity-70">📈</span> SGPT (ALT)
                    </label>
                    <input class="w-full bg-[#131720]/80 text-white p-3.5 rounded-xl border border-gray-700/50 focus:border-[#ff4b4b] focus:ring-2 focus:ring-[#ff4b4b]/30 outline-none transition-all duration-300 shadow-inner group-hover:border-gray-600" type="number" id="sgpt" bind:value={data.sgpt} />
                </div>
            </div>

            <div class="grid grid-cols-2 gap-5">
                <div class="flex flex-col gap-1.5 group" title="Normal range: 10–40 U/L. Assesses liver damage.">
                    <label class="text-xs font-bold text-gray-400 uppercase tracking-wider flex items-center cursor-help" for="sgot">
                        <span class="mr-1.5 opacity-70">📉</span> SGOT (AST)
                    </label>
                    <input class="w-full bg-[#131720]/80 text-white p-3.5 rounded-xl border border-gray-700/50 focus:border-[#ff4b4b] focus:ring-2 focus:ring-[#ff4b4b]/30 outline-none transition-all duration-300 shadow-inner group-hover:border-gray-600" type="number" id="sgot" bind:value={data.sgot} />
                </div>
                <div class="flex flex-col gap-1.5 group" title="Normal range: 6.0–8.3 g/dL">
                    <label class="text-xs font-bold text-gray-400 uppercase tracking-wider flex items-center cursor-help" for="tot_proteins">
                        <span class="mr-1.5 opacity-70">🥩</span> Total Proteins
                    </label>
                    <input class="w-full bg-[#131720]/80 text-white p-3.5 rounded-xl border border-gray-700/50 focus:border-[#ff4b4b] focus:ring-2 focus:ring-[#ff4b4b]/30 outline-none transition-all duration-300 shadow-inner group-hover:border-gray-600" type="number" step="0.1" id="tot_proteins" bind:value={data.tot_proteins} />
                </div>
            </div>

            <div class="grid grid-cols-2 gap-5">
                <div class="flex flex-col gap-1.5 group" title="Normal range: 3.4–5.4 g/dL. Main protein made by liver.">
                    <label class="text-xs font-bold text-gray-400 uppercase tracking-wider flex items-center cursor-help" for="albumin">
                        <span class="mr-1.5 opacity-70">🛡️</span> Albumin
                    </label>
                    <input class="w-full bg-[#131720]/80 text-white p-3.5 rounded-xl border border-gray-700/50 focus:border-[#ff4b4b] focus:ring-2 focus:ring-[#ff4b4b]/30 outline-none transition-all duration-300 shadow-inner group-hover:border-gray-600" type="number" step="0.1" id="albumin" bind:value={data.albumin} />
                </div>
                <div class="flex flex-col gap-1.5 group" title="Normal range: 1.1–2.5">
                    <label class="text-xs font-bold text-gray-400 uppercase tracking-wider flex items-center cursor-help" for="ag_ratio">
                        <span class="mr-1.5 opacity-70">⚖️</span> A/G Ratio
                    </label>
                    <input class="w-full bg-[#131720]/80 text-white p-3.5 rounded-xl border border-gray-700/50 focus:border-[#ff4b4b] focus:ring-2 focus:ring-[#ff4b4b]/30 outline-none transition-all duration-300 shadow-inner group-hover:border-gray-600" type="number" step="0.1" id="ag_ratio" bind:value={data.ag_ratio} />
                </div>
            </div>

            <div class="pt-6 flex gap-3">
                <button 
                    type="button" 
                    title="Reset to default normal values"
                    class="bg-[#2a3140] hover:bg-[#374151] text-gray-300 font-bold py-4 px-5 rounded-xl border border-gray-700 transition-colors duration-300 flex items-center justify-center"
                    onclick={() => {
                        data = { age: 45, gender: 'Male', tot_bilirubin: 0.8, direct_bilirubin: 0.2, alkphos: 150.0, sgpt: 35.0, sgot: 40.0, tot_proteins: 6.5, albumin: 3.5, ag_ratio: 1.0 };
                        result = null;
                        errorMessage = '';
                    }}
                >
                    <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"></path></svg>
                </button>
                
                <button 
                    type="submit" 
                    disabled={isLoading}
                    class="flex-1 bg-gradient-to-r from-[#ff4b4b] to-[#ff7b7b] hover:from-[#e33e3e] hover:to-[#ff6b6b] disabled:opacity-50 disabled:cursor-not-allowed text-white font-bold py-4 px-6 rounded-xl shadow-[0_5px_15px_rgba(255,75,75,0.3)] hover:shadow-[0_8px_20px_rgba(255,75,75,0.5)] hover:-translate-y-1 transition-all duration-300 flex justify-center items-center glow-border"
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
            <div transition:slide class="mt-5 bg-red-900/40 border-l-4 border-red-500 text-red-200 p-4 rounded-r-lg backdrop-blur-sm">
                <strong class="font-bold flex items-center mb-1"><svg class="w-4 h-4 mr-2" fill="currentColor" viewBox="0 0 20 20"><path fill-rule="evenodd" d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7 4a1 1 0 11-2 0 1 1 0 012 0zm-1-9a1 1 0 00-1 1v4a1 1 0 102 0V6a1 1 0 00-1-1z" clip-rule="evenodd"></path></svg> Error:</strong> {errorMessage}
            </div>
        {/if}
    </div>

    <!-- RESULTS SECTION -->
    <div class="lg:col-span-7 relative">
        {#if !result && !isLoading}
            <!-- Empty State -->
             <div class="h-full min-h-[500px] flex items-center justify-center border-2 border-dashed border-gray-700/50 rounded-3xl bg-[#131720]/80 backdrop-blur-sm text-gray-500 flex-col gap-6 relative overflow-hidden group">
                <!-- Parallax Background Element -->
                <div class="absolute inset-0 opacity-5 bg-[radial-gradient(ellipse_at_center,_var(--tw-gradient-stops))] from-[#ff4b4b] via-[#0e1117] to-transparent group-hover:scale-110 transition-transform duration-1000"></div>
                
                <div class="relative z-10 p-6 bg-[#1e2430]/50 rounded-full border border-gray-800 shadow-xl">
                    <svg class="w-16 h-16 text-gray-600 group-hover:text-[#ff4b4b] transition-colors duration-500" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 002-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10"></path></svg>
                </div>
                <h2 class="text-xl font-medium tracking-wide z-10 relative text-gray-300">CLICK ANALYZE PATIENT DATA..</h2>
                <p class="text-xs text-gray-500 max-w-xs text-center z-10 relative">Submit patient clinical data to generate the AI diagnostic staging and SHAP analysis plot.</p>
             </div>
        {/if}

        {#if isLoading}
            <!-- Loading Skeleton - Shimmer Effect -->
            <div class="h-full min-h-[500px] flex flex-col justify-center items-center rounded-3xl bg-[#131720]/80 backdrop-blur-sm border border-gray-800 shadow-inner">
               <div class="flex flex-col items-center space-y-8 w-full max-w-md">
                   <!-- Circular Shimmer -->
                   <div class="relative overflow-hidden rounded-full bg-gray-800 h-32 w-32 border border-gray-700 shadow-[0_0_20px_rgba(255,75,75,0.1)]">
                       <div class="absolute inset-0 -translate-x-full animate-[shimmer_1.5s_infinite] bg-gradient-to-r from-transparent via-gray-700/50 to-transparent"></div>
                   </div>
                   <!-- Lines Shimmer -->
                   <div class="w-full px-8 space-y-4">
                       <div class="relative overflow-hidden h-4 bg-gray-800 rounded w-3/4 mx-auto">
                           <div class="absolute inset-0 -translate-x-full animate-[shimmer_1.5s_infinite] bg-gradient-to-r from-transparent via-gray-700/50 to-transparent"></div>
                       </div>
                       <div class="relative overflow-hidden h-3 bg-gray-800 rounded w-full">
                           <div class="absolute inset-0 -translate-x-full animate-[shimmer_1.5s_infinite] bg-gradient-to-r from-transparent via-gray-700/50 to-transparent"></div>
                       </div>
                       <div class="relative overflow-hidden h-3 bg-gray-800 rounded w-5/6">
                           <div class="absolute inset-0 -translate-x-full animate-[shimmer_1.5s_infinite] bg-gradient-to-r from-transparent via-gray-700/50 to-transparent"></div>
                       </div>
                   </div>
               </div>
            </div>
            <style>
                @keyframes shimmer {
                    100% { transform: translateX(100%); }
                }
            </style>
        {/if}

        {#if result && !isLoading}
            <!-- Populated Results -->
             <div class="space-y-6">
                <!-- Staging Card - Animated Entrance -->
                <div in:fade={{duration: 400, delay: 0}} class="glass-card p-8 rounded-2xl border-t-[6px] text-center transform transition duration-500" style="border-top-color: {result.color}; box-shadow: 0 10px 40px -10px {result.color}40;">
                    
                    <div class="flex justify-between items-center mb-6">
                        <h2 class="text-2xl font-black tracking-wide" style="color: {result.color};">{result.prediction_text}</h2>
                        <span class="pulse-dot" style="background-color: {result.color}; box-shadow: 0 0 10px {result.color};"></span>
                    </div>
                    
                    <div class="flex flex-col sm:flex-row items-center justify-center gap-10 my-8">
                        <!-- Circular SVG Gauge -->
                        <div class="relative w-40 h-40">
                            <!-- Background Circle -->
                            <svg class="w-full h-full -rotate-90" viewBox="0 0 36 36">
                                <path class="text-gray-800" d="M18 2.0845 a 15.9155 15.9155 0 0 1 0 31.831 a 15.9155 15.9155 0 0 1 0 -31.831" fill="none" stroke="currentColor" stroke-width="3" stroke-dasharray="100, 100" />
                                <!-- Foreground Animated Circle -->
                                <path class="animate-[drawGauge_1.5s_ease-out_forwards]" style="stroke: {result.color}; stroke-dasharray: {result.confidence_score * 100}, 100;" d="M18 2.0845 a 15.9155 15.9155 0 0 1 0 31.831 a 15.9155 15.9155 0 0 1 0 -31.831" fill="none" stroke-width="3" stroke-linecap="round" />
                            </svg>
                            <!-- Inner Text -->
                            <div class="absolute inset-0 flex flex-col items-center justify-center bg-[#131720]/50 rounded-full m-[8px] border border-gray-700/50 backdrop-blur-sm shadow-inner">
                                <span class="text-4xl font-black text-white drop-shadow-[0_2px_4px_rgba(0,0,0,0.8)]">{(result.confidence_score * 100).toFixed(0)}<span class="text-xl opacity-70">%</span></span>
                            </div>
                        </div>
                        <div class="text-left max-w-sm">
                            <p class="text-gray-400 uppercase tracking-widest text-xs font-bold mb-2">AI Diagnostic Confidence</p>
                            <p class="text-gray-300 text-sm leading-relaxed bg-[#131720]/80 p-4 rounded-xl border border-gray-700/50 shadow-inner">{result.clinical_description}</p>
                        </div>
                    </div>
                </div>
                
                <style>
                    @keyframes drawGauge {
                        0% { stroke-dasharray: 0, 100; opacity: 0;}
                    }
                </style>

                <!-- Confidence Bars - Delayed Entrance -->
                <div in:fade={{duration: 400, delay: 150}} class="glass-card p-6 rounded-2xl">
                    <h3 class="text-lg font-bold text-white mb-6 flex items-center border-b border-gray-700/50 pb-3">
                        <span class="mr-2 opacity-80">📊</span> Diagnostic Probability Distribution
                    </h3>
                    <div class="space-y-4">
                        {#each ['Healthy Liver', 'Stage 1: Inflammation', 'Stage 2: Fibrosis/Significant', 'Stage 3: Cirrhosis/Severe', 'Stage 4: End-Stage/Critical'] as label, i}
                            <div class="space-y-1.5 group">
                                <div class="flex justify-between text-xs font-bold uppercase tracking-wider">
                                    <span class="transition-colors duration-300 {i === result.prediction_code ? 'text-white' : 'text-gray-500'}">{label}</span>
                                    <span class="font-mono {i === result.prediction_code ? 'text-white drop-shadow-md' : 'text-gray-500'}">{(result.stage_confidence[i] * 100).toFixed(1)}%</span>
                                </div>
                                <div class="h-2.5 w-full bg-[#131720] rounded-full overflow-hidden shadow-inner border border-gray-800">
                                    <div 
                                        class="h-full transition-all duration-1000 ease-out animate-[slideRight_1s_ease-out_forwards]" 
                                        style="width: {result.stage_confidence[i] * 100}%; background-color: {i === result.prediction_code ? result.color : '#374151'}; opacity: {i === result.prediction_code ? '1' : '0.6'}"
                                    ></div>
                                </div>
                            </div>
                        {/each}
                    </div>
                </div>
                
                <style>
                    @keyframes slideRight {
                        0% { width: 0%; opacity: 0; }
                    }
                </style>

                <!-- Patient Summary Grid - Delayed Entrance -->
                <div in:fade={{duration: 400, delay: 300}} class="glass-card p-6 rounded-2xl">
                    <h3 class="text-lg font-bold text-white mb-4 flex items-center border-b border-gray-700/50 pb-3">
                        <span class="mr-2 opacity-80">📋</span> Patient Biomarker Summary
                    </h3>
                    <div class="grid grid-cols-2 sm:grid-cols-4 gap-y-7 gap-x-2 text-sm bg-[#131720]/80 px-5 pt-5 pb-7 rounded-xl border border-gray-700/50 shadow-inner">
                        <div class="flex flex-col gap-1 border-r border-gray-700/50 pr-2 relative">
                            <span class="text-gray-500 text-xs font-semibold uppercase tracking-wider">Age / Sex</span>
                            <span class="text-white font-mono text-base whitespace-nowrap">{data.age ?? result.imputed_data.age} / {(data.gender ?? (result.imputed_data.gender === 1 ? 'Male' : 'Female'))[0]}</span>
                            {#if data.age === null || data.gender === null}
                                <span class="text-[#a855f7] text-[10px] font-bold uppercase tracking-widest absolute -bottom-5 left-0">Predicted</span>
                            {/if}
                        </div>
                        <div class="flex flex-col gap-1 border-r border-gray-700/50 pr-2 relative">
                            <span class="text-gray-500 text-xs font-semibold uppercase tracking-wider pt-0">Tot. Bilirubin</span>
                            <span class="text-white font-mono text-base">{data.tot_bilirubin !== null ? data.tot_bilirubin : result.imputed_data.tot_bilirubin.toFixed(2)}</span>
                            {#if data.tot_bilirubin === null}
                                <span class="text-[#a855f7] text-[10px] font-bold uppercase tracking-widest absolute -bottom-5 left-0">Predicted</span>
                            {/if}
                        </div>
                        <div class="flex flex-col gap-1 border-r sm:border-r-gray-700/50 border-r-transparent pr-2 relative">
                            <span class="text-gray-500 text-xs font-semibold uppercase tracking-wider pt-0">Dir. Bilirubin</span>
                            <span class="text-white font-mono text-base">{data.direct_bilirubin !== null ? data.direct_bilirubin : result.imputed_data.direct_bilirubin.toFixed(2)}</span>
                            {#if data.direct_bilirubin === null}
                                <span class="text-[#a855f7] text-[10px] font-bold uppercase tracking-widest absolute -bottom-5 left-0">Predicted</span>
                            {/if}
                        </div>
                        <div class="flex flex-col gap-1 relative">
                            <span class="text-gray-500 text-xs font-semibold uppercase tracking-wider pt-0">Alk. Phos.</span>
                            <span class="text-white font-mono text-base">{data.alkphos !== null ? data.alkphos : result.imputed_data.alkphos.toFixed(1)}</span>
                            {#if data.alkphos === null}
                                <span class="text-[#a855f7] text-[10px] font-bold uppercase tracking-widest absolute -bottom-5 left-0">Predicted</span>
                            {/if}
                        </div>
                        
                        <div class="flex flex-col gap-1 border-r border-gray-700/50 pr-2 pt-2 relative">
                            <span class="text-gray-500 text-xs font-semibold uppercase tracking-wider">SGPT (ALT)</span>
                            <span class="text-white font-mono text-base">{data.sgpt !== null ? data.sgpt : result.imputed_data.sgpt.toFixed(1)}</span>
                            {#if data.sgpt === null}
                                <span class="text-[#a855f7] text-[10px] font-bold uppercase tracking-widest absolute -bottom-5 left-0">Predicted</span>
                            {/if}
                        </div>
                        <div class="flex flex-col gap-1 border-r border-gray-700/50 pr-2 pt-2 relative">
                            <span class="text-gray-500 text-xs font-semibold uppercase tracking-wider">SGOT (AST)</span>
                            <span class="text-white font-mono text-base">{data.sgot !== null ? data.sgot : result.imputed_data.sgot.toFixed(1)}</span>
                            {#if data.sgot === null}
                                <span class="text-[#a855f7] text-[10px] font-bold uppercase tracking-widest absolute -bottom-5 left-0">Predicted</span>
                            {/if}
                        </div>
                        <div class="flex flex-col gap-1 border-r sm:border-r-gray-700/50 border-r-transparent pr-2 pt-2 relative">
                            <span class="text-gray-500 text-xs font-semibold uppercase tracking-wider">Tot. Proteins</span>
                            <span class="text-white font-mono text-base">{data.tot_proteins !== null ? data.tot_proteins : result.imputed_data.tot_proteins.toFixed(2)}</span>
                            {#if data.tot_proteins === null}
                                <span class="text-[#a855f7] text-[10px] font-bold uppercase tracking-widest absolute -bottom-5 left-0">Predicted</span>
                            {/if}
                        </div>
                        <div class="flex flex-col gap-1 pt-2 relative">
                            <span class="text-gray-500 text-xs font-semibold uppercase tracking-wider">Alb. / AG</span>
                            <span class="text-white font-mono text-base whitespace-nowrap">{data.albumin !== null ? data.albumin : result.imputed_data.albumin.toFixed(2)} / {data.ag_ratio !== null ? data.ag_ratio : result.imputed_data.ag_ratio.toFixed(2)}</span>
                            {#if data.albumin === null || data.ag_ratio === null}
                                <span class="text-[#a855f7] text-[10px] font-bold uppercase tracking-widest absolute -bottom-5 left-0">Predicted</span>
                            {/if}
                        </div>
                    </div>
                </div>

                <!-- SHAP Card - Delayed Entrance -->
                <div in:fade={{duration: 400, delay: 450}} class="glass-card p-6 rounded-2xl mt-6">
                    <div class="flex justify-between items-center mb-3 border-b border-gray-700/50 pb-3">
                        <h3 class="text-lg font-bold text-white flex items-center">
                            <span class="mr-2 opacity-80">🧠</span> Deep Medical Insight (CatBoost SHAP)
                        </h3>
                    </div>
                    <p class="text-gray-400 text-xs leading-relaxed mb-5"><span class="text-gray-300">This chart explains the exact factors the CatBoost estimator used to reach the diagnosis.</span> <span class="text-[#ff4b4b] font-bold">Red bars</span> increase disease risk, <span class="text-[#3b82f6] font-bold">Blue bars</span> decrease it.</p>
                    
                    <div class="bg-[#f8f9fa] p-3 rounded-xl overflow-hidden flex justify-center group relative border-[3px] border-gray-200 transition-all hover:border-[#ff4b4b]/50">
                         <!-- The image comes raw base64 from the API -->
                         <img 
                            src={`data:image/png;base64,${result.shap_plot_base64}`} 
                            alt="SHAP Local Waterfall Plot" 
                            class="w-full mix-blend-multiply opacity-95 group-hover:opacity-100 transition-opacity duration-300 group-hover:scale-[1.01] transform" 
                        />
                    </div>
                    
                    <!-- Rule-Based Text Explanation -->
                    {#if result.text_explanation}
                         <div class="mt-5 bg-[#131720]/80 p-5 rounded-xl border border-gray-700/50 shadow-inner flex gap-4 items-start group hover:border-gray-600 transition-colors">
                              <div class="text-2xl mt-1 opacity-80 group-hover:scale-110 transition-transform duration-300"> :) </div>
                              <div class="text-gray-300 text-sm leading-relaxed">
                                   <span class="font-bold text-white mb-1.5 block tracking-wide text-xs uppercase opacity-80">AI Clinical Reasoning Summary</span>
                                   <p>{@html result.text_explanation.replace(/\*\*(.*?)\*\*/g, '<strong class="text-white font-semibold">$1</strong>')}</p>
                              </div>
                         </div>
                    {/if}
                </div>

                <!-- Download Report Button -->
                <div in:fade={{duration: 400, delay: 600}} class="pt-4 pb-4">
                     <ReportDownload {data} {result} />
                </div>
             </div>
        {/if}
    </div>
</div>
