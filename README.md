<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8">
  <title>Payoff Simulator Avanzato</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body { font-family: Arial; margin: 20px; background: #f4f4f4; }
    h1 { text-align: center; }
    .option-form { margin: 10px 0; display: flex; gap: 10px; flex-wrap: wrap; }
    .option-form input, .option-form select, button {
      padding: 5px; font-size: 14px;
    }
    #chart-container { max-width: 800px; margin: auto; }
    .options-list { margin: 20px 0; }
  </style>
</head>
<body>

  <h1>Simulatore Payoff Multistrategia</h1>

  <div class="option-form">
    <select id="tipo">
      <option value="call">Call</option>
      <option value="put">Put</option>
    </select>
    <select id="posizione">
      <option value="long">Long</option>
      <option value="short">Short</option>
    </select>
    <input type="number" id="strike" placeholder="Strike" />
    <input type="number" id="premio" placeholder="Premio" />
    <input type="number" id="quantita" placeholder="Quantità" />
    <button onclick="aggiungiOpzione()">Aggiungi Opzione</button>
  </div>

  <div class="options-list" id="listaOpzioni"></div>

  <div id="chart-container">
    <canvas id="graficoPayoff" height="400"></canvas>
  </div>

  <script>
    const opzioni = [];
    const rangePrezzi = Array.from({ length: 101 }, (_, i) => i + 50); // da 50 a 150

    function aggiungiOpzione() {
      const tipo = document.getElementById("tipo").value;
      const posizione = document.getElementById("posizione").value;
      const strike = parseFloat(document.getElementById("strike").value);
      const premio = parseFloat(document.getElementById("premio").value);
      const quantita = parseInt(document.getElementById("quantita").value);

      if (isNaN(strike) || isNaN(premio) || isNaN(quantita)) {
        alert("Compila tutti i campi!");
        return;
      }

      opzioni.push({ tipo, posizione, strike, premio, quantita });
      aggiornaLista();
      disegnaGrafico();
    }

    function calcolaPayoff(opzione) {
      return rangePrezzi.map(prezzo => {
        let base = 0;
        if (opzione.tipo === "call") {
          base = Math.max(0, prezzo - opzione.strike) - opzione.premio;
        } else {
          base = Math.max(0, opzione.strike - prezzo) - opzione.premio;
        }
        if (opzione.posizione === "short") base = -base;
        return base * opzione.quantita;
      });
    }

    function sommaTotalePayoff() {
      const totale = Array(rangePrezzi.length).fill(0);
      opzioni.forEach(opt => {
        const payoff = calcolaPayoff(opt);
        payoff.forEach((val, i) => totale[i] += val);
      });
      return totale;
    }

    function aggiornaLista() {
      const lista = document.getElementById("listaOpzioni");
      lista.innerHTML = "<strong>Opzioni Aggiunte:</strong><br>" + opzioni.map((opt, i) =>
        `${i + 1}. ${opt.posizione.toUpperCase()} ${opt.tipo.toUpperCase()} | Strike: ${opt.strike}, Premio: ${opt.premio}, Q: ${opt.quantita}`
      ).join("<br>");
    }

    let chart = null;

    function disegnaGrafico() {
      const datasets = opzioni.map((opt, idx) => ({
        label: `${opt.posizione.toUpperCase()} ${opt.tipo.toUpperCase()} ${idx + 1}`,
        data: calcolaPayoff(opt),
        borderColor: opt.tipo === "call" ? "blue" : "red",
        fill: false,
        borderWidth: 1
      }));

      datasets.push({
        label: "Totale Strategia",
        data: sommaTotalePayoff(),
        borderColor: "black",
        borderWidth: 3,
        fill: false
      });

      const ctx = document.getElementById("graficoPayoff").getContext("2d");
      if (chart) chart.destroy();

      chart = new Chart(ctx, {
        type: "line",
        data: {
          labels: rangePrezzi,
          datasets: datasets
        },
        options: {
          scales: {
            x: { title: { display: true, text: "Prezzo del Sottostante" } },
            y: { title: { display: true, text: "Payoff Totale" } }
          }
        }
      });
    }
  </script>

</body>
</html>

