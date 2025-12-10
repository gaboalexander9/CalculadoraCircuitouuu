<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calculadora de Circuito RL</title>
    <style>
        body { background: white; font-family: Arial, sans-serif; margin: 0; padding: 20px; color: #333; line-height: 1.7; }
        h1 { text-align: center; color: #007BFF; font-size: 2.5em; margin: 20px 0; }
        .subtitle { text-align: center; font-size: 1.2em; color: #555; margin-bottom: 40px; }
        .container { max-width: 1000px; margin: 0 auto; }
        .input-group { margin: 25px 0; display: flex; align-items: center; flex-wrap: wrap; gap: 20px; }
        label { font-weight: bold; width: 300px; font-size: 1.1em; }
        input { padding: 12px; width: 220px; border: 2px solid #007BFF; border-radius: 8px; font-size: 1.1em; }
        .preview { padding: 18px; background: #f0f8ff; border-radius: 12px; min-height: 80px; flex: 1; font-size: 1.05em; box-shadow: 0 2px 8px rgba(0,123,255,0.1); }
        button { background: #007BFF; color: white; padding: 16px 50px; border: none; border-radius: 10px; cursor: pointer; font-size: 1.4em; display: block; margin: 40px auto; box-shadow: 0 4px 15px rgba(0,123,255,0.3); }
        button:hover { background: #0056b3; transform: translateY(-2px); }
        #results { margin-top: 60px; display: none; background: #f8fbff; padding: 30px; border-radius: 15px; box-shadow: 0 5px 20px rgba(0,0,0,0.1); }
        .graph { width: 100%; height: 500px; margin: 40px 0; background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
        .formula-box { background: #e3f2fd; padding: 20px; border-left: 6px solid #007BFF; margin: 20px 0; border-radius: 0 10px 10px 0; font-size: 1.1em; }
        .pasos { background: #fff8e1; padding: 25px; border-radius: 12px; margin: 40px 0; border: 2px solid #ffca28; }
        table { width: 100%; border-collapse: collapse; margin: 30px 0; font-size: 1em; }
        th, td { border: 1px solid #90caf9; padding: 12px; text-align: center; }
        th { background: #e3f2fd; font-weight: bold; }
        .banda { display: inline-block; width: 30px; height: 20px; margin: 0 3px; border: 1px solid #ccc; border-radius: 4px; vertical-align: middle; }
        .highlight { background: #ffff99; padding: 10px; border-radius: 8px; margin-bottom: 20px; font-weight: bold; }
        .optional { color: #666; font-style: italic; }
        @media (max-width: 768px) { .input-group { flex-direction: column; align-items: stretch; } .graph { height: 400px; } }
    </style>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
<div class="container">
    <h1>Calculadora de Circuito RL</h1>
    <div class="subtitle">Simulador educativo del decaimiento de corriente en un circuito RL serie<br>
        (Ecuación diferencial de primer orden homogénea)</div>
    <div class="highlight">
        <h3>Magnitudes Eléctricas y Unidades</h3>
        <ul>
            <li><strong>Resistencia (R)</strong>: ohm (Ω) - Valor numérico, permite kΩ con conversión interna.</li>
            <li><strong>Inductancia (L)</strong>: henry (H) - Permite mH o µH con conversión interna a H.</li>
            <li><strong>Corriente inicial (I₀)</strong>: ampere (A) - Corriente en t=0.</li>
            <li><strong>Tiempo máximo (t_max)</strong>: segundos (s) - Intervalo de simulación.</li>
            <li><strong>Paso de tiempo (Δt)</strong>: segundos (s) - Para cálculos y gráfica.</li>
            <li><strong>Corriente objetivo (I_objetivo, opcional)</strong>: ampere (A) - Valor deseado para calcular tiempo.</li>
            <li><strong>Punto experimental (opcional)</strong>: t_medido (s) y I_medido (A) - Para comparación con modelo.</li>
        </ul>
    </div>
    <div class="input-group">
        <label>Resistencia R (ohm o kΩ):</label>
        <input type="number" id="R" step="any" placeholder="Ej: 1399" oninput="updateResistor()">
        <div class="preview" id="resistorColors">Ingresa un valor mayor que 0</div>
    </div>
    <div class="input-group">
        <label>Inductancia L (henry, mH o µH):</label>
        <input type="number" id="L" step="any" placeholder="Ej: 49 para 49 mH" oninput="updateInductor()">
        <div class="preview" id="inductorInfo">Ingresa un valor mayor que 0</div>
    </div>
    <div class="input-group">
        <label>Corriente inicial I₀ (A):</label>
        <input type="number" id="I0" step="any" required>
    </div>
    <div class="input-group">
        <label>Tiempo máximo t_max (s):</label>
        <input type="number" id="tmax" step="any" required>
    </div>
    <div class="input-group">
        <label>Paso de tiempo Δt (s):</label>
        <input type="number" id="dt" step="any" required>
    </div>
    <div class="input-group optional">
        <label>Corriente objetivo I_objetivo (A):</label>
        <input type="number" id="Iobj" step="any">
    </div>
    <div class="input-group optional">
        <label>Punto experimental: t_medido (s) e I_medido (A):</label>
        <input type="number" id="tmed" step="any" placeholder="t_medido">
        <input type="number" id="Imed" step="any" placeholder="I_medido">
    </div>
    <center><button onclick="calcular()">Calcular</button></center>
</div>

<div id="results" class="container">
    <h2 style="color:#007BFF; text-align:center;">Resultados del Circuito RL Serie</h2>
    <div id="inputSummary" style="font-size:1.3em; text-align:center; background:#e3f2fd; padding:20px; border-radius:12px;"></div>
    <div class="formula-box">
        <strong>Constante de tiempo:</strong> τ = L/R = <span id="tauValue"></span> s (tiempo característico de decaimiento de la corriente)<br>
        <strong>Función de corriente:</strong> i(t) = <span id="I0Value"></span> × e<sup>-t/τ</sup> A
    </div>
    <div id="keyResults" class="formula-box">
        <strong>Resultados numéricos clave:</strong><br>
        <span id="keyValues"></span>
    </div>
    <div style="display:flex; flex-wrap:wrap; gap:20px; justify-content:center;">
        <div class="graph"><canvas id="graficaI"></canvas>
            <div class="formula-box">Gráfica de i(t) vs tiempo - Eje x: tiempo [s], Eje y: corriente [A]. Muestra el decaimiento exponencial.</div></div>
        <div class="graph"><canvas id="graficaVR"></canvas>
            <div class="formula-box">Gráfica de v_R(t) = R × i(t) [V] - Voltaje en resistencia.</div></div>
        <div class="graph"><canvas id="graficaVL"></canvas>
            <div class="formula-box">Gráfica de v_L(t) = -R × i(t) [V] - Voltaje en inductancia.</div></div>
    </div>
    <h3 style="color:#007BFF;">Tabla de valores</h3>
    <table id="tabla">
        <thead><tr><th>t (s)</th><th>i(t) (A)</th><th>v_R(t) (V)</th><th>v_L(t) (V)</th></tr></thead>
        <tbody></tbody>
    </table>
    <div class="pasos">
        <h3 style="color:#d81b60; margin-top:0;">Memoria de cálculo: Problema resuelto paso a paso</h3>
        <p>Se resuelve el problema con los valores ingresados, formulando el modelo matemático y calculando resultados.</p>
        <ol>
            <li><strong>Formulación del modelo:</strong><br>
                Ecuación diferencial: L × di/dt + R × i = 0 → di/dt = - (R/L) × i (homogénea de primer orden).</li>
            <li><strong>Solución general:</strong><br>
                i(t) = K × e<sup> - (R/L) × t</sup></li>
            <li><strong>Condición inicial:</strong> i(0) = I₀ → K = I₀</li>
            <li><strong>Solución particular:</strong><br>
                i(t) = I₀ × e<sup> - t / τ</sup>, donde τ = L/R = <span id="tauCalc"></span> s</li>
            <li><strong>Cálculo de voltajes:</strong><br>
                v_R(t) = R × i(t) = <span id="vR0"></span> × e<sup> - t / τ</sup> V<br>
                v_L(t) = - R × i(t) = - <span id="vL0"></span> × e<sup> - t / τ</sup> V</li>
            <li><strong>Ejemplo en t = 0:</strong><br>
                i(0) = <span id="i0Calc"></span> A, v_R(0) = <span id="vR0Calc"></span> V, v_L(0) = <span id="vL0Calc"></span> V</li>
            <li><strong>Ejemplo en t = τ:</strong><br>
                i(τ) ≈ 0,368 × I₀ = <span id="iTau"></span> A</li>
            <li><strong>Tiempo para I_objetivo (si ingresado):</strong><br>
                t_objetivo = - τ × ln(I_objetivo / I₀) = <span id="tObj"></span> s (tiempo para que la corriente descienda a I_objetivo; menor R o mayor L implica más tiempo).</li>
            <li><strong>Comparación experimental (si ingresado):</strong><br>
                i(t_medido) calculado = <span id="iMedCalc"></span> A, error porcentual = <span id="errorPerc"></span>% (diferencia entre medido y teórico).</li>
        </ol>
        <p><strong>Interpretación:</strong> El decaimiento es exponencial. Mayor R acelera el decaimiento (menor τ), mayor L lo retarda. En práctica, afecta conmutación rápida en electrónica o paro suave de motores.</p>
    </div>
</div>

<script>
// Formato chileno: coma y 3 decimales
function formato(num) {
    return num.toFixed(3).replace(".", ",");
}

// Colores reales para resistencias
const colorCodes = {
    0: "#000000", 1: "#8B4513", 2: "#FF0000", 3: "#FFA500", 4: "#FFFF00",
    5: "#008000", 6: "#0000FF", 7: "#EE82EE", 8: "#808080", 9: "#FFFFFF"
};

function updateResistor() {
    let input = document.getElementById("R").value;
    let val = parseFloat(input);
    if (isNaN(val) || val <= 0) { 
        document.getElementById("resistorColors").innerHTML = "Ingresa un valor mayor que 0"; 
        return; 
    }
    if (input.toLowerCase().includes("k")) val *= 1000;

    let strVal = val.toString().replace('.', '');
    let d1 = parseInt(strVal[0] || 0);
    let d2 = parseInt(strVal[1] || 0);
    let mult = strVal.length - 2;
    if (mult < 0) mult = 0;

    document.getElementById("resistorColors").innerHTML = `
        <strong>Resistencia = ${formato(val)} Ω</strong><br>
        Bandas: 
        <span class="banda" style="background:${colorCodes[d1]}"></span>
        <span class="banda" style="background:${colorCodes[d2]}"></span>
        <span class="banda" style="background:${colorCodes[mult]}"></span>
        <span class="banda" style="background:gold"></span>
        → ${d1}${d2} × 10<sup>${mult}</sup> Ω ±5%<br>
        <small>Ejemplo visual de resistencia de 4 bandas</small>`;
}

function updateInductor() {
    let val = parseFloat(document.getElementById("L").value);
    if (isNaN(val) || val <= 0) { 
        document.getElementById("inductorInfo").innerHTML = "Ingresa un valor mayor que 0"; 
        return; 
    }
    
    let Lh = val;
    if (document.getElementById("L").value.toLowerCase().includes("m")) Lh /= 1000;
    if (document.getElementById("L").value.includes("µ")) Lh /= 1e6;

    let vueltas = Math.round(300 * Math.pow(Lh, 0.4));
    let diametro = Lh < 0.01 ? "pequeño (5-10 mm)" : Lh < 0.1 ? "mediano (10-30 mm)" : "grande (mayor que 30 mm)";

    document.getElementById("inductorInfo").innerHTML = `
        <strong>Bobina = ${formato(Lh)} H</strong><br>
        Estimación didáctica:<br>
        • Aproximadamente ${vueltas} vueltas de alambre<br>
        • Diámetro aproximado: ${diametro}<br>
        <small>Valores típicos en bobinas reales</small>`;
}

function calcular() {
    let R = parseFloat(document.getElementById("R").value) || 0;
    if (document.getElementById("R").value.toLowerCase().includes("k")) R *= 1000;

    let L = parseFloat(document.getElementById("L").value) || 0;
    if (document.getElementById("L").value.toLowerCase().includes("m")) L /= 1000;
    if (document.getElementById("L").value.includes("µ")) L /= 1e6;

    const I0 = parseFloat(document.getElementById("I0").value);
    const tmax = parseFloat(document.getElementById("tmax").value);
    const dt = parseFloat(document.getElementById("dt").value);
    const Iobj = parseFloat(document.getElementById("Iobj").value) || NaN;
    const tmed = parseFloat(document.getElementById("tmed").value) || NaN;
    const Imed = parseFloat(document.getElementById("Imed").value) || NaN;

    if (R <= 0 || L <= 0 || I0 <= 0 || tmax <= 0 || dt <= 0) { 
        alert("Los valores de R, L, I₀, t_max y Δt deben ser positivos y distintos de cero."); 
        return; 
    }
    if (!isNaN(Iobj) && (Iobj >= I0 || Iobj <= 0)) { 
        alert("I_objetivo debe ser positivo y menor que I₀ en decaimiento."); 
        return; 
    }
    if ((!isNaN(tmed) && isNaN(Imed)) || (isNaN(tmed) && !isNaN(Imed))) { 
        alert("Para comparación experimental, ingrese ambos t_medido e I_medido."); 
        return; 
    }
    if (!isNaN(tmed) && tmed < 0) { 
        alert("t_medido debe ser positivo."); 
        return; 
    }

    const tau = L / R;
    const VR0 = I0 * R;
    const iTau = I0 * Math.exp(-1);

    // Resumen de entrada
    let summary = `<strong>R = ${formato(R)} Ω, L = ${formato(L)} H, I₀ = ${formato(I0)} A, t_max = ${formato(tmax)} s, Δt = ${formato(dt)} s</strong>`;
    if (!isNaN(Iobj)) summary += `, I_objetivo = ${formato(Iobj)} A`;
    if (!isNaN(tmed)) summary += `, t_medido = ${formato(tmed)} s, I_medido = ${formato(Imed)} A`;
    document.getElementById("inputSummary").innerHTML = summary;

    document.getElementById("tauValue").innerText = formato(tau);
    document.getElementById("I0Value").innerText = formato(I0);
    document.getElementById("tauCalc").innerText = formato(tau);
    document.getElementById("vR0").innerText = formato(VR0);
    document.getElementById("vL0").innerText = formato(VR0);
    document.getElementById("i0Calc").innerText = formato(I0);
    document.getElementById("vR0Calc").innerText = formato(VR0);
    document.getElementById("vL0Calc").innerText = formato(-VR0);
    document.getElementById("iTau").innerText = formato(iTau);

    // Resultados clave
    let keyValues = `I(0) = ${formato(I0)} A<br>I(t_max) = ${formato(I0 * Math.exp(-tmax / tau))} A`;
    if (!isNaN(Iobj)) {
        const tObj = -tau * Math.log(Iobj / I0);
        keyValues += `<br>t para I_objetivo = ${formato(tObj)} s`;
        document.getElementById("tObj").innerText = formato(tObj);
    } else {
        document.getElementById("tObj").innerText = "No ingresado";
    }
    if (!isNaN(tmed)) {
        const iMedCalc = I0 * Math.exp(-tmed / tau);
        const error = Math.abs((Imed - iMedCalc) / iMedCalc) * 100;
        keyValues += `<br>I(t_medido) calculado = ${formato(iMedCalc)} A<br>Error porcentual = ${formato(error)} %`;
        document.getElementById("iMedCalc").innerText = formato(iMedCalc);
        document.getElementById("errorPerc").innerText = formato(error);
    } else {
        document.getElementById("iMedCalc").innerText = "No ingresado";
        document.getElementById("errorPerc").innerText = "No ingresado";
    }
    document.getElementById("keyValues").innerHTML = keyValues;

    // Datos para tabla y gráficas
    const tiempos = [], corrientes = [], vR = [], vL = [];
    for (let t = 0; t <= tmax; t += dt) {
        tiempos.push(t.toFixed(3));
        const i = I0 * Math.exp(-t / tau);
        corrientes.push(i);
        vR.push(i * R);
        vL.push(-i * R);
    }

    // Tabla
    let tbody = "";
    tiempos.forEach((t, i) => {
        tbody += `<tr><td>${t}</td><td>${formato(corrientes[i])}</td><td>${formato(vR[i])}</td><td>${formato(vL[i])}</td></tr>`;
    });
    document.querySelector("#tabla tbody").innerHTML = tbody;

    // Gráficas
    crearGrafica("graficaI", tiempos, corrientes, "Corriente i(t) [A]", "#007BFF");
    crearGrafica("graficaVR", tiempos, vR, "Voltaje v_R(t) [V]", "#D32F2F");
    crearGrafica("graficaVL", tiempos, vL, "Voltaje v_L(t) [V]", "#D32F2F");

    document.getElementById("results").style.display = "block";
    window.scrollTo(0, document.getElementById("results").offsetTop);
}

function crearGrafica(id, labels, data, titulo, color) {
    new Chart(document.getElementById(id), {
        type: 'line',
        data: { 
            labels: labels, 
            datasets: [{ 
                label: titulo, 
                data: data.map(d => Number(d.toFixed(3))), 
                borderColor: color, 
                backgroundColor: color + '33',
                fill: true,
                tension: 0.4,
                pointRadius: 3
            }] 
        },
        options: { 
            responsive: true,
            maintainAspectRatio: false,
            scales: { 
                x: { title: { display: true, text: 'Tiempo (s)', font: { size: 14 }}},
                y: { title: { display: true, text: titulo.includes('Corriente') ? 'Corriente (A)' : 'Voltaje (V)', font: { size: 14 }}}
            },
            plugins: { legend: { labels: { font: { size: 14 }}} }
        }
    });
}
</script>
</body>
</html>
