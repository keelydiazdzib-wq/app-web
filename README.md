<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Probabilidad y Estadística Pro</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        .card { background: white; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); padding: 20px; margin-bottom: 20px; }
        canvas { max-height: 400px; }
    </style>
</head>
<body class="bg-gray-100 font-sans text-gray-800">

    <nav class="bg-blue-600 p-4 text-white shadow-lg mb-8">
        <div class="container mx-auto flex justify-between items-center">
            <h1 class="text-2xl font-bold">Estadística Interactiva</h1>
            <p class="text-sm">Proyecto Final - Probabilidad</p>
        </div>
    </nav>

    <main class="container mx-auto px-4 pb-12">
        
        <section class="grid md:grid-cols-2 gap-6">
            <div class="card">
                <h2 class="text-xl font-bold mb-4">1. Ingreso de Datos</h2>
                <textarea id="dataInput" class="w-full p-3 border rounded mb-4" rows="4" placeholder="Ej: 10, 15, 20, 10, 5..."></textarea>
                <div class="flex gap-2">
                    <button onclick="processManualData()" class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600 transition">Procesar Manual</button>
                    <button onclick="generateRandomData()" class="bg-green-500 text-white px-4 py-2 rounded hover:bg-green-600 transition">Datos Aleatorios (20+)</button>
                </div>
            </div>

            <div class="card">
                <h2 class="text-xl font-bold mb-4">Resultados Descriptivos</h2>
                <div id="statsResults" class="grid grid-cols-2 gap-4 text-sm">
                    <p>Media: <span id="resMedia" class="font-bold">-</span></p>
                    <p>Mediana: <span id="resMediana" class="font-bold">-</span></p>
                    <p>Moda: <span id="resModa" class="font-bold">-</span></p>
                    <p>Rango: <span id="resRango" class="font-bold">-</span></p>
                    <p>Mín: <span id="resMin" class="font-bold">-</span></p>
                    <p>Máx: <span id="resMax" class="font-bold">-</span></p>
                </div>
            </div>
        </section>

        <section class="card overflow-x-auto">
            <h2 class="text-xl font-bold mb-4">2. Tabla de Frecuencias</h2>
            <table class="w-full text-left border-collapse">
                <thead>
                    <tr class="bg-gray-200">
                        <th class="p-2 border">Dato (x)</th>
                        <th class="p-2 border">fi (Absoluta)</th>
                        <th class="p-2 border">fr (Relativa)</th>
                        <th class="p-2 border">Fi (Acumulada)</th>
                        <th class="p-2 border">Fr (Relativa Acum.)</th>
                    </tr>
                </thead>
                <tbody id="freqTableBodyContent"></tbody>
            </table>
        </section>

        <section class="grid md:grid-cols-2 gap-6">
            <div class="card">
                <h2 class="text-xl font-bold mb-4">Histograma y Polígono</h2>
                <canvas id="histChart"></canvas>
            </div>
            <div class="card">
                <h2 class="text-xl font-bold mb-4">Ojiva (Frecuencia Acumulada)</h2>
                <canvas id="ojivaChart"></canvas>
            </div>
            <div class="card md:col-span-2">
                <h2 class="text-xl font-bold mb-4">Diagrama de Pareto</h2>
                <canvas id="paretoChart"></canvas>
            </div>
        </section>

        <section class="grid md:grid-cols-2 gap-6">
            <div class="card">
                <h2 class="text-xl font-bold mb-4">Combinatoria y Probabilidad</h2>
                <div class="space-y-4">
                    <div>
                        <label class="block text-sm">n (Total items):</label>
                        <input type="number" id="inputN" class="border p-1 w-full" value="5">
                    </div>
                    <div>
                        <label class="block text-sm">r (Seleccionados):</label>
                        <input type="number" id="inputR" class="border p-1 w-full" value="2">
                    </div>
                    <button onclick="calcCombinatoria()" class="bg-purple-600 text-white px-4 py-2 rounded w-full">Calcular Permutación/Combinación</button>
                    <div class="mt-4 p-3 bg-purple-50 rounded border border-purple-200">
                        <p>Permutaciones (nPr): <span id="resPerm" class="font-bold">0</span></p>
                        <p>Combinaciones (nCr): <span id="resComb" class="font-bold">0</span></p>
                    </div>
                </div>
            </div>

            <div class="card">
                <h2 class="text-xl font-bold mb-4">Conjuntos y Árbol</h2>
                <p class="text-sm mb-2 text-gray-600 italic">Simulación de Regla Multiplicativa</p>
                <div class="space-y-2">
                    <p>Evento A: <input type="number" id="probA" class="border p-1 w-16" value="0.5" step="0.1"> P(A)</p>
                    <p>Evento B: <input type="number" id="probB" class="border p-1 w-16" value="0.3" step="0.1"> P(B)</p>
                    <button onclick="calcConjuntos()" class="bg-indigo-500 text-white px-4 py-2 rounded">Calcular Intersección</button>
                    <div class="mt-4" id="conjuntoResult">
                        <p>P(A ∩ B) = <span id="resInter" class="font-bold">0.15</span> (Independientes)</p>
                    </div>
                </div>
            </div>
        </section>

    </main>

    <script>
        let charts = {};

        function generateRandomData() {
            const data = Array.from({length: 25}, () => Math.floor(Math.random() * 50) + 1);
            document.getElementById('dataInput').value = data.join(', ');
            processManualData();
        }

        function processManualData() {
            const input = document.getElementById('dataInput').value;
            let rawData = input.split(',').map(n => parseFloat(n.trim())).filter(n => !isNaN(n));
            
            if (rawData.length < 5) return alert("Por favor ingresa más datos.");

            rawData.sort((a, b) => a - b);
            calculateStats(rawData);
            renderTableAndCharts(rawData);
        }

        function calculateStats(data) {
            const n = data.length;
            const sum = data.reduce((a, b) => a + b, 0);
            const media = sum / n;
            
            const mid = Math.floor(n / 2);
            const mediana = n % 2 !== 0 ? data[mid] : (data[mid - 1] + data[mid]) / 2;

            const counts = {};
            data.forEach(x => counts[x] = (counts[x] || 0) + 1);
            let maxFreq = 0;
            let moda = [];
            for(let key in counts) {
                if(counts[key] > maxFreq) { maxFreq = counts[key]; moda = [key]; }
                else if(counts[key] === maxFreq) { moda.push(key); }
            }

            document.getElementById('resMedia').innerText = media.toFixed(2);
            document.getElementById('resMediana').innerText = mediana;
            document.getElementById('resModa').innerText = moda.join(', ');
            document.getElementById('resMin').innerText = data[0];
            document.getElementById('resMax').innerText = data[n-1];
            document.getElementById('resRango').innerText = data[n-1] - data[0];
        }

        function renderTableAndCharts(data) {
            const counts = {};
            data.forEach(x => counts[x] = (counts[x] || 0) + 1);
            const uniqueValues = Object.keys(counts).sort((a,b) => a-b);
            const n = data.length;

            let tableHTML = "";
            let Fi = 0;
            let labels = [], fiValues = [], FiValues = [], frValues = [];

            uniqueValues.forEach(val => {
                let fi = counts[val];
                let fr = fi / n;
                Fi += fi;
                let Fr = Fi / n;
                
                labels.push(val);
                fiValues.push(fi);
                FiValues.push(Fi);
                frValues.push(fr * 100);

                tableHTML += `<tr>
                    <td class="p-2 border">${val}</td>
                    <td class="p-2 border">${fi}</td>
                    <td class="p-2 border">${fr.toFixed(4)}</td>
                    <td class="p-2 border">${Fi}</td>
                    <td class="p-2 border">${Fr.toFixed(4)}</td>
                </tr>`;
            });
            document.getElementById('freqTableBodyContent').innerHTML = tableHTML;

            // Gráficos
            updateChart('histChart', 'bar', labels, fiValues, 'Histograma', true);
            updateChart('ojivaChart', 'line', labels, FiValues, 'Ojiva (Acumulada)', false);
            renderPareto(labels, fiValues);
        }

        function updateChart(id, type, labels, data, title, showLine) {
            if (charts[id]) charts[id].destroy();
            const datasets = [{
                label: title,
                data: data,
                backgroundColor: 'rgba(54, 162, 235, 0.5)',
                borderColor: 'rgba(54, 162, 235, 1)',
                borderWidth: 1,
                tension: 0.4
            }];

            if(showLine) {
                datasets.push({
                    label: 'Polígono de Frecuencia',
                    data: data,
                    type: 'line',
                    borderColor: '#ff6384',
                    fill: false
                });
            }

            charts[id] = new Chart(document.getElementById(id), {
                type: type,
                data: { labels, datasets }
            });
        }

        function renderPareto(labels, fi) {
            if (charts['paretoChart']) charts['paretoChart'].destroy();
            
            // Ordenar para Pareto
            let combined = labels.map((l, i) => ({l, f: fi[i]})).sort((a,b) => b.f - a.f);
            let sortedLabels = combined.map(o => o.l);
            let sortedFi = combined.map(o => o.f);
            let total = sortedFi.reduce((a,b) => a+b, 0);
            let cumulative = 0;
            let percentLine = sortedFi.map(f => {
                cumulative += f;
                return (cumulative / total * 100).toFixed(2);
            });

            charts['paretoChart'] = new Chart(document.getElementById('paretoChart'), {
                type: 'bar',
                data: {
                    labels: sortedLabels,
                    datasets: [
                        { label: 'Frecuencia', data: sortedFi, backgroundColor: '#3b82f6', yAxisID: 'y' },
                        { label: '% Acumulado', data: percentLine, type: 'line', borderColor: '#ef4444', yAxisID: 'y1' }
                    ]
                },
                options: {
                    scales: {
                        y: { beginAtZero: true, position: 'left' },
                        y1: { beginAtZero: true, max: 100, position: 'right', grid: { drawOnChartArea: false } }
                    }
                }
            });
        }

        // Combinatoria
        function factorial(n) {
            return (n <= 1) ? 1 : n * factorial(n - 1);
        }

        function calcCombinatoria() {
            const n = parseInt(document.getElementById('inputN').value);
            const r = parseInt(document.getElementById('inputR').value);
            if (r > n) return alert("r no puede ser mayor que n");

            const nPr = factorial(n) / factorial(n - r);
            const nCr = nPr / factorial(r);

            document.getElementById('resPerm').innerText = nPr.toLocaleString();
            document.getElementById('resComb').innerText = nCr.toLocaleString();
        }

        function calcConjuntos() {
            const a = parseFloat(document.getElementById('probA').value);
            const b = parseFloat(document.getElementById('probB').value);
            document.getElementById('resInter').innerText = (a * b).toFixed(4);
        }
    </script>
</body>
</html>
