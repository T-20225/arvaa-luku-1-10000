
<html lang="fi">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Arvaa luku (1–10 000)</title>
  <style>
    body {
      font-family: sans-serif;
      background: #eef2f7;
      text-align: center;
      padding: 50px;
    }
    .game-container {
      background: white;
      padding: 30px;
      border-radius: 10px;
      display: inline-block;
      box-shadow: 0 4px 12px rgba(0,0,0,0.1);
      max-width: 400px;
    }
    input[type="number"] {
      padding: 10px;
      font-size: 16px;
      width: 120px;
    }
    button {
      padding: 10px 20px;
      margin: 5px;
      font-size: 16px;
      cursor: pointer;
    }
    #result, #record, #stats, #topScores {
      margin-top: 15px;
      font-size: 16px;
    }
    hr {
      margin: 20px 0;
    }
  </style>
</head>
<body>
  <div class="game-container">
    <h1>Arvaa luku (1–10 000)</h1>
    <input type="number" id="guessInput" min="1" max="10000" />
    <br />
    <button onclick="checkGuess()">Arvaa</button>
    <button onclick="resetGame()">Uudestaan</button>
    <button onclick="resetStats()">Nollaa tilastot</button>
    <p id="result"></p>
    <p id="record"></p>
    <div id="stats"></div>
    <div id="topScores"></div>
  </div>

  <audio id="soundWin" src="https://www.soundjay.com/buttons/sounds/button-4.mp3"></audio>
  <audio id="soundFail" src="https://www.soundjay.com/buttons/sounds/button-10.mp3"></audio>

  <script>
    let targetNumber = Math.floor(Math.random() * 10000) + 1;
    let hasGuessed = false;
    let startTime = Date.now();

    function checkGuess() {
      if (hasGuessed) {
        document.getElementById('result').textContent = "Olet jo arvannut! Paina 'Uudestaan'.";
        return;
      }

      const guess = Number(document.getElementById('guessInput').value);
      const result = document.getElementById('result');

      if (!guess || guess < 1 || guess > 10000) {
        result.textContent = "Syötä luku väliltä 1–10 000!";
        result.style.color = "orange";
        return;
      }

      const diff = Math.abs(targetNumber - guess);
      const duration = Math.round((Date.now() - startTime) / 1000); // sekunteina
      hasGuessed = true;

      if (guess === targetNumber) {
        result.textContent = `Täysin oikein! Luku oli ${targetNumber}. 🎯 (${duration}s)`;
        result.style.color = "green";
        document.getElementById("soundWin").play();
      } else {
        const suunta = guess > targetNumber ? "liian suuri" : "liian pieni";
        result.textContent = `Väärin! Arvauksesi oli ${diff} ${suunta}. (${duration}s)`;
        result.style.color = "red";
        document.getElementById("soundFail").play();
      }

      updateRecord(diff);
      updateStats(diff, guess === targetNumber, duration);
      updateTopScores(diff);
    }

    function updateRecord(newDiff) {
      const recordElement = document.getElementById('record');
      const oldRecord = localStorage.getItem("parasEro");

      if (oldRecord === null || newDiff < Number(oldRecord)) {
        localStorage.setItem("parasEro", newDiff);
        recordElement.textContent = `Uusi ennätys: vain ${newDiff} harhaan! 🎉`;
        recordElement.style.color = "blue";
      } else {
        recordElement.textContent = `Paras tulos: ${oldRecord} harhaan`;
        recordElement.style.color = "gray";
      }
    }

    function updateStats(diff, oikein, aika) {
      let pelit = Number(localStorage.getItem("pelit") || 0);
      let voitot = Number(localStorage.getItem("voitot") || 0);
      let summa = Number(localStorage.getItem("virhesumma") || 0);
      let ajat = Number(localStorage.getItem("aikasumma") || 0);

      pelit += 1;
      if (oikein) voitot += 1;
      summa += diff;
      ajat += aika;

      localStorage.setItem("pelit", pelit);
      localStorage.setItem("voitot", voitot);
      localStorage.setItem("virhesumma", summa);
      localStorage.setItem("aikasumma", ajat);

      showStats();
    }

    function updateTopScores(diff) {
      let top = JSON.parse(localStorage.getItem("top5") || "[]");
      top.push(diff);
      top.sort((a, b) => a - b);
      if (top.length > 5) top = top.slice(0, 5);
      localStorage.setItem("top5", JSON.stringify(top));
      showTopScores();
    }

    function showStats() {
      const pelit = Number(localStorage.getItem("pelit") || 0);
      const voitot = Number(localStorage.getItem("voitot") || 0);
      const summa = Number(localStorage.getItem("virhesumma") || 0);
      const ajat = Number(localStorage.getItem("aikasumma") || 0);
      const paras = localStorage.getItem("parasEro");

      const statsText = `
        <hr>
        <strong>Tilastot:</strong><br>
        Pelejä pelattu: ${pelit}<br>
        Oikein arvattu: ${voitot}<br>
        Keskimääräinen virhe: ${pelit > 0 ? Math.round(summa / pelit) : 0}<br>
        Keskimääräinen aika: ${pelit > 0 ? Math.round(ajat / pelit) : 0} s<br>
        Paras tulos: ${paras || "–"} harhaan
      `;

      document.getElementById("stats").innerHTML = statsText;
    }

    function showTopScores() {
      const top = JSON.parse(localStorage.getItem("top5") || "[]");
      let html = "<hr><strong>Top 5 -tulokset:</strong><br>";
      top.forEach((val, i) => {
        html += `${i + 1}. ${val} harhaan<br>`;
      });
      document.getElementById("topScores").innerHTML = html;
    }

    function resetGame() {
      targetNumber = Math.floor(Math.random() * 10000) + 1;
      hasGuessed = false;
      startTime = Date.now();
      document.getElementById('guessInput').value = '';
      document.getElementById('result').textContent = '';
      document.getElementById('record').textContent = '';
    }

    function resetStats() {
      if (confirm("Haluatko varmasti nollata kaikki tilastot?")) {
        localStorage.clear();
        document.getElementById("record").textContent = '';
        document.getElementById("stats").textContent = '';
        document.getElementById("topScores").textContent = '';
      }
    }

    window.onload = () => {
      const oldRecord = localStorage.getItem("parasEro");
      if (oldRecord) {
        document.getElementById('record').textContent = `Paras tulos: ${oldRecord} harhaan`;
        document.getElementById('record').style.color = "gray";
      }
      showStats();
      showTopScores();
    }
  </script>
</body>
</html>
