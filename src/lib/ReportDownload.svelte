<script lang="ts">
    import jsPDF from 'jspdf';
    
    let { data, result } = $props();

    let isGenerating = $state(false);

    async function generatePDF() {
        if (!result) return;
        isGenerating = true;
        
        try {
            // Need a tiny delay for the UI to update to "Generating..."
            await new Promise(resolve => setTimeout(resolve, 50));

            const doc = new jsPDF({
                orientation: 'portrait',
                unit: 'mm',
                format: 'a4'
            });

            // Colors
            const headerBg = '#0f172a'; // Deep Navy
            const primaryBlue = '#2563eb';
            const textDark = '#1e293b';
            const textMuted = '#64748b';
            const cardBg = '#f8fafc';
            const sectionBorder = '#cbd5e1';
            const pageWidth = doc.internal.pageSize.width;
            const marginX = 20;
            let currentY = 0;

            // --- HEADER ---
            doc.setFillColor(headerBg);
            doc.rect(0, 0, pageWidth, 45, 'F');
            
            // Logo / Title
            doc.setTextColor('#ff4b4b'); // Keep Sentinel Red for logo
            doc.setFont('helvetica', 'bold');
            doc.setFontSize(28);
            doc.text('SENTINEL', marginX, 26);
            
            doc.setTextColor('#ffffff');
            doc.setFont('helvetica', 'normal');
            doc.setFontSize(14);
            doc.text('Diagnostic Intelligence Report', pageWidth - marginX, 26, { align: 'right' });
            
            doc.setFontSize(9);
            doc.setTextColor('#94a3b8');
            const date = new Date().toLocaleString();
            doc.text(`Generated: ${date}`, pageWidth - marginX, 34, { align: 'right' });

            currentY = 55;

            // --- PATIENT SUMMARY SECTION ---
            doc.setTextColor(primaryBlue);
            doc.setFont('helvetica', 'bold');
            doc.setFontSize(14);
            doc.text('Patient Biomarker Summary', marginX, currentY);
            currentY += 8;

            doc.setFillColor(cardBg);
            doc.setDrawColor(sectionBorder);
            doc.setLineWidth(0.5);
            doc.roundedRect(marginX, currentY, pageWidth - 40, 48, 4, 4, 'FD');

            currentY += 8;

            doc.setFontSize(10);
            const getVal = (key: keyof typeof data, formatFn = (x: any) => x.toString()) => {
                const isPred = data[key] === null;
                const val = isPred ? result.imputed_data[key] : data[key];
                return { text: formatFn(val), isPred };
            };
            
            const getGenderVal = () => {
                const isPred = data.gender === null;
                const val = isPred ? (result.imputed_data.gender === 1 ? 'Male' : 'Female') : data.gender;
                return { text: val, isPred };
            };

            const tData: any[] = [
                ['Age (Years)', getVal('age'), 'Total Bilirubin', getVal('tot_bilirubin', (x: any) => x.toFixed(2))],
                ['Gender', getGenderVal(), 'Direct Bilirubin', getVal('direct_bilirubin', (x: any) => x.toFixed(2))],
                ['Alk. Phosphatase', getVal('alkphos', (x: any) => x.toFixed(1)), 'SGPT (ALT)', getVal('sgpt', (x: any) => x.toFixed(1))],
                ['SGOT (AST)', getVal('sgot', (x: any) => x.toFixed(1)), 'Total Proteins', getVal('tot_proteins', (x: any) => x.toFixed(2))],
                ['Albumin', getVal('albumin', (x: any) => x.toFixed(2)), 'A/G Ratio', getVal('ag_ratio', (x: any) => x.toFixed(2))]
            ];

            let rowY = currentY;
            tData.forEach((row) => {
                const [label1, valObj1, label2, valObj2] = row;
                
                doc.setTextColor(textMuted);
                doc.setFont('helvetica', 'normal');
                doc.text(label1, marginX + 6, rowY);
                
                doc.setTextColor(textDark);
                doc.setFont('helvetica', 'bold');
                doc.text(valObj1.text, marginX + 45, rowY);
                if (valObj1.isPred) {
                    doc.setTextColor('#a855f7');
                    doc.setFontSize(7);
                    doc.text('PREDICTED', marginX + 45, rowY + 3);
                    doc.setFontSize(10);
                }
                
                doc.setTextColor(textMuted);
                doc.setFont('helvetica', 'normal');
                doc.text(label2, marginX + 90, rowY);
                
                doc.setTextColor(textDark);
                doc.setFont('helvetica', 'bold');
                doc.text(valObj2.text, marginX + 130, rowY);
                if (valObj2.isPred) {
                    doc.setTextColor('#a855f7');
                    doc.setFontSize(7);
                    doc.text('PREDICTED', marginX + 130, rowY + 3);
                    doc.setFontSize(10);
                }
                rowY += 8;
            });

            currentY = rowY + 12;

            // --- DIAGNOSTIC RESULT ---
            doc.setTextColor(primaryBlue);
            doc.setFont('helvetica', 'bold');
            doc.setFontSize(14);
            doc.text('Primary AI Diagnosis', marginX, currentY);
            currentY += 8;
            
            const isHealthy = result.prediction_code === 0;
            const resultColor = isHealthy ? '#059669' : '#dc2626'; // Green or Red
            const resultBg = isHealthy ? '#ecfdf5' : '#fef2f2';

            doc.setFillColor(resultBg);
            doc.setDrawColor(resultColor);
            doc.setLineWidth(0.5);
            doc.roundedRect(marginX, currentY, pageWidth - 40, 38, 4, 4, 'FD');

            doc.setTextColor(resultColor);
            doc.setFont('helvetica', 'bold');
            doc.setFontSize(16);
            doc.text(`Stage: ${result.prediction_text}`, marginX + 6, currentY + 12);
            
            doc.setFontSize(26);
            doc.setTextColor(textDark);
            doc.text(`${(result.confidence_score * 100).toFixed(1)}% Confidence`, marginX + 6, currentY + 28);

            currentY += 50;

            // Diagnostic Distribution table
            doc.setTextColor(primaryBlue);
            doc.setFont('helvetica', 'bold');
            doc.setFontSize(14);
            doc.text('Diagnostic Probability Distribution', marginX, currentY);
            currentY += 8;
            
            doc.setFillColor(cardBg);
            doc.setDrawColor(sectionBorder);
            doc.roundedRect(marginX, currentY, pageWidth - 40, 48, 4, 4, 'FD');
            currentY += 8;

            doc.setFontSize(10);
            const stages = ['Healthy Liver', 'Stage 1: Inflammation', 'Stage 2: Significant', 'Stage 3: Severe', 'Stage 4: Critical'];
            
            stages.forEach((stage, i) => {
                const conf = (result.stage_confidence[i] * 100).toFixed(1) + '%';
                if (i === result.prediction_code) {
                    doc.setFont('helvetica', 'bold');
                    doc.setTextColor(resultColor);
                } else {
                    doc.setFont('helvetica', 'normal');
                    doc.setTextColor(textDark);
                }
                doc.text(stage, marginX + 6, currentY);
                doc.text(conf, marginX + 65, currentY);
                currentY += 8;
            });

            currentY += 14;

            // --- SHAP PLOT & XAI ---
            if (currentY > 180) {
               doc.addPage();
               currentY = 20;
            }

            doc.setTextColor(primaryBlue);
            doc.setFont('helvetica', 'bold');
            doc.setFontSize(14);
            doc.text('Deep Medical Insight (CatBoost SHAP)', marginX, currentY);
            currentY += 6;
            
            if (result.text_explanation) {
                doc.setTextColor(textDark);
                doc.setFont('helvetica', 'italic');
                doc.setFontSize(10);
                // Strip markdown bold asterisks for PDF
                const cleanText = result.text_explanation.replace(/\*\*/g, '');
                const explanationLines = doc.splitTextToSize(cleanText, pageWidth - 40);
                doc.text(explanationLines, marginX, currentY);
                currentY += (explanationLines.length * 5) + 4;
            } else {
                doc.setFont('helvetica', 'normal');
                doc.setFontSize(9);
                doc.setTextColor(textMuted);
                doc.text('Explanation of factors contributing to the AI diagnosis. Red increases risk, blue decreases.', marginX, currentY);
                currentY += 8;
            }

            if (result.shap_plot_base64) {
                // Better aspect ratio calculation to prevent squishing
                const imgWidth = pageWidth - 40;
                const imgHeight = imgWidth / 1.6; // Increased height significantly for crisp rendering
                
                doc.setFillColor('#ffffff');
                doc.setDrawColor(sectionBorder);
                doc.roundedRect(marginX, currentY, imgWidth, imgHeight, 2, 2, 'FD');
                
                // Add image with no compression
                doc.addImage(result.shap_plot_base64, 'PNG', marginX + 2, currentY + 2, imgWidth - 4, imgHeight - 4, undefined, 'NONE');
            }

            // --- FOOTER ---
            const pageCount = doc.getNumberOfPages();
            for(let i = 1; i <= pageCount; i++) {
                doc.setPage(i);
                doc.setFontSize(8);
                doc.setTextColor('#94a3b8');
                doc.text('This report is generated by SENTINEL AI V2. It is for informational purposes only.', marginX, 285);
                doc.text('It does not replace professional medical advice. Always consult a physician.', marginX, 290);
                doc.text(`Page ${i} of ${pageCount}`, pageWidth - marginX, 290, { align: 'right' });
            }
            doc.save(`SENTINEL_Report_${data.age}_${data.gender}_${new Date().toISOString().slice(0,10)}.pdf`);

        } catch (error) {
            console.error("PDF generation failed", error);
        } finally {
            isGenerating = false;
        }
    }
</script>

<button 
    onclick={generatePDF}
    disabled={isGenerating || !result}
    class="w-full bg-[#1e2430] hover:bg-[#131720] text-gray-300 hover:text-white font-bold py-5 px-6 rounded-xl border-2 border-gray-700 hover:border-[#ff4b4b] shadow-[0_5px_15px_rgba(0,0,0,0.3)] hover:shadow-[0_0_20px_rgba(255,75,75,0.2)] transition-all duration-300 flex justify-center items-center group relative overflow-hidden disabled:opacity-50 disabled:cursor-not-allowed"
>
    {#if isGenerating}
        <svg class="animate-spin -ml-1 mr-3 h-5 w-5 text-gray-500" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
        </svg>
        <span>GENERATING REPORT...</span>
    {:else}
        <div class="absolute inset-0 w-full h-full bg-gradient-to-r from-transparent via-white/5 to-transparent -translate-x-full group-hover:animate-[shimmer_1.5s_infinite]"></div>
        <svg class="w-6 h-6 mr-3 text-gray-500 group-hover:text-[#ff4b4b] transition-colors duration-300" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 10v6m0 0l-3-3m3 3l3-3m2 8H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"></path></svg>
        <span class="tracking-wide text-sm sm:text-base">DOWNLOAD PDF MEDICAL REPORT</span>
    {/if}
</button>
