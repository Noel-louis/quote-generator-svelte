<script>
  let width = 800;
  let height = 700;
  let imagesPrecedentes = [];
  let premiereImage = true;
  let selectedImage = null;
  function openImage(image) {
    selectedImage = image;
  }

  function closeModal() {
    selectedImage = null;
  }
  // Fonction pour ajouter une image générée à la liste des images précédentes
  function ajoutImageListe() {
    const canvas = document.getElementById("canvas");
    const imageUrl = canvas.toDataURL("image/png");
    imagesPrecedentes = [imageUrl, ...imagesPrecedentes];
  }

  function changeDimension() {
    height = document.getElementById("height").value;
    width = document.getElementById("width").value;
  }
  async function getRandomQuote() {
    const res = await fetch("https://api.quotable.io/random?maxLength=100");
    const data = await res.json();
    return data.content;
  }
  async function getRandomImage(width, height) {
    const res = await fetch("https://picsum.photos/" + width + "/" + height);
    return res.url;
  }

  function ajoutTexteSurImage(ctx, text, x, y, maxWidth, lineHeight) {
    const mots = text.split(" ");
    let ligne = "";
    let lignes = [];
    for (let n = 0; n < mots.length; n++) {
      let testLine = ligne + mots[n] + " ";
      let metrics = ctx.measureText(testLine);
      let testWidth = metrics.width;
      if (testWidth > maxWidth && n > 0) {
        lignes.push(ligne);
        ligne = mots[n] + " ";
      } else {
        ligne = testLine;
      }
    }
    lignes.push(ligne);

    // Afficher chaque ligne
    for (let i = 0; i < lignes.length; i++) {
      ctx.strokeText(lignes[i], x, y + i * lineHeight);
      ctx.fillText(lignes[i], x, y + i * lineHeight);
    }
  }

  async function getBoth() {
    document.getElementById("genbutton").disabled = true;
    setTimeout(() => {
      document.getElementById("genbutton").disabled = false;
    }, 2000);
    if (!premiereImage) {
      ajoutImageListe();
    } else {
      premiereImage = false;
    }
    let quote = await getRandomQuote();
    let image = await getRandomImage(width, height);

    const canvas = document.getElementById("canvas");
    const ctx = canvas.getContext("2d");
    const img = new Image();
    img.src = image;

    const largeurMax = canvas.width * 0.8; // Largeur maximum du texte (pour ne pas dépasser de l'image)
    const hauteurLigne = 50;
    const x = canvas.width / 2;
    const y = canvas.height / 2;
    img.crossOrigin = "anonymous"; //permet de ne pas avoir d'erreur de sécurité lors du téléchargement de l'image
    img.onload = () => {
      ctx.drawImage(img, 0, 0);
      ctx.font = "40px Times New Roman";
      ctx.fillStyle = "white";
      ctx.lineWidth = 4;
      ctx.strokeStyle = "black";
      ctx.textAlign = "center";
      ajoutTexteSurImage(ctx, quote, x, y, largeurMax, hauteurLigne);
    };
  }
  function downloadFromImage() {
    const canvas = document.getElementById("canvas");
    const image = canvas.toDataURL("image/png");
    const link = document.createElement("a");
    link.href = image;
    link.download = "image.png";
    link.click();
  }
  function downloadFromMiniature(imageUrl) {
    const link = document.createElement("a");
    link.href = imageUrl;
    link.download = "miniature.png";
    link.click();
  }
</script>

<h1>Générateur de citations aléatoires</h1>
<div>
  <button id="genbutton" on:click={getBoth}>Citation aléatoire</button>
  <button id="downloadBtn" on:click={downloadFromImage}
    >Télécharger l'image</button
  >
  <label for="width">Largeur</label>
  <input
    type="number"
    id="width"
    placeholder="Largeur"
    value={width}
    on:change={changeDimension}
  />
  <label for="height">Hauteur</label>
  <input
    type="number"
    id="height"
    placeholder="Hauteur"
    value={height}
    on:change={changeDimension}
  />
</div>

<canvas id="canvas" {width} {height}></canvas>

{#if imagesPrecedentes.length > 0}
  <div class="thumbnails">
    <h2>Historique Des Images</h2>
    {#each imagesPrecedentes as image (image)}
      <div class="thumbnail">
        <img src={image} alt="Miniature" on:click={openImage(image)} />
      </div>
    {/each}
  </div>
{/if}

{#if selectedImage}
  <div class="modal" on:click={closeModal}>
    <div class="modal-content" on:click|stopPropagation>
      <button class="close-btn" on:click={closeModal}>×</button>
      <button
        class="download-btn"
        on:click={() => downloadFromMiniature(selectedImage)}>⬇️</button
      >
      <img src={selectedImage} alt="Grande Image" />
    </div>
  </div>
{/if}

<style>
  @import "random.css";
</style>
