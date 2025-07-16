<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Bingo Stream</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #121212;
      color: white;
      padding: 20px;
      text-align: center;
    }
    .tabs {
      display: flex;
      justify-content: center;
      margin-bottom: 20px;
      flex-wrap: wrap;
    }
    .tab {
      padding: 10px 20px;
      background-color: #333;
      margin: 5px;
      cursor: pointer;
      border-radius: 5px 5px 0 0;
    }
    .tab.active {
      background-color: #ff4081;
    }
    .tab-content {
      display: none;
    }
    .tab-content.active {
      display: block;
    }
    .grid {
      display: grid;
      grid-template-columns: repeat(5, 1fr);
      gap: 10px;
      max-width: 1000px;
      margin: 0 auto 20px;
    }
    .cell {
      position: relative;
      padding: 15px;
      background-color: #1e1e1e;
      border: 3px solid #999;
      border-radius: 10px;
      cursor: pointer;
      user-select: none;
      overflow: hidden;
    }
    .cell span {
      position: relative;
      z-index: 2;
      font-weight: bold;
      text-shadow: 1px 1px 2px black;
    }
    .cell.checked {
      color: white;
      text-decoration: line-through;
    }
    .cell.checked::before {
      content: "";
      position: absolute;
      inset: 0;
      background-image: var(--bg-image);
      background-size: cover;
      background-position: center;
      opacity: 0.6;
      z-index: 1;
    }
    .library {
      display: flex;
      justify-content: center;
      flex-wrap: wrap;
      gap: 10px;
      margin-top: 10px;
    }
    .library img, .library audio {
      max-height: 50px;
      cursor: pointer;
    }
    button {
      padding: 8px 15px;
      font-size: 0.9em;
      border: none;
      background-color: #ff4081;
      color: white;
      border-radius: 5px;
      cursor: pointer;
    }
    button:hover {
      background-color: #f50057;
    }
    input[type="text"] {
      padding: 8px;
      margin: 5px;
      border-radius: 4px;
      border: none;
      width: 300px;
    }
    /* Nouveaux styles pour petits boutons modifier/supprimer */
    .small-btn {
      font-size: 0.7em;
      padding: 2px 6px;
      margin-left: 5px;
      cursor: pointer;
      border: none;
      background-color: #555;
      color: white;
      border-radius: 3px;
    }
    .small-btn:hover {
      background-color: #ff4081;
    }
  </style>
</head>
<body>
  <div class="tabs">
    <div class="tab active" onclick="switchTab('bingo')">🎮 Bingo</div>
    <div class="tab" onclick="switchTab('edit')">🛠️ Modifier les cases</div>
    <div class="tab" onclick="switchTab('imageLibraryTab')">🖼️ Bibliothèque images</div>
    <div class="tab" onclick="switchTab('soundLibraryTab')">🔊 Bibliothèque sons</div>
  </div>

  <div id="bingo" class="tab-content active">
    <label for="imageInput">URL de l’image pour une case cochée :</label><br />
    <input type="text" id="imageInput" placeholder="https://exemple.com/image.jpg" /><br />

    <label for="soundInput">URL du son à jouer lors du clic :</label><br />
    <input type="text" id="soundInput" placeholder="https://exemple.com/son.mp3" /><br />

    <div class="grid" id="bingoGrid"></div>
    <button onclick="resetBingo()">Réinitialiser</button>
  </div>

  <div id="edit" class="tab-content">
    <h2>Modifier les cases</h2>
    <input type="text" id="newCaseInput" placeholder="Texte de la nouvelle case" />
    <button onclick="addCase()">Ajouter</button>
    <button style="background-color:#f44336; margin-left:10px;" onclick="deleteAllCases()">Supprimer toutes les cases</button>
    <div id="caseList"></div>
  </div>

  <div id="imageLibraryTab" class="tab-content">
    <h2>Bibliothèque d’images</h2>
    <div class="library" id="imageLibrary"></div>
  </div>

  <div id="soundLibraryTab" class="tab-content">
    <h2>Bibliothèque de sons</h2>
    <div class="library" id="soundLibrary"></div>
  </div>

  <script>
    let cases = [];
    const usedImages = new Set();
    const usedSounds = new Set();

    function loadCases() {
      const stored = localStorage.getItem("bingoCases");
      if (stored) {
        cases = JSON.parse(stored);
      } else {
        cases = [
          "Créer du drama",
          "Chiale",
          "Jeremstar drama",
          "La justice c'est pas la meme pour tous",
          "8eme fois enceinte",
          "mes enfants sont heureux",
          "mon couple va mal a cause de ca",
          "harcelement",
          "regard insistant vers la cam",
          "J'ai rien fait pour mériter ca",
          "Notre famille a énormement soufer",
          "Dépression",
          "Acharnement médiatique",
          "Je suis malade",
          "J'ai arrete le porno",
        ];
      }
    }

    function saveCases() {
      localStorage.setItem("bingoCases", JSON.stringify(cases));
    }

    function loadUsedAssets() {
      const storedImages = localStorage.getItem("bingoUsedImages");
      if (storedImages) {
        JSON.parse(storedImages).forEach((url) => usedImages.add(url));
      }

      const storedSounds = localStorage.getItem("bingoUsedSounds");
      if (storedSounds) {
        JSON.parse(storedSounds).forEach((url) => usedSounds.add(url));
      }
    }

    function saveUsedAssets() {
      localStorage.setItem("bingoUsedImages", JSON.stringify([...usedImages]));
      localStorage.setItem("bingoUsedSounds", JSON.stringify([...usedSounds]));
    }

    function renderGrid() {
      const grid = document.getElementById("bingoGrid");
      grid.innerHTML = "";
      cases.forEach((text) => {
        const cell = document.createElement("div");
        cell.className = "cell";
        const span = document.createElement("span");
        span.textContent = text;
        cell.appendChild(span);

        cell.addEventListener("click", () => {
          const imgURL = document.getElementById("imageInput").value.trim();
          const soundURL = document.getElementById("soundInput").value.trim();

          if (imgURL) {
            cell.style.setProperty("--bg-image", `url(${imgURL})`);
            usedImages.add(imgURL);
            updateImageLibrary();
          }

          cell.classList.toggle("checked");

          if (soundURL) {
            const audio = new Audio(soundURL);
            audio.play().catch((e) => console.warn("Erreur audio:", e));
            usedSounds.add(soundURL);
            updateSoundLibrary();
          }
        });

        grid.appendChild(cell);
      });
    }

    function updateImageLibrary() {
      const lib = document.getElementById("imageLibrary");
      lib.innerHTML = "";
      usedImages.forEach((url) => {
        const img = document.createElement("img");
        img.src = url;
        img.onclick = () => (document.getElementById("imageInput").value = url);
        lib.appendChild(img);
      });
      saveUsedAssets();
    }

    function updateSoundLibrary() {
      const lib = document.getElementById("soundLibrary");
      lib.innerHTML = "";
      usedSounds.forEach((url) => {
        const audio = document.createElement("audio");
        audio.src = url;
        audio.controls = true;
        audio.onclick = () => (document.getElementById("soundInput").value = url);
        audio.addEventListener("play", () =>
          (document.getElementById("soundInput").value = url)
        );
        lib.appendChild(audio);
      });
      saveUsedAssets();
    }

    function renderCaseList() {
      const list = document.getElementById("caseList");
      list.innerHTML = "";
      cases.forEach((text, i) => {
        const div = document.createElement("div");
        div.innerHTML = `
          ${text}
          <button class="small-btn" onclick="editCase(${i})">✏️</button>
          <button class="small-btn" onclick="deleteCase(${i})">🗑️</button>
        `;
        list.appendChild(div);
      });
    }

    function addCase() {
      const input = document.getElementById("newCaseInput");
      const value = input.value.trim();
      if (value) {
        cases.push(value);
        input.value = "";
        saveCases();
        renderGrid();
        renderCaseList();
      }
    }

    function editCase(index) {
      const newText = prompt("Modifier la case:", cases[index]);
      if (newText) {
        cases[index] = newText.trim();
        saveCases();
        renderGrid();
        renderCaseList();
      }
    }

    function deleteCase(index) {
      if (confirm("Supprimer cette case ?")) {
        cases.splice(index, 1);
        saveCases();
        renderGrid();
        renderCaseList();
      }
    }

    function deleteAllCases() {
      if (confirm("Voulez-vous vraiment supprimer toutes les cases ?")) {
        cases = [];
        saveCases();
        renderGrid();
        renderCaseList();
      }
    }

    function resetBingo() {
      document.querySelectorAll(".cell").forEach((cell) => {
        cell.classList.remove("checked");
        cell.style.removeProperty("--bg-image");
      });
    }

    function switchTab(tabName) {
      document.querySelectorAll(".tab").forEach((t) => t.classList.remove("active"));
      document.querySelectorAll(".tab-content").forEach((c) => c.classList.remove("active"));
      document.querySelector(`.tab[onclick*="${tabName}"]`).classList.add("active");
      document.getElementById(tabName).classList.add("active");
    }

    // Initialisation
    loadCases();
    loadUsedAssets();
    renderGrid();
    renderCaseList();
    updateImageLibrary();
    updateSoundLibrary();
  </script>
</body>
</html>
