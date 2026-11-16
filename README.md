<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>Les Voyageurs du Temps : Sur les traces de la Corée</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    background: #111;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    font-family: 'Trebuchet MS', sans-serif;
    overflow: hidden;
  }
  /* Zone de jeu : centrée dans la fenêtre, puis mise à l'échelle par JavaScript (fitGame).
     Sans JavaScript, elle reste simplement centrée à sa taille d'origine. */
  #wrap { position: absolute; left: 50%; top: 50%; width: 908px; transform: translate(-50%, -50%); }
  #oldBrowserMsg {
    display: none; position: fixed; top: 0; left: 0; right: 0; bottom: 0; z-index: 2000;
    background: #111; color: #fff; align-items: center; justify-content: center;
    text-align: center; padding: 30px; font-size: 18px; line-height: 1.6;
  }
  #oldBrowserMsg div { max-width: 640px; border: 3px solid #d4af37; border-radius: 12px; padding: 28px 32px; background: #1b1630; }
  #oldBrowserMsg b { color: #d4af37; }
  #ui {
    color: #fff;
    text-shadow: 2px 2px 0 #000;
    font-size: 17px;
    font-weight: bold;
    width: 900px;
    display: flex;
    justify-content: space-between;
    margin-bottom: 8px;
  }
  /* Nom de zone : fondu entrant puis sortant au centre de l'écran à chaque entrée dans une nouvelle zone */
  #zoneBanner {
    position: absolute; left: 0; right: 0; top: 50%;
    transform: translateY(-50%);
    padding: 18px 0; text-align: center; pointer-events: none; opacity: 0;
    background: linear-gradient(90deg, rgba(0,0,0,0) 0%, rgba(0,0,0,0.5) 18%, rgba(0,0,0,0.5) 82%, rgba(0,0,0,0) 100%);
  }
  #zoneBanner.show { animation: zoneFade 3.2s ease-in-out forwards; }
  #zoneBannerText {
    display: inline-block; max-width: 780px; padding: 0 20px;
    font-family: Georgia, 'Times New Roman', serif; font-size: 36px; font-weight: bold;
    letter-spacing: 3px; line-height: 1.25; color: #fff3c4;
    text-shadow: 0 2px 4px #000, 0 0 14px rgba(0,0,0,0.95), 0 0 28px rgba(212,175,55,0.55);
  }
  @keyframes zoneFade {
    0%   { opacity: 0; transform: translateY(-50%) scale(0.96); }
    22%  { opacity: 1; transform: translateY(-50%) scale(1); }
    72%  { opacity: 1; transform: translateY(-50%) scale(1); }
    100% { opacity: 0; transform: translateY(-50%) scale(1.02); }
  }
  @media (prefers-reduced-motion: reduce) {
    #zoneBanner.show { animation-name: zoneFadeSimple; }
    @keyframes zoneFadeSimple { 0% { opacity: 0; } 22% { opacity: 1; } 72% { opacity: 1; } 100% { opacity: 0; } }
  }
  #alertBanner {
    position: absolute;
    top: 12px; left: 50%;
    transform: translateX(-50%);
    color: #fff;
    background: rgba(160,0,0,0.85);
    padding: 4px 18px;
    border-radius: 20px;
    font-weight: bold;
    font-size: 14px;
    letter-spacing: 0.5px;
    pointer-events: none;
    display: none;
    animation: pulse 0.8s infinite alternate;
  }
  @keyframes pulse {
    from { opacity: 0.7; transform: translateX(-50%) scale(1); }
    to { opacity: 1; transform: translateX(-50%) scale(1.05); }
  }
  #game {
    background: #3a2a1a;
    border: 4px solid #1a1a1a;
    border-radius: 6px;
    box-shadow: 0 8px 24px rgba(0,0,0,0.5);
    display: block;
  }
  #hint {
    color: #ccc;
    text-shadow: 1px 1px 0 #000;
    margin-top: 8px;
    font-size: 13px;
  }
  #dialogue {
    position: absolute;
    bottom: 20px; left: 20px; right: 20px;
    background: rgba(20,20,30,0.94);
    border: 3px solid #d4af37;
    border-radius: 10px;
    padding: 14px 18px;
    color: #fff;
    display: none;
    font-size: 15px;
    line-height: 1.4;
  }
  #dialogue .name { color: #d4af37; font-weight: bold; margin-bottom: 4px; display:block; }
  #dialogue .continue, #tooltip .continue { font-size: 12px; color: #aaa; margin-top: 8px; text-align: right; }

  #tooltip {
    position: absolute;
    top: 40%; left: 50%;
    transform: translate(-50%, -50%);
    background: rgba(255,250,230,0.97);
    border: 3px solid #d4af37;
    border-radius: 10px;
    padding: 16px 22px;
    max-width: 420px;
    color: #222;
    display: none;
    font-size: 14px;
    line-height: 1.4;
    box-shadow: 0 6px 20px rgba(0,0,0,0.4);
  }
  #tooltip .title { color: #8b5a00; font-weight: bold; margin-bottom: 6px; display:block; font-size:15px;}

  #quizModal {
    position: absolute;
    top: 0; left: 0; right: 0; bottom: 0;
    background: rgba(0,0,0,0.75);
    display: none;
    align-items: center;
    justify-content: center;
  }
  #quizBox {
    background: #fdf6e3;
    border: 4px solid #d4af37;
    border-radius: 12px;
    padding: 28px 32px;
    width: 480px;
    max-width: 90%;
    text-align: center;
  }
  #quizBox h2 { color: #8b5a00; margin-bottom: 6px; }
  #quizBox .sub { color: #555; font-size: 13px; margin-bottom: 18px; }
  #quizQuestion { font-size: 17px; font-weight: bold; margin-bottom: 16px; color: #222; }
  .quizOption {
    display: block;
    width: 100%;
    background: #fff;
    border: 2px solid #d4af37;
    border-radius: 8px;
    padding: 10px;
    margin-bottom: 10px;
    cursor: pointer;
    font-size: 14px;
    text-align: left;
  }
  .quizOption:hover { background: #fff3d6; }
  .quizOption.correct { background: #b7e4b7 !important; border-color: #4caf50; }
  .quizOption.wrong { background: #f5b7b7 !important; border-color: #e63946; }
  #quizFeedback { font-size: 13px; min-height: 20px; margin-top: 8px; color: #444; }

  /* --- Puzzle Hangul (niveau 3) --- */
  #puzzleModal {
    position: absolute; top: 0; left: 0; right: 0; bottom: 0;
    background: rgba(0,0,0,0.75); display: none;
    align-items: center; justify-content: center;
  }
  #puzzleBox {
    background: #fdf6e3; border: 4px solid #d4af37; border-radius: 12px;
    padding: 28px 32px; width: 480px; max-width: 90%; text-align: center;
  }
  #puzzleBox h2 { color: #8b5a00; margin-bottom: 6px; }
  #puzzleBox .sub { color: #555; font-size: 13px; margin-bottom: 18px; }
  #puzzleAnswer { display: flex; justify-content: center; gap: 10px; margin-bottom: 20px; }
  .answerSlot {
    width: 54px; height: 54px; border: 2px dashed #b89b5a; border-radius: 8px;
    display: flex; align-items: center; justify-content: center;
    font-size: 26px; font-weight: bold; color: #8b5a00; background: #fffdf5;
  }
  #puzzleOptions { display: flex; justify-content: center; gap: 10px; flex-wrap: wrap; margin-bottom: 14px; }
  .syllableBtn {
    width: 54px; height: 54px; font-size: 26px; font-weight: bold;
    background: #fff; border: 2px solid #d4af37; border-radius: 8px; cursor: pointer; color: #333;
  }
  .syllableBtn:disabled { opacity: 0.3; cursor: default; }
  .syllableBtn:hover:not(:disabled) { background: #fff3d6; }
  #puzzleFeedback { font-size: 13px; min-height: 20px; margin-top: 8px; color: #444; }
  #puzzleReset { background: none; border: none; color: #8b5a00; text-decoration: underline; cursor: pointer; font-size: 13px; }

  /* --- Horloge narrative (niveau 4) --- */
  #clockModal {
    position: absolute; top: 0; left: 0; right: 0; bottom: 0;
    background: rgba(0,0,0,0.85); display: none;
    align-items: center; justify-content: center;
  }
  #clockBox {
    background: #f5f3eb; border: 4px solid #c9b98a; border-radius: 12px;
    padding: 30px 34px; width: 500px; max-width: 90%; text-align: center;
  }
  #clockBox .clockIcon { font-size: 44px; margin-bottom: 10px; }
  #clockYear { font-size: 22px; font-weight: bold; color: #5a5a40; margin-bottom: 10px; }
  #clockText { font-size: 15px; color: #333; line-height: 1.5; min-height: 70px; margin-bottom: 18px; }
  #clockContinue {
    background: #5a5a40; color: #fff; border: none; padding: 10px 22px;
    border-radius: 8px; font-size: 14px; cursor: pointer; font-weight: bold;
  }
  #clockContinue:hover { background: #45452f; }
  #discussionPrompt {
    display: none; background: #eee5c8; border: 2px dashed #c9b98a; border-radius: 8px;
    padding: 12px 16px; font-size: 13px; color: #4a4a30; margin: 6px 0 20px 0;
    text-align: left; line-height: 1.5;
  }

  /* --- Frise chronologique finale (niveau 6) --- */
  #timelineModal {
    position: absolute; top: 0; left: 0; right: 0; bottom: 0;
    background: rgba(0,0,0,0.88); display: none;
    align-items: center; justify-content: center;
  }
  #timelineBox {
    background: #f5f3ee; border: 4px solid #d4af37; border-radius: 12px;
    padding: 26px 30px; width: 620px; max-width: 92%; text-align: center;
  }
  #timelineBox h2 { color: #4a3a70; margin-bottom: 4px; }
  #timelineBox .sub { color: #555; font-size: 12px; margin-bottom: 16px; }
  #timelineEras { display: flex; justify-content: center; gap: 8px; flex-wrap: wrap; margin-bottom: 16px; }
  .eraBtn {
    background: #fff; border: 2px solid #d4af37; border-radius: 8px;
    padding: 8px 6px; cursor: pointer; font-size: 20px; width: 68px; text-align: center;
  }
  .eraBtn .lbl { display:block; font-size: 9px; margin-top: 3px; color: #555; font-weight: bold; }
  .eraBtn.visited { background: #eef7ea; border-color: #4caf50; }
  .eraBtn:hover { background: #fff3d6; }
  #timelineDetail {
    background: #fffdf5; border: 2px dashed #d4af37; border-radius: 8px;
    padding: 14px 18px; min-height: 90px; text-align: left; font-size: 14px;
    color: #333; line-height: 1.5; margin-bottom: 16px;
  }
  #timelineDetail .eraTitle { font-weight: bold; color: #4a3a70; display: block; margin-bottom: 4px; }
  #compassBtn {
    background: #4a3a70; color: #fff; border: none; padding: 12px 26px;
    border-radius: 8px; font-size: 15px; font-weight: bold; cursor: pointer;
  }
  #compassBtn:disabled { background: #b0aac0; cursor: not-allowed; }
  #compassBtn:hover:not(:disabled) { background: #362a56; }
  #compassHint { font-size: 12px; color: #888; margin-top: 8px; }

  /* --- Toast de fragment obtenu (ne bloque pas le jeu) --- */
  #fragmentToast {
    position: absolute;
    top: 80px; left: 50%;
    transform: translateX(-50%) translateY(-10px);
    background: rgba(20,15,5,0.92);
    border: 2px solid #d4af37;
    border-radius: 10px;
    padding: 10px 20px;
    color: #ffe9b0;
    font-size: 14px;
    font-weight: bold;
    text-align: center;
    opacity: 0;
    pointer-events: none;
    transition: opacity 0.4s, transform 0.4s;
    z-index: 5;
  }
  #fragmentToast.show { opacity: 1; transform: translateX(-50%) translateY(0); }

  /* --- Pop-up de fin + confettis --- */
  #endScreen {
    position: absolute;
    top: 0; left: 0; right: 0; bottom: 0;
    background: rgba(8,6,16,0.78);
    display: none;
    align-items: center;
    justify-content: center;
    padding: 20px;
  }
  #endScreen.show { animation: endFade 0.5s ease-out; }
  @keyframes endFade { from { opacity: 0; } to { opacity: 1; } }
  #endCard {
    background: linear-gradient(160deg, #1b1630, #0f0c1e);
    border: 3px solid #d4af37;
    border-radius: 16px;
    box-shadow: 0 0 40px rgba(212,175,55,0.45), 0 12px 40px rgba(0,0,0,0.6);
    padding: 28px 36px 24px;
    max-width: 560px;
    text-align: center;
    color: #fff;
  }
  #endScreen.show #endCard { animation: endPop 0.7s cubic-bezier(0.18, 1.25, 0.4, 1); }
  @keyframes endPop {
    from { transform: scale(0.4) translateY(40px); opacity: 0; }
    to { transform: scale(1) translateY(0); opacity: 1; }
  }
  #endCard h1 { color: #d4af37; margin-bottom: 6px; font-size: 28px; }
  #endCard p { max-width: 500px; margin: 0 auto 8px; line-height: 1.5; font-size: 14px; }
  #endStats {
    margin: 14px auto 6px;
    display: flex; justify-content: center; gap: 18px; flex-wrap: wrap;
    font-size: 13px; font-weight: bold; color: #ffe9b0;
  }
  #endStats span {
    background: rgba(212,175,55,0.14); border: 1px solid rgba(212,175,55,0.5);
    border-radius: 20px; padding: 4px 12px;
  }
  #replayBtn {
    margin-top: 14px; background: #d4af37; color: #1b1630; border: none;
    padding: 10px 24px; border-radius: 8px; font-size: 15px; font-weight: bold; cursor: pointer;
  }
  #replayBtn:hover { background: #f0c94a; }
  #endHint { font-size: 12px; color: #999; margin-top: 10px; }
  .compassFragment {
    font-size: 60px;
    margin: 6px 0;
    animation: glow 1.5s infinite alternate;
  }
  @keyframes glow {
    from { text-shadow: 0 0 10px #d4af37; }
    to { text-shadow: 0 0 30px #fff, 0 0 40px #d4af37; }
  }
  #confettiCanvas {
    position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
    pointer-events: none; z-index: 1000;
    background: transparent; border: none; border-radius: 0; box-shadow: none;
  }

  /* --- Écran de titre --- */
  #titleScreen {
    position: absolute; top: 0; left: 0; right: 0; bottom: 0; z-index: 50;
    display: flex; flex-direction: column; align-items: center; justify-content: center;
    text-align: center; color: #fff; overflow: hidden; border-radius: 8px;
    background: radial-gradient(ellipse at 50% 36%, #3d2b5a 0%, #1b1630 46%, #0a0713 100%);
    transition: opacity 0.45s ease;
  }
  #titleScreen.leaving { opacity: 0; pointer-events: none; }
  #titleStars {
    position: absolute; top: 0; left: 0; right: 0; bottom: 0; opacity: 0.9;
    background-image:
      radial-gradient(1.5px 1.5px at 12% 18%, #fff, transparent), radial-gradient(1px 1px at 27% 72%, #ffe9b0, transparent),
      radial-gradient(1.5px 1.5px at 41% 30%, #fff, transparent), radial-gradient(1px 1px at 58% 82%, #fff, transparent),
      radial-gradient(2px 2px at 73% 22%, #ffe9b0, transparent), radial-gradient(1px 1px at 86% 64%, #fff, transparent),
      radial-gradient(1.5px 1.5px at 93% 14%, #fff, transparent), radial-gradient(1px 1px at 6% 56%, #fff, transparent),
      radial-gradient(1.5px 1.5px at 66% 48%, #c0a8e0, transparent), radial-gradient(1px 1px at 34% 90%, #fff, transparent),
      radial-gradient(1px 1px at 80% 90%, #ffe9b0, transparent), radial-gradient(1.5px 1.5px at 19% 40%, #c0a8e0, transparent);
    animation: starTwinkle 4s ease-in-out infinite alternate;
  }
  @keyframes starTwinkle { from { opacity: 0.45; } to { opacity: 1; } }
  .titleRing {
    position: absolute; left: 50%; top: 29.3%; border-radius: 50%;
    border: 2px dashed rgba(212,175,55,0.38); transform: translate(-50%, -50%);
  }
  #titleRing1 { width: 250px; height: 250px; animation: ringSpin 40s linear infinite; }
  #titleRing2 { width: 370px; height: 370px; border-style: dotted; border-color: rgba(192,168,224,0.32); animation: ringSpin 70s linear infinite reverse; }
  @keyframes ringSpin { from { transform: translate(-50%, -50%) rotate(0deg); } to { transform: translate(-50%, -50%) rotate(360deg); } }
  #titleCompass { position: relative; font-size: 74px; line-height: 1; margin-bottom: 14px; animation: glow 1.8s infinite alternate; }
  #titleScreen h1 {
    position: relative; margin: 0; padding: 0 20px; font-family: Georgia, 'Times New Roman', serif;
    font-size: 54px; line-height: 1.1; letter-spacing: 1px; color: #f0c94a;
    background: linear-gradient(180deg, #fff3c4 0%, #f0c94a 45%, #b8860b 100%);
    -webkit-background-clip: text; background-clip: text; -webkit-text-fill-color: transparent;
    filter: drop-shadow(0 3px 10px rgba(212,175,55,0.45));
  }
  #titleSubtitle { position: relative; margin-top: 12px; font-size: 17px; letter-spacing: 4px; text-transform: uppercase; color: #c0a8e0; }
  #playBtn {
    position: relative; margin-top: 34px; background: #d4af37; color: #1b1630; border: none;
    padding: 13px 44px; border-radius: 10px; font-size: 21px; font-weight: bold; cursor: pointer;
    box-shadow: 0 0 22px rgba(212,175,55,0.5); animation: playPulse 1.6s ease-in-out infinite alternate;
  }
  #playBtn:hover { background: #f0c94a; }
  @keyframes playPulse { from { transform: scale(1); } to { transform: scale(1.06); } }
  #titleHint { position: relative; margin-top: 18px; font-size: 13px; color: #a9a3bf; }
  #titleFooter {
    position: absolute; left: 0; right: 0; bottom: 14px; text-align: center;
    font-size: 13px; letter-spacing: 1.5px; color: #8f88a8;
  }
  @media (prefers-reduced-motion: reduce) { #titleScreen *, #titleScreen { animation: none !important; } }
</style>
</head>
<body>
<noscript><div style="position:fixed;top:0;left:0;right:0;z-index:3000;color:#fff;background:#111;padding:30px;font-family:sans-serif;text-align:center">
  Ce jeu a besoin de JavaScript. Active-le dans ton navigateur, puis recharge la page.</div></noscript>
<div id="oldBrowserMsg"><div>
  <b>⚠️ Ce jeu ne peut pas démarrer sur ce navigateur.</b><br><br>
  Il nécessite un navigateur récent : <b>Chrome, Edge ou Firefox à jour</b>.<br>
  Ouvre le fichier avec l'un de ces navigateurs (clic droit sur le fichier, « Ouvrir avec »),<br>
  ou demande à la personne qui gère les ordinateurs de le mettre à jour.
</div></div>
<script>
/* Script de garde + substituts pour navigateurs anciens (JavaScript ancien volontairement). */
(function () {
  // Substitut de ctx.roundRect (absent des navigateurs avant 2022) : sans lui, le jeu planterait.
  if (window.CanvasRenderingContext2D && !CanvasRenderingContext2D.prototype.roundRect) {
    CanvasRenderingContext2D.prototype.roundRect = function (x, y, w, h, r) {
      if (r instanceof Array) r = r[0];
      r = Math.min(r || 0, Math.abs(w) / 2, Math.abs(h) / 2);
      this.moveTo(x + r, y);
      this.arcTo(x + w, y, x + w, y + h, r);
      this.arcTo(x + w, y + h, x, y + h, r);
      this.arcTo(x, y + h, x, y, r);
      this.arcTo(x, y, x + w, y, r);
      this.closePath();
    };
  }
  // Si le jeu n'a pas démarré (script incompris ou erreur au lancement), on l'explique au lieu d'afficher un écran vide.
  window.__gameStarted = false;
  function showMsg() {
    var m = document.getElementById('oldBrowserMsg');
    if (m) m.style.display = 'flex';
  }
  window.addEventListener('error', function () { if (!window.__gameStarted) showMsg(); });
  window.addEventListener('load', function () {
    setTimeout(function () { if (!window.__gameStarted) showMsg(); }, 800);
  });
})();
</script>
<div id="wrap">
  <div id="ui">
    <div>🪙 Connaissances: <span id="coins">0</span>/<span id="coinsTotal">0</span></div>
    <div>🧭 Fragments: <span id="fragments">0</span>/6</div>
    <div>⭐ Score: <span id="score">0</span></div>
  </div>
  <div style="position:relative;">
    <canvas id="game" width="900" height="450"></canvas>
    <div id="zoneBanner"><span id="zoneBannerText"></span></div>
    <div id="alertBanner">⚠️ FUIS VERS LA GROTTE AVEC LES TABLETTES ! ⚠️</div>
    <div id="fragmentToast"></div>
    <div id="dialogue">
      <span class="name"></span>
      <span class="text"></span>
      <div class="continue">Appuie sur E pour continuer ▶</div>
    </div>
    <div id="tooltip">
      <span class="title"></span>
      <span class="text"></span>
      <div class="continue">Appuie sur E pour fermer ▶</div>
    </div>
    <div id="quizModal">
      <div id="quizBox">
        <h2 id="quizTitle">⚔️ Le Général de Silla</h2>
        <div class="sub" id="quizSub">Réponds correctement pour obtenir le fragment de la boussole !</div>
        <div id="quizQuestion"></div>
        <div id="quizOptions"></div>
        <div id="quizFeedback"></div>
      </div>
    </div>
    <div id="puzzleModal">
      <div id="puzzleBox">
        <h2>📝 Le Lettré du Roi</h2>
        <div class="sub">Assemble les syllabes en Hangul pour former le nom du roi qui a créé cet alphabet : <strong>Sejong le Grand</strong> (se · jong · dae · wang)</div>
        <div id="puzzleAnswer"></div>
        <div id="puzzleOptions"></div>
        <div id="puzzleFeedback"></div>
        <button id="puzzleReset">↺ Recommencer</button>
      </div>
    </div>
    <div id="clockModal">
      <div id="clockBox">
        <div class="clockIcon">🕰️</div>
        <div id="clockYear"></div>
        <div id="clockText"></div>
        <div id="discussionPrompt"></div>
        <button id="clockContinue">Continuer ▶</button>
      </div>
    </div>
    <div id="timelineModal">
      <div id="timelineBox">
        <h2>🧭 La Boussole retrouve sa mémoire</h2>
        <div class="sub">Clique sur chaque époque pour la revisiter, puis passe le quiz final pour obtenir le dernier fragment.</div>
        <div id="timelineEras"></div>
        <div id="timelineDetail">Clique sur une époque ci-dessus pour afficher son résumé.</div>
        <button id="compassBtn" disabled>Passer le quiz final 🧭</button>
        <div id="compassHint">Revisite les 6 époques pour activer le quiz final.</div>
      </div>
    </div>
    <div id="endScreen">
      <div id="endCard">
        <h1>🎉 La Boussole est complète ! 🎉</h1>
        <div class="compassFragment">🧭</div>
        <p>Des Trois Royaumes à aujourd'hui, un même peuple, une même langue, une même culture a traversé divisions et réunifications successives.</p>
        <p style="color:#d4af37; font-weight:bold;">Haneul referme la boussole, et avec elle, tout ce qu'elle vient d'apprendre sur l'histoire de la Corée.</p>
        <div id="endStats">
          <span>🧭 Fragments : <b id="endFragments">0</b>/6</span>
          <span>🪙 Connaissances : <b id="endCoins">0</b>/<b id="endCoinsTotal">0</b></span>
          <span id="endQuizChip">🧠 Quiz final : <b id="endQuiz">0</b>/<b id="endQuizTotal">0</b></span>
          <span>⭐ Score : <b id="endScore">0</b></span>
        </div>
        <button id="replayBtn">↺ Rejouer l'aventure</button>
        <div id="endHint">ou appuie sur R pour recommencer</div>
      </div>
    </div>
  </div>
  <div id="hint">← → ou Q/D pour bouger • Espace pour sauter • E près d'un PNJ pour parler • R pour recommencer • F pour le plein écran</div>
  <div id="titleScreen">
    <div id="titleStars"></div>
    <div class="titleRing" id="titleRing1"></div>
    <div class="titleRing" id="titleRing2"></div>
    <div id="titleCompass">🧭</div>
    <h1>Les Voyageurs du Temps</h1>
    <div id="titleSubtitle">Sur les traces de la Corée</div>
    <button id="playBtn">▶ Jouer</button>
    <div id="titleHint">Appuie sur Entrée ou Espace pour commencer • F : plein écran</div>
    <div id="titleFooter">Club Corée du Lycée Pierre Mendès France. Anthony CAUDRON & Claude AI.</div>
  </div>
</div>
<canvas id="confettiCanvas"></canvas>

<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
const dialogueEl = document.getElementById('dialogue');
const tooltipEl = document.getElementById('tooltip');
const quizModal = document.getElementById('quizModal');
const puzzleModal = document.getElementById('puzzleModal');
const clockModal = document.getElementById('clockModal');
const timelineModal = document.getElementById('timelineModal');
const endScreen = document.getElementById('endScreen');
const zoneBanner = document.getElementById('zoneBanner');
const zoneBannerText = document.getElementById('zoneBannerText');
const alertBanner = document.getElementById('alertBanner');
const fragmentToast = document.getElementById('fragmentToast');

const GRAVITY = 0.6;
const FRICTION = 0.8;
let cameraX = 0;
let paused = false;

// ============================================================
//  FUSION : Intro (0-450) + Niveau 1 Trois Royaumes (450-3250)
//           + Niveau 2 Goryeo (3250-5850)
//  Tout le contenu des niveaux originaux est décalé en x pour
//  former un seul monde continu, sans écran de chargement.
// ============================================================
const OFFSET1 = 450;   // début du niveau 1 (fin de la zone d'intro)
const OFFSET2 = OFFSET1 + 2800; // 3250 — début du niveau 2
const OFFSET3 = OFFSET2 + 2600; // 5850 — début du niveau 3 (Joseon)
const OFFSET4 = OFFSET3 + 2600; // 8450 — début du niveau 4 (Occupation)
const OFFSET5 = OFFSET4 + 2400; // 10850 — début du niveau 5 (Guerre de Corée)
const OFFSET6 = OFFSET5 + 2800; // 13650 — début du niveau 6 (Corée moderne)

const PHASE2_START = OFFSET2 + 930;   // 4180 — invasion mongole
const PHASE3_START = OFFSET2 + 1830;  // 5080 — grotte refuge

const JOSEON_B1 = OFFSET3 + 800;   // 6650
const JOSEON_B2 = OFFSET3 + 1800;  // 7650
const JOSEON_B3 = OFFSET3 + 2300;  // 8150

const OCC_B1 = OFFSET4 + 900;   // 9350
const OCC_B2 = OFFSET4 + 1800;  // 10250

const WAR_B1 = OFFSET5 + 800;   // 11650
const WAR_B2 = OFFSET5 + 1600;  // 12450
const WAR_B3 = OFFSET5 + 2200;  // 13050
const DMZ38 = WAR_B2;            // ligne du 38e parallèle (dans la zone "vers la Yalu")

const MODERN_B1 = OFFSET6 + 900;  // 14550
const MODERN_B2 = OFFSET6 + 1800; // 15450
const DMZ_MODERN = MODERN_B2;      // ligne de la DMZ (niveau 6)

const ZONE_BOUNDARIES = [
  OFFSET1, OFFSET1 + 950, OFFSET1 + 1900, OFFSET2, PHASE2_START, PHASE3_START,
  OFFSET3, JOSEON_B1, JOSEON_B2, JOSEON_B3,
  OFFSET4, OCC_B1, OCC_B2,
  OFFSET5, WAR_B1, WAR_B2, WAR_B3,
  OFFSET6, MODERN_B1, MODERN_B2
];
const zones = [
  {name:"🏛️ ANCIEN MUSÉE : LE DÉPART",      sky:"#3a2a1a", ground:"#2a1a10"},
  {name:"고구려 GOGURYEO",                   sky:"#4a6fa5", ground:"#3a3a5c"},
  {name:"백제 BAEKJE",                       sky:"#5c9e6f", ground:"#3d5c3a"},
  {name:"신라 SILLA",                        sky:"#c9a227", ground:"#8b6a1a"},
  {name:"🏯 MONASTÈRE DE SONGDO",            sky:"#6ba368", ground:"#3a5a30"},
  {name:"🔥 INVASION MONGOLE",               sky:"#8a3a2a", ground:"#5a1a10"},
  {name:"⛰️ GROTTE REFUGE",                  sky:"#4a5a6b", ground:"#2a3a45"},
  {name:"👑 PALAIS ROYAL : QUARTIER YANGBAN", sky:"#c9a961", ground:"#8b6b2a"},
  {name:"🏘️ QUARTIER POPULAIRE (JOSEON)",    sky:"#8a9a6b", ground:"#5a6b3a"},
  {name:"🛡️ DÉFENSE DE LA FORTERESSE",       sky:"#7a4a3a", ground:"#4a2a1a"},
  {name:"🏛️ SALLE DU ROI SEJONG",            sky:"#5a6b8a", ground:"#3a3a5a"},
  {name:"🌆 RUES SOUS CONTRÔLE COLONIAL",    sky:"#4a4a52", ground:"#2a2a30"},
  {name:"🏘️ QUARTIER RÉSIDENTIEL CLANDESTIN", sky:"#3a3a48", ground:"#22222a"},
  {name:"🕰️ L'HORLOGE DE LA LIBÉRATION",     sky:"#6a6a48", ground:"#4a4a28"},
  {name:"🌊 DÉBARQUEMENT D'INCHEON (1950)",  sky:"#5a7a9a", ground:"#3a4a5a"},
  {name:"🏙️ SÉOUL LIBÉRÉE",                  sky:"#7a7a6a", ground:"#4a4a3a"},
  {name:"🏔️ VERS LA RIVIÈRE YALU",           sky:"#aeb8c2", ground:"#6a7480"},
  {name:"🕊️ PANMUNJOM : ARMISTICE",          sky:"#8a9a7a", ground:"#5a6a4a"},
  {name:"🌆 CORÉE DU SUD : MIRACLE DU HAN",  sky:"#2a1a4a", ground:"#1a0f30"},
  {name:"🏭 CORÉE DU NORD : ISOLEMENT",      sky:"#5a5a5a", ground:"#3a3a3a"},
  {name:"🌉 LE PONT DE LA RÉUNIFICATION",    sky:"#4a6a5a", ground:"#2a4a3a"},
];

// --- Dégradé fluide entre les couleurs de zone ---
const BLEND = 200;
function hexToRgb(hex) {
  const v = parseInt(hex.slice(1), 16);
  return { r: (v >> 16) & 255, g: (v >> 8) & 255, b: v & 255 };
}
function rgbToHex(r, g, b) {
  return '#' + [r, g, b].map(c => Math.round(c).toString(16).padStart(2, '0')).join('');
}
function lerpColor(hexA, hexB, t) {
  t = Math.max(0, Math.min(1, t));
  const a = hexToRgb(hexA), b = hexToRgb(hexB);
  return rgbToHex(a.r + (b.r - a.r) * t, a.g + (b.g - a.g) * t, a.b + (b.b - a.b) * t);
}
function zoneIndexAt(x) {
  let idx = 0;
  for (let i = 0; i < ZONE_BOUNDARIES.length; i++) if (x >= ZONE_BOUNDARIES[i]) idx = i + 1;
  return idx;
}
function getZoneColor(x, key) {
  for (let i = 0; i < ZONE_BOUNDARIES.length; i++) {
    const b = ZONE_BOUNDARIES[i];
    if (x >= b - BLEND && x <= b + BLEND) {
      const t = (x - (b - BLEND)) / (2 * BLEND);
      return lerpColor(zones[i][key], zones[i+1][key], t);
    }
  }
  return zones[zoneIndexAt(x)][key];
}

const level = {
  width: OFFSET6 + 2400, // 16050
  platforms: [
    // --- Intro (musée) ---
    {x:0, y:400, w:450, h:50},
    // --- Niveau 1 : Goguryeo ---
    {x:450, y:400, w:400, h:50},
    {x:900, y:350, w:150, h:20},
    {x:1100, y:300, w:130, h:20},
    {x:1280, y:400, w:150, h:50},
    // --- Niveau 1 : Baekje ---
    {x:1450, y:400, w:250, h:50},
    {x:1750, y:340, w:130, h:20},
    {x:1930, y:280, w:130, h:20},
    {x:2110, y:400, w:280, h:50},
    // --- Niveau 1 : Silla ---
    {x:2430, y:400, w:200, h:50},
    {x:2680, y:330, w:140, h:20},
    {x:2870, y:260, w:140, h:20},
    {x:3060, y:400, w:190, h:50},
    // --- Niveau 2 : Monastère de Songdo ---
    {x:3250, y:400, w:400, h:50},
    {x:3700, y:340, w:150, h:20},
    {x:3900, y:280, w:130, h:20},
    {x:4080, y:400, w:100, h:50},
    // --- Niveau 2 : Invasion mongole ---
    {x:4180, y:400, w:200, h:50},
    {x:4430, y:340, w:120, h:20},
    {x:4600, y:280, w:120, h:20},
    {x:4770, y:400, w:310, h:50},
    // --- Niveau 2 : Grotte refuge ---
    {x:5080, y:400, w:200, h:50},
    {x:5330, y:340, w:150, h:20},
    {x:5530, y:400, w:320, h:50},
    // --- Niveau 3 : Palais (Yangban) ---
    {x:5850, y:400, w:350, h:50},
    {x:6250, y:340, w:150, h:20},
    {x:6450, y:280, w:130, h:20},
    {x:6580, y:400, w:110, h:50},
    // (pont du puzzle Hangul ajouté dynamiquement ici : x:6690-6850)
    // --- Niveau 3 : quartier populaire ---
    {x:6850, y:400, w:250, h:50},
    {x:7150, y:340, w:130, h:20},
    {x:7330, y:280, w:130, h:20},
    {x:7510, y:400, w:200, h:50},
    // --- Niveau 3 : forteresse ---
    {x:7750, y:400, w:150, h:50},
    {x:7950, y:340, w:150, h:20},
    // --- Niveau 3 : salle du roi ---
    {x:8130, y:400, w:320, h:50},
    // --- Niveau 4 : rues coloniales ---
    {x:8450, y:400, w:350, h:50},
    {x:8850, y:340, w:150, h:20},
    {x:9050, y:280, w:130, h:20},
    {x:9230, y:400, w:170, h:50},
    // --- Niveau 4 : quartier résidentiel ---
    {x:9450, y:400, w:220, h:50},
    {x:9730, y:340, w:130, h:20},
    {x:9910, y:280, w:130, h:20},
    {x:10090, y:400, w:210, h:50},
    // --- Niveau 4 : horloge de la Libération ---
    {x:10350, y:400, w:200, h:50},
    {x:10600, y:400, w:250, h:50},
    // --- Niveau 5 : Incheon ---
    {x:10850, y:400, w:350, h:50},
    {x:11250, y:340, w:150, h:20},
    {x:11450, y:280, w:130, h:20},
    {x:11630, y:400, w:150, h:50},
    // --- Niveau 5 : Séoul ---
    {x:11850, y:400, w:220, h:50},
    {x:12130, y:340, w:130, h:20},
    {x:12310, y:280, w:130, h:20},
    {x:12490, y:400, w:190, h:50},
    // --- Niveau 5 : vers la Yalu ---
    {x:12750, y:400, w:150, h:50},
    {x:12950, y:340, w:150, h:20},
    // --- Niveau 5 : Panmunjom ---
    {x:13130, y:400, w:520, h:50},
    // --- Niveau 6 : Corée du Sud ---
    {x:13650, y:400, w:350, h:50},
    {x:14050, y:340, w:150, h:20},
    {x:14250, y:280, w:130, h:20},
    {x:14430, y:400, w:150, h:50},
    // --- Niveau 6 : Corée du Nord ---
    {x:14650, y:400, w:220, h:50},
    {x:14930, y:340, w:130, h:20},
    {x:15110, y:280, w:130, h:20},
    {x:15290, y:400, w:190, h:50},
    // --- Niveau 6 : le pont de la réunification ---
    {x:15550, y:400, w:150, h:50},
    {x:15750, y:340, w:150, h:20},
    {x:15930, y:400, w:120, h:50},
  ],
  coins: [
    // --- Niveau 1 : Trois Royaumes ---
    {x:550,y:350, title:"Goguryeo", text:"Goguryeo (37 av. J.-C. – 668 ap. J.-C.) était le plus grand des Trois Royaumes, connu pour sa puissance militaire et son expansion territoriale au nord."},
    {x:930,y:300, title:"Territoire", text:"Goguryeo s'étendait sur le nord de la péninsule coréenne et une partie de la Chine actuelle."},
    {x:1130,y:250, title:"Force militaire", text:"Les cavaliers et archers de Goguryeo étaient redoutés. Le royaume repoussa même des invasions chinoises."},
    {x:1500,y:350, title:"Baekje", text:"Baekje (18 av. J.-C. – 660 ap. J.-C.) était réputé pour sa culture raffinée et son art, situé au sud-ouest."},
    {x:1780,y:290, title:"Échanges culturels", text:"Baekje entretenait des liens étroits avec la Chine et le Japon, influençant fortement la culture japonaise."},
    {x:1960,y:230, title:"Art bouddhiste", text:"Les artisans de Baekje ont diffusé l'art et le bouddhisme jusqu'au Japon voisin."},
    {x:2150,y:350, title:"Silla", text:"Silla (57 av. J.-C. – 935 ap. J.-C.), situé au sud-est, deviendra finalement le plus puissant des trois royaumes."},
    {x:2470,y:350, title:"Alliance Tang", text:"Silla s'allie à la dynastie Tang de Chine pour renverser ses rivaux."},
    {x:2710,y:280, title:"Chute de Baekje", text:"En 660, l'alliance Silla-Tang met fin au royaume de Baekje."},
    {x:2900,y:210, title:"Chute de Goguryeo", text:"En 668, Goguryeo tombe à son tour, laissant Silla seul maître de la péninsule."},
    {x:3100,y:350, title:"Unification", text:"En 668, Silla unifie la péninsule coréenne pour la première fois de son histoire."},
    // --- Niveau 2 : Goryeo ---
    {x:3350,y:350, title:"Fondation de Goryeo", text:"En 918, Wang Geon se proclame chef d'un nouvel État qu'il nomme Goryeo, mettant fin au déclin de Silla."},
    {x:3730,y:290, title:"Origine du nom Corée", text:"Le nom « Goryeo » est à l'origine du nom moderne « Corée »."},
    {x:3930,y:230, title:"Capitale Songdo", text:"La capitale de Goryeo, Songdo, est aujourd'hui connue sous le nom de Kaesong, en Corée du Nord."},
    {x:4110,y:350, title:"Le bouddhisme à Goryeo", text:"Le bouddhisme joue un rôle central dans la culture et la politique de Goryeo."},
    {x:4460,y:290, title:"Tripitaka Koreana", text:"Le Tripitaka Koreana est une collection de textes bouddhistes gravés sur des tablettes de bois."},
    {x:4810,y:350, title:"Les invasions mongoles", text:"Aux XIIe-XIIIe siècles, les invasions mongoles ravagent le pays et menacent ces précieuses tablettes."},
    // --- Niveau 3 : Joseon ---
    {x:5930,y:350, title:"Fondation de Joseon", text:"En 1392, le général Yi Seong-gye renverse Goryeo et fonde la dynastie Joseon sous le nom de roi Taejo."},
    {x:6280,y:290, title:"Capitale Hanyang", text:"La capitale est établie à Hanyang, aujourd'hui Séoul, capitale de la Corée du Sud."},
    {x:6460,y:230, title:"Le confucianisme", text:"Le confucianisme est la philosophie dominante de Joseon : il structure la société, l'éducation et la gouvernance."},
    {x:6620,y:350, title:"Société hiérarchisée", text:"La société Joseon est très hiérarchisée : les Yangban (élite lettrée) dominent, suivis des paysans, artisans et marchands."},
    {x:7180,y:290, title:"Le peuple de Joseon", text:"Paysans, artisans et marchands forment la majorité de la population, avec un statut social très différent de celui des Yangban."},
    {x:7540,y:350, title:"Les esclaves", text:"Les esclaves formaient la couche la plus basse de la société de Joseon."},
    {x:7980,y:290, title:"Tensions avec les voisins", text:"Joseon a souvent été confronté à des invasions et conflits avec le Japon et la Chine au fil des siècles."},
    // --- Niveau 4 : Occupation japonaise ---
    {x:8530,y:350, title:"L'annexion de 1910", text:"En 1910, le Japon annexe officiellement la Corée par un traité imposé, mettant fin à la dynastie Joseon."},
    {x:8880,y:290, title:"Noms japonais imposés", text:"Les autorités coloniales imposent aux Coréens d'adopter des noms japonais, dans le cadre d'une politique d'assimilation forcée."},
    {x:9080,y:230, title:"Langue coréenne interdite", text:"L'enseignement de la langue coréenne est interdit dans les écoles : préserver la langue devient un acte de résistance."},
    {x:9300,y:350, title:"Exploitation économique", text:"Les ressources naturelles de la Corée sont extraites et envoyées au Japon ; de nombreux Coréens sont forcés de travailler dans des usines et des mines."},
    {x:9760,y:290, title:"Résistance culturelle", text:"Malgré la répression, les Coréens continuent de transmettre leur langue et leurs traditions en secret."},
    {x:10140,y:350, title:"Mouvements d'indépendance", text:"Des mouvements d'indépendance émergent, en Corée comme à l'étranger, pour lutter contre le contrôle japonais."},
    // --- Niveau 5 : Guerre de Corée ---
    {x:10930,y:350, title:"Une péninsule divisée", text:"Après la Seconde Guerre mondiale, la Corée est divisée le long du 38e parallèle : l'URSS occupe le nord, les États-Unis le sud."},
    {x:11150,y:350, isPhoto:true, title:"Photo de famille perdue", text:"Une photo abîmée montre une famille souriante, séparée depuis, comme tant d'autres, par la ligne de division."},
    {x:11480,y:230, title:"Le début du conflit", text:"Le 25 juin 1950, les forces nord-coréennes envahissent la Corée du Sud pour tenter de réunifier la péninsule par la force."},
    {x:11900,y:350, title:"L'intervention de l'ONU", text:"Les Nations Unies, menées par les États-Unis, interviennent pour soutenir la Corée du Sud."},
    {x:12340,y:230, title:"Avancées et retraites", text:"La guerre est marquée par des mouvements rapides de troupes : les lignes de front avancent et reculent à plusieurs reprises."},
    {x:12800,y:350, title:"L'intervention chinoise", text:"Lorsque les troupes de l'ONU approchent de la rivière Yalu, à la frontière chinoise, la Chine intervient massivement pour soutenir la Corée du Nord."},
    {x:13170,y:350, title:"L'armistice de 1953", text:"La guerre se termine le 27 juillet 1953 par la signature de l'armistice de Panmunjom, mais aucun traité de paix formel n'a jamais été signé."},
    // --- Niveau 6 : Corée moderne ---
    {x:13730,y:350, title:"Le miracle sur le fleuve Han", text:"À partir des années 1960, la Corée du Sud connaît une croissance économique rapide et devient l'une des économies les plus avancées au monde."},
    {x:14080,y:290, title:"De la dictature à la démocratie", text:"La Corée du Sud a été dirigée par des régimes autoritaires jusqu'à la fin des années 1980, avant que des mouvements démocratiques n'établissent un gouvernement démocratique."},
    {x:14280,y:230, title:"Samsung, LG et la tech", text:"La Corée du Sud est devenue un leader mondial de la technologie, avec des entreprises comme Samsung et LG."},
    {x:14500,y:350, title:"La vague Hallyu", text:"La K-pop et les dramas coréens sont devenus un phénomène culturel mondial, prolongeant le rayonnement de la Corée du Sud."},
    {x:14700,y:350, title:"La dynastie des Kim", text:"La Corée du Nord est dirigée depuis sa fondation par la dynastie des Kim : Kim Il-sung, Kim Jong-il, puis Kim Jong-un."},
    {x:15140,y:230, title:"Isolement et sanctions", text:"La Corée du Nord est largement isolée sur la scène internationale, en raison notamment de son programme d'armement nucléaire, qui a entraîné des sanctions économiques."},
    {x:15340,y:350, title:"Difficultés économiques", text:"L'économie de la Corée du Nord a stagné en raison de l'isolement et des sanctions ; le pays a connu des famines et une pauvreté généralisée."},
  ],
  enemies: [
    {x:950,y:368,w:32,h:32,dir:1,minX:900,maxX:1050,alive:true,color:"#2c3e70"},
    {x:1300,y:368,w:32,h:32,dir:1,minX:1280,maxX:1430,alive:true,color:"#2c3e70"},
    {x:1800,y:308,w:32,h:32,dir:1,minX:1750,maxX:1880,alive:true,color:"#2e6e3e"},
    {x:2710,y:298,w:32,h:32,dir:-1,minX:2680,maxX:2820,alive:true,color:"#a8811a"},
    {x:3100,y:368,w:32,h:32,dir:1,minX:3060,maxX:3190,alive:true,color:"#a8811a"},
  ],
  hazards: [
    // projectiles non-vaincables (à esquiver) dans la forteresse de Joseon
    {x:7800, y:200, w:20, h:20, dir:1, minX:7770, maxX:8110, speed:2.2},
    {x:7900, y:150, w:20, h:20, dir:-1, minX:7770, maxX:8110, speed:1.8},
    {x:8000, y:250, w:20, h:20, dir:1, minX:7770, maxX:8110, speed:2.5},
  ],
  patrols: [
    // patrouilles japonaises (niveau 4) — à éviter, jamais à combattre
    {x:8860, y:308, w:30, h:32, dir:1, minX:8850, maxX:8970, speed:1.0},
    {x:9350, y:368, w:30, h:32, dir:-1, minX:9240, maxX:9390, speed:0.9},
    {x:9770, y:308, w:30, h:32, dir:1, minX:9730, maxX:9850, speed:0.9},
    {x:10150, y:368, w:30, h:32, dir:-1, minX:10100, maxX:10290, speed:1.0},
  ],
  npcs: [
    {x:1180, y:250, w:34, h:44, name:"Éclaireur de Goguryeo", color:"#4a6fa5",
      lines:["Halte ! Peu d'étrangers s'aventurent sur nos terres montagneuses.",
             "Goguryeo est le plus vaste des Trois Royaumes. Notre force réside dans notre cavalerie.",
             "Continue vers le sud si tu veux atteindre Baekje... mais méfie-toi des rochers qui dévalent la pente !"]},
    {x:2000, y:230, w:34, h:44, name:"Artisan de Baekje", color:"#2e6e3e",
      lines:["Bienvenue à Baekje, terre d'art et de culture.",
             "Nous entretenons d'étroites relations avec le Japon. Nos artisans y ont transmis le bouddhisme et l'art de la sculpture.",
             "La route vers Silla est encore longue. Fais attention, une guerre se prépare..."]},
    {x:2920, y:210, w:34, h:44, name:"Messager de Silla", color:"#a8811a",
      lines:["Silla se prépare à lancer son offensive finale contre Baekje et Goguryeo.",
             "Avec l'aide de la dynastie Tang de Chine, notre royaume va bientôt unifier toute la péninsule...",
             "Le Général t'attend un peu plus loin. Il jugera si tu es digne de comprendre notre histoire."]},
    {x:3950, y:230, w:34, h:44, name:"Roi Taejo (Wang Geon)", color:"#6b4a2b",
      lines:["Bienvenue, jeune voyageuse. Je suis Taejo, fondateur de Goryeo.",
             "Silla déclinait, rongé par les guerres civiles et les soulèvements. J'ai unifié les derniers royaumes rivaux sous une seule bannière en 918.",
             "Nos moines conservent des trésors inestimables dans ce monastère... mais je crains qu'un danger n'approche de l'est."]},
    // --- Niveau 3 : Joseon ---
    {x:6500, y:230, w:34, h:44, name:"Érudit Yangban", color:"#8b6b2a",
      lines:["Salutations, voyageuse. Je suis un lettré Yangban, membre de l'élite de Joseon.",
             "Le confucianisme guide toute notre société : il définit l'éducation, la politique et même les rapports entre les gens.",
             "Notre société est très hiérarchisée. Traverse le royaume et observe par toi-même les différentes conditions de vie."]},
    {x:6650, y:350, w:34, h:44, name:"Lettré du Roi Sejong", color:"#5a4a8b",
      lines:["Le roi Sejong souhaite que chacun, même les plus humbles, puisse apprendre à lire et écrire.",
             "Il a créé un nouvel alphabet : le Hangul, plus simple que les caractères chinois classiques.",
             "Pour prouver que tu as compris son importance, assemble ces syllabes pour former son nom... Un pont apparaîtra si tu réussis !"]},
    {x:7050, y:350, w:34, h:44, name:"Marchand du marché", color:"#6b4a2b",
      lines:["Ici, au marché, nous, marchands, paysans et artisans, faisons vivre le royaume au quotidien.",
             "Nous n'avons pas le même statut que les Yangban, mais sans nous, rien ne fonctionnerait.",
             "Qu'en penses-tu ? Cela mérite peut-être d'en discuter avec ta classe..."]},
    {x:7650, y:350, w:34, h:44, name:"Garde de la forteresse", color:"#7a4a3a",
      lines:["Attention ! Des tensions avec nos voisins provoquent parfois des tirs de catapultes.",
             "Ce n'est rien de grave, mais mieux vaut esquiver les projectiles plutôt que de les recevoir !"]},
    // --- Niveau 4 : Occupation japonaise ---
    {x:8670, y:350, w:34, h:44, name:"Passant discret", color:"#5a5a60",
      lines:["Chut... ne parle pas trop fort près des patrouilles.",
             "Certains d'entre nous cachent encore des livres et des objets coréens, malgré l'interdiction.",
             "Si tu veux aider, retrouve un membre du réseau dans le quartier résidentiel, plus à l'est."]},
    {x:9600, y:350, w:34, h:44, name:"Un résistant anonyme", color:"#6b5a3a",
      lines:["Je ne peux pas te dire mon nom, c'est plus sûr ainsi pour nous tous.",
             "Nous faisons circuler des messages en coréen de maison en maison, malgré l'interdiction.",
             "Ce n'est pas de la violence que nous opposons à l'occupation, mais de la mémoire : notre langue, nos objets, nos histoires. Tant qu'ils survivent, nous résistons.",
             "Aide-nous à mettre à l'abri ce qu'il reste d'objets culturels avant qu'ils ne disparaissent."]},
    // --- Niveau 5 : Guerre de Corée ---
    {x:11330, y:296, w:34, h:44, name:"Réfugiée sud-coréenne", color:"#5a6a7a",
      lines:["Nous avons dû fuir notre village en une nuit, sans savoir si nous reverrions ceux restés derrière...",
             "Tant de familles ont été séparées par cette ligne de front qui n'arrête pas de bouger.",
             "Si tu trouves des objets ou des photos perdus sur ta route, ils comptent peut-être encore beaucoup pour quelqu'un."]},
    {x:11930, y:356, w:34, h:44, name:"Mère cherchant son fils", color:"#7a5a6a",
      lines:["Mon fils... on nous a séparés pendant l'exode vers Séoul. Je ne sais même pas s'il est encore en vie.",
             "Je n'ai qu'une photo de nous deux, mais je l'ai perdue en chemin..."]},
    // --- Niveau 6 : Corée moderne ---
    {x:14130, y:296, w:34, h:44, name:"Jeune entrepreneur séoulite", color:"#e8a33a",
      lines:["Regarde autour de toi : gratte-ciels, néons, technologie... Séoul n'a pas toujours ressemblé à ça.",
             "Pendant longtemps, notre pays a connu des régimes autoritaires. Ce n'est qu'à la fin des années 1980 que la démocratie s'est installée, grâce à des mouvements citoyens.",
             "Depuis, la Corée du Sud a connu une croissance économique et culturelle immense. Mais ça n'a pas toujours été simple d'y arriver."]},
    {x:14770, y:350, w:34, h:44, name:"Panneau d'information officiel", color:"#3a3a3a", isSign:true,
      lines:["【 INFORMATION 】 Les médias sont entièrement contrôlés par l'État, et l'accès à Internet est très restreint pour la population.",
             "【 INFORMATION 】 Même les déplacements à l'intérieur du pays sont soumis à une autorisation officielle.",
             "【 INFORMATION 】 Certaines zones économiques spéciales, comme Kaesong, ont permis par le passé des échanges limités avec l'extérieur."]},
    {x:15670, y:350, w:34, h:44, name:"Passeur du dialogue intercoréen", color:"#4a8a6a",
      lines:["Ce pont symbolise la ligne de démarcation entre les deux Corées : la DMZ.",
             "Des sommets et des tentatives de dialogue ont eu lieu au fil des décennies, avec des avancées... et des tensions persistantes.",
             "La question d'une possible réunification reste ouverte. Les avis divergent, en Corée du Nord comme du Sud, sur si et comment elle pourrait se faire un jour.",
             "Qu'en penses-tu, toi qui as traversé toute cette histoire ?"]},
  ],
  guards: [
    {x:3200, y:340, w:40, h:60, name:"Général de Silla", icon:"⚔️", color:"#7a2f1a", stroke:"#2a1005", type:"quiz",
      quizTitle:"⚔️ Le Général de Silla",
      quizQuestions: [
        { q: "Quels sont les trois royaumes de cette période ?",
          options: ["Goguryeo, Baekje, Silla", "Joseon, Goryeo, Silla", "Baekje, Han, Yuan", "Goguryeo, Tang, Ming"], correct: 0 },
        { q: "Qui a unifié la péninsule coréenne en 668 après J.-C. ?",
          options: ["Goguryeo, seul", "Baekje, avec l'aide du Japon", "Silla, avec l'aide de la dynastie Tang", "Une alliance entre les trois royaumes"], correct: 2 },
        { q: "Quel royaume était réputé pour sa culture et son art, et a influencé la culture japonaise ?",
          options: ["Goguryeo", "Silla", "La dynastie Tang", "Baekje"], correct: 3 }
      ],
      waitLines: ["Halte, jeune voyageuse ! Je suis le Général de Silla.",
                  "Tu n'as pas encore rassemblé toutes les connaissances de cette portion du périple.",
                  "Reviens me voir quand tu auras tout appris sur les Trois Royaumes."],
      toast: "🧭 Fragment 1/6 obtenu ! Direction : la dynastie Goryeo...",
      coinRangeEnd: 11,
      isFinal: false
    },
    {x:5810, y:340, w:40, h:60, name:"Moine bibliothécaire", icon:"📚", color:"#c9a961", stroke:"#151510", type:"quiz",
      quizTitle:"📚 Le Moine Bibliothécaire",
      quizQuestions: [
        { q: "D'où vient le nom moderne « Corée » ?",
          options: ["Du royaume de Silla", "De la dynastie Goryeo", "De la ville de Hanyang", "De la dynastie Joseon"], correct: 1 },
        { q: "Qu'est-ce que le Tripitaka Koreana ?",
          options: ["Un palais royal de Songdo", "Un traité de paix avec les Mongols",
                    "Une collection de textes bouddhistes gravés sur des tablettes de bois", "Un alphabet créé par le roi Taejo"], correct: 2 },
        { q: "Quel peuple a envahi la Corée aux XIIe et XIIIe siècles, menaçant les tablettes ?",
          options: ["Les Japonais", "Les Mongols", "Les Russes", "Les armées de la dynastie Tang"], correct: 1 }
      ],
      waitLines: ["Attends ! Toutes les tablettes ne sont pas encore en sécurité.",
                  "Il en reste quelque part dans le monastère ou sur le chemin de fuite.",
                  "Reviens me voir quand elles seront toutes récupérées."],
      toast: "🧭 Fragment 2/6 obtenu ! Direction : la dynastie Joseon...",
      coinRangeEnd: 17,
      isFinal: false
    },
    {x:8410, y:340, w:40, h:60, name:"Roi Sejong", icon:"👑", color:"#5a4a8b", stroke:"#2a2050", type:"quiz",
      quizTitle:"👑 Le Roi Sejong",
      quizQuestions: [
        { q: "Quelle philosophie guidait la dynastie Joseon ?",
          options: ["Le bouddhisme", "Le confucianisme", "Le taoïsme", "Le chamanisme"], correct: 1 },
        { q: "Qui a créé le Hangul, et pourquoi ?",
          options: ["Le roi Taejo, pour unifier les royaumes rivaux", "Le roi Sejong le Grand, pour améliorer l'alphabétisation du peuple",
                    "Les moines bouddhistes, pour traduire les textes sacrés", "Les marchands, pour faciliter le commerce avec la Chine"], correct: 1 },
        { q: "Comment s'appelait l'élite lettrée qui dominait la société de Joseon ?",
          options: ["Les Yangban", "Les marchands", "Les paysans", "Les artisans"], correct: 0 }
      ],
      waitLines: ["Bienvenue, jeune voyageuse. Avant de repartir, as-tu bien tout appris sur mon royaume ?",
                  "Il te reste des connaissances à recueillir dans Joseon.",
                  "Reviens me voir une fois ta collecte terminée."],
      toast: "🧭 Fragment 3/6 obtenu ! Direction : l'Occupation japonaise...",
      coinRangeEnd: 24,
      isFinal: false
    },
    {x:10770, y:340, w:40, h:60, name:"🕰️ L'horloge", icon:"🕰️", color:"#c9b98a", stroke:"#151510", type:"narrative",
      slides: [
        {year:"1910", text:"Le Japon annexe officiellement la Corée. Pendant plus de 30 ans, la vie quotidienne, l'éducation et la culture coréennes sont placées sous contrôle colonial."},
        {year:"1919", text:"Le 1er mars, des millions de Coréens manifestent pacifiquement pour l'indépendance. Le mouvement est réprimé, mais l'espoir ne s'éteint pas."},
        {year:"1910–1945", text:"Malgré la répression, des Coréens continuent, en secret, de parler leur langue, de transmettre leurs objets et leurs histoires, en Corée comme à l'étranger."},
        {year:"1945", text:"Le Japon est vaincu à la fin de la Seconde Guerre mondiale. C'est la Libération ! Mais la péninsule va bientôt se retrouver divisée...",
         discussion:"💬 À discuter en classe : pourquoi continuer à parler sa langue et à transmettre sa culture, même en secret, peut-il être une forme de résistance ?"}
      ],
      waitLines: ["L'horloge reste figée... il manque encore des objets culturels à mettre en sécurité.",
                  "Reviens une fois ta mission de sauvegarde terminée."],
      toast: "🧭 Fragment 4/6 obtenu ! Direction : la guerre de Corée...",
      coinRangeEnd: 30,
      isFinal: false
    },
    {x:13590, y:340, w:40, h:60, name:"Gardienne de la Mémoire", icon:"🕊️", color:"#405060", stroke:"#202838", type:"quiz",
      quizTitle:"🕊️ La Gardienne de la Mémoire",
      quizQuestions: [
        { q: "En quelle année la guerre de Corée commence-t-elle, et en quelle année se termine-t-elle ?",
          options: ["1945 à 1948", "1950 à 1953", "1953 à 1960", "1910 à 1945"], correct: 1 },
        { q: "Que symbolise la DMZ (zone démilitarisée) ?",
          options: ["La capitale commune des deux Corées", "La frontière séparant le Nord et le Sud depuis l'armistice de 1953",
                    "Le lieu où le traité de paix a été signé", "Une région entièrement reconstruite après la guerre"], correct: 1 },
        { q: "Le long de quelle ligne la Corée a-t-elle été divisée après la Seconde Guerre mondiale ?",
          options: ["Le fleuve Han", "La rivière Yalu", "Le 38e parallèle", "Le 25e parallèle"], correct: 2 }
      ],
      waitLines: ["Je veille sur la mémoire de ce conflit, pour qu'il ne soit jamais oublié.",
                  "Il te reste des connaissances à recueillir avant de pouvoir continuer.",
                  "Reviens me voir une fois ta collecte terminée."],
      toast: "🧭 Fragment 5/6 obtenu ! Direction : la Corée moderne...",
      coinRangeEnd: 37,
      isFinal: false
    },
    {x:16010, y:340, w:40, h:60, name:"Haneul face à la boussole", icon:"🧭", color:"#d4af37", stroke:"#3a2f18", type:"timeline",
      quizTitle:"🧭 Le Quiz Final",
      quizSub:"Dernière épreuve : montre tout ce que tu as retenu de ton voyage à travers l'histoire de la Corée !",
      quizQuestions: [
        // — Les Trois Royaumes —
        { q: "Quelle dynastie chinoise s'est alliée à Silla pour renverser ses rivaux ?",
          options: ["La dynastie Ming", "La dynastie Yuan", "La dynastie Qing", "La dynastie Tang"], correct: 3 },
        { q: "Quel royaume, situé au nord de la péninsule, était réputé pour sa force militaire ?",
          options: ["Baekje", "Goguryeo", "Silla", "Goryeo"], correct: 1 },
        // — La dynastie Goryeo —
        { q: "Qui a fondé la dynastie Goryeo en 918 ?",
          options: ["Yi Seong-gye", "Sejong le Grand", "Wang Geon", "Kim Il-sung"], correct: 2 },
        { q: "Quelle religion jouait un rôle central dans la culture et la politique de Goryeo ?",
          options: ["Le bouddhisme", "Le confucianisme", "Le christianisme", "Le chamanisme"], correct: 0 },
        // — La dynastie Joseon —
        { q: "Quelle capitale de Joseon est devenue celle de la Corée du Sud ?",
          options: ["Songdo (Kaesong)", "Pyongyang", "Hanyang (Séoul)", "Incheon"], correct: 2 },
        { q: "En 1392, quel général renverse Goryeo et fonde la dynastie Joseon ?",
          options: ["Yi Seong-gye", "Wang Geon", "Sejong le Grand", "Kim Il-sung"], correct: 0 },
        // — L'occupation japonaise —
        { q: "En quelle année le Japon a-t-il officiellement annexé la Corée ?",
          options: ["1392", "1910", "1945", "1950"], correct: 1 },
        { q: "Pourquoi préserver la langue coréenne est-il devenu un acte de résistance ?",
          options: ["Parce qu'elle était réservée aux nobles", "Parce qu'elle venait d'être inventée",
                    "Parce que le Japon la jugeait trop difficile", "Parce que son enseignement était interdit dans les écoles"], correct: 3 },
        // — La guerre de Corée —
        { q: "Quel pays est intervenu aux côtés de la Corée du Nord près de la rivière Yalu ?",
          options: ["Le Japon", "Les États-Unis", "La Chine", "La Mongolie"], correct: 2 },
        { q: "Comment la guerre de Corée s'est-elle terminée en 1953 ?",
          options: ["Par un armistice, sans traité de paix formel", "Par un traité de paix signé à Panmunjom",
                    "Par la victoire totale du Sud", "Par la réunification du pays"], correct: 0 },
        // — La Corée moderne —
        { q: "Comment appelle-t-on l'essor économique sud-coréen des années 1960 ?",
          options: ["La vague Hallyu", "Le miracle sur le fleuve Han", "Le grand bond en avant", "La révolution Joseon"], correct: 1 },
        { q: "Quelle question reste ouverte entre la Corée du Nord et la Corée du Sud ?",
          options: ["Le choix de la capitale", "Le nom du pays", "L'alphabet à utiliser", "Une possible réunification"], correct: 3 }
      ],
      waitLines: ["La boussole n'est pas encore complète... il me manque encore des connaissances sur cette dernière époque.",
                  "Il en reste à recueillir, au Sud comme au Nord.",
                  "Revenons ici une fois notre collecte terminée."],
      toast: "🧭 La Boussole est complète !",
      coinRangeEnd: 44,
      isFinal: true
    }
  ]
};
document.getElementById('coinsTotal').textContent = level.coins.length;

const player = {
  x: 50, y: 300, w: 30, h: 40,
  vx: 0, vy: 0,
  speed: 4, jumpPower: 13,
  onGround: false,
  facing: 1,
  invuln: 0
};

let coinsCollected = 0;
let score = 0;
let fragmentsObtained = 0;
let levelComplete = false;
let currentZoneIdx = 0;
let checkpoint = {x:50, y:300};
let guardWarned = {}; // par nom de gardien
let hasFamilyPhoto = false;
let reunionDone = false;

// vague mongole
const wave = {x: PHASE2_START - 350, w: 60, h: 450, active: false};
let phase2Warned = false;

let activeDialogue = null;
let nearbyNPC = null;
let introShown = false;

const keys = {};
let eJustPressed = false;
document.addEventListener('keydown', e => {
  const k = e.key.toLowerCase();
  if (!keys[k] && k === 'e') eJustPressed = true;
  keys[k] = true;
  if (k === 'r') resetGame();
});
document.addEventListener('keyup', e => keys[e.key.toLowerCase()] = false);

const originalCoins = JSON.parse(JSON.stringify(level.coins));
const originalPlatformsCount = level.platforms.length;

// --- Le "pont" du puzzle Hangul (niveau 3) — absent tant que le puzzle n'est pas résolu ---
const BRIDGE = {x: OFFSET3 + 840, y:400, w:160, h:50};
let puzzleSolved = false;
const PUZZLE_TARGET = ['세','종','대','왕']; // Sejong Daewang = "Roi Sejong le Grand"
let puzzleAnswer = [];
let puzzleScrambled = [];
let usedIndices = new Set();

function shuffle(arr) {
  const a = arr.slice();
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}
function openPuzzle() {
  paused = true;
  puzzleAnswer = [];
  usedIndices = new Set();
  puzzleScrambled = shuffle(PUZZLE_TARGET);
  if (puzzleScrambled.join('') === PUZZLE_TARGET.join('')) puzzleScrambled.reverse();
  renderPuzzle();
  puzzleModal.style.display = 'flex';
}
function renderPuzzle() {
  const answerEl = document.getElementById('puzzleAnswer');
  answerEl.innerHTML = '';
  for (let i = 0; i < PUZZLE_TARGET.length; i++) {
    const slot = document.createElement('div');
    slot.className = 'answerSlot';
    slot.textContent = puzzleAnswer[i] || '';
    answerEl.appendChild(slot);
  }
  const optsEl = document.getElementById('puzzleOptions');
  optsEl.innerHTML = '';
  document.getElementById('puzzleFeedback').textContent = '';
  puzzleScrambled.forEach((syl, i) => {
    const btn = document.createElement('button');
    btn.className = 'syllableBtn';
    btn.textContent = syl;
    btn.disabled = usedIndices.has(i);
    btn.onclick = () => pickSyllable(i, syl);
    optsEl.appendChild(btn);
  });
}
function pickSyllable(i, syl) {
  if (usedIndices.has(i) || puzzleAnswer.length >= PUZZLE_TARGET.length) return;
  usedIndices.add(i);
  puzzleAnswer.push(syl);
  renderPuzzle();
  if (puzzleAnswer.length === PUZZLE_TARGET.length) setTimeout(checkPuzzle, 300);
}
function resetPuzzleAttempt() {
  puzzleAnswer = [];
  usedIndices = new Set();
  renderPuzzle();
}
function checkPuzzle() {
  if (puzzleAnswer.join('') === PUZZLE_TARGET.join('')) {
    document.getElementById('puzzleFeedback').textContent = "✅ Exact ! Le pont apparaît...";
    setTimeout(() => {
      puzzleModal.style.display = 'none';
      puzzleSolved = true;
      level.platforms.push({x:BRIDGE.x, y:BRIDGE.y, w:BRIDGE.w, h:BRIDGE.h});
      paused = false;
      startDialogue({name:"Lettré du Roi Sejong", lines:[
        "Bravo ! Tu as assemblé « 세종대왕 » Sejong le Grand.",
        "Le pont est maintenant stable. Tu peux poursuivre ta route vers l'est."
      ]});
    }, 900);
  } else {
    document.getElementById('puzzleFeedback').textContent = "❌ Pas tout à fait... réessaie.";
    setTimeout(resetPuzzleAttempt, 900);
  }
}
document.getElementById('puzzleReset').onclick = resetPuzzleAttempt;

const introDialogue = {
  name: "🧭 Prologue",
  lines: [
    "Tu es Haneul (하늘, « ciel » en coréen), une jeune élève passionnée d'histoire.",
    "Dans un vieux musée coréen, tu viens de trouver une boussole mystérieuse gravée de hanja anciens. En la touchant, tu as été aspirée dans le temps !",
    "Ta mission : traverser les grandes époques de l'histoire coréenne pour réparer la boussole, en récupérant un fragment caché dans chacune d'elles.",
    "Contrôles : ← → ou Q/D pour bouger, Espace pour sauter, E pour interagir avec un PNJ ou fermer une fenêtre, R pour recommencer, F pour le plein écran.",
    "Ton aventure commence à l'époque des Trois Royaumes. Bonne route !"
  ]
};

function showIntro() {
  activeDialogue = { lines: introDialogue.lines, idx: 0, name: introDialogue.name };
  dialogueEl.querySelector('.name').textContent = introDialogue.name;
  dialogueEl.querySelector('.text').textContent = introDialogue.lines[0];
  dialogueEl.style.display = 'block';
  paused = true;
  introShown = true;
}

function resetGame() {
  player.x = 50; player.y = 300;
  player.vx = 0; player.vy = 0;
  player.invuln = 60;
  coinsCollected = 0; score = 0; fragmentsObtained = 0;
  levelComplete = false;
  activeDialogue = null; paused = false;
  guardWarned = {};
  checkpoint = {x:50, y:300};
  currentZoneIdx = 0;
  wave.x = PHASE2_START - 350;
  wave.active = false;
  phase2Warned = false;
  puzzleSolved = false;
  hasFamilyPhoto = false;
  reunionDone = false;
  visitedEras = new Set();
  level.coins = JSON.parse(JSON.stringify(originalCoins));
  level.platforms = level.platforms.slice(0, originalPlatformsCount);
  level.enemies.forEach(en => en.alive = true);
  level.guards.forEach(g => g.__done = false);
  dialogueEl.style.display = 'none';
  tooltipEl.style.display = 'none';
  quizModal.style.display = 'none';
  puzzleModal.style.display = 'none';
  clockModal.style.display = 'none';
  timelineModal.style.display = 'none';
  endScreen.style.display = 'none';
  endScreen.classList.remove('show');
  stopConfetti();
  alertBanner.style.display = 'none';
  zoneBanner.classList.remove('show');   // le nom de zone en cours disparaît
  zoneTitleShownAt = {};                 // on pourra le revoir en rejouant
  updateUI();
  showIntro();
}

function updateUI() {
  document.getElementById('coins').textContent = coinsCollected;
  document.getElementById('score').textContent = score;
  document.getElementById('fragments').textContent = fragmentsObtained;
}

function rectsOverlap(a, b) {
  return a.x < b.x + b.w && a.x + a.w > b.x && a.y < b.y + b.h && a.y + a.h > b.y;
}

function respawnAtCheckpoint() {
  player.x = checkpoint.x; player.y = checkpoint.y;
  player.vx = 0; player.vy = 0;
  player.invuln = 90;
}

function knockback(from) {
  const pushDir = (player.x + player.w/2) < (from.x + from.w/2) ? -1 : 1;
  player.vx = pushDir * 7;
  player.vy = -6;
  player.invuln = 60;
}

function caughtByWave() {
  respawnAtCheckpoint();
  wave.x = PHASE2_START - 350;
}

function showTooltip(coin) {
  tooltipEl.querySelector('.title').textContent = "🪙 " + coin.title;
  tooltipEl.querySelector('.text').textContent = coin.text;
  tooltipEl.style.display = 'block';
  paused = true;
}
function closeTooltip() {
  tooltipEl.style.display = 'none';
  paused = false;
}

function startDialogue(npc) {
  activeDialogue = { lines: npc.lines, idx: 0, name: npc.name, onEnd: npc.onEnd };
  dialogueEl.querySelector('.name').textContent = npc.name;
  dialogueEl.querySelector('.text').textContent = npc.lines[0];
  dialogueEl.style.display = 'block';
  paused = true;
}
function advanceDialogue() {
  activeDialogue.idx++;
  if (activeDialogue.idx >= activeDialogue.lines.length) {
    dialogueEl.style.display = 'none';
    if (activeDialogue.onEnd) activeDialogue.onEnd();
    activeDialogue = null;
    paused = false;
  } else {
    dialogueEl.querySelector('.text').textContent = activeDialogue.lines[activeDialogue.idx];
  }
}

function showFragmentToast(text) {
  fragmentToast.textContent = text;
  fragmentToast.classList.add('show');
  setTimeout(() => fragmentToast.classList.remove('show'), 3200);
}

// ============ QUIZ (générique, par gardien) ============
let currentGuard = null;
let quizIdx = 0;
let quizCorrectCount = 0;

function startQuiz(guard) {
  currentGuard = guard;
  paused = true;
  quizIdx = 0;
  quizCorrectCount = 0;
  document.getElementById('quizTitle').textContent = guard.quizTitle;
  document.getElementById('quizSub').textContent = guard.quizSub || 'Réponds correctement pour obtenir le fragment de la boussole !';
  quizModal.style.display = 'flex';
  renderQuizQuestion();
}
function renderQuizQuestion() {
  const q = currentGuard.quizQuestions[quizIdx];
  document.getElementById('quizQuestion').textContent = `Question ${quizIdx+1}/${currentGuard.quizQuestions.length} — ${q.q}`;
  const optsEl = document.getElementById('quizOptions');
  optsEl.innerHTML = '';
  document.getElementById('quizFeedback').textContent = '';
  q.options.forEach((opt, i) => {
    const btn = document.createElement('button');
    btn.className = 'quizOption';
    btn.textContent = opt;
    btn.onclick = () => answerQuiz(i, btn);
    optsEl.appendChild(btn);
  });
}
function answerQuiz(i, btn) {
  const q = currentGuard.quizQuestions[quizIdx];
  const allBtns = document.querySelectorAll('.quizOption');
  allBtns.forEach(b => b.onclick = null);
  if (i === q.correct) {
    btn.classList.add('correct');
    document.getElementById('quizFeedback').textContent = "✅ Exact !";
    quizCorrectCount++;
  } else {
    btn.classList.add('wrong');
    allBtns[q.correct].classList.add('correct');
    document.getElementById('quizFeedback').textContent = "❌ Pas tout à fait...";
  }
  setTimeout(() => {
    quizIdx++;
    if (quizIdx >= currentGuard.quizQuestions.length) {
      quizModal.style.display = 'none';
      onGuardComplete(currentGuard);
    } else {
      renderQuizQuestion();
    }
  }, 1400);
}

// ============ SÉQUENCE NARRATIVE (horloge, niveau 4) ============
let narrativeIdx = 0;
function startNarrative(guard) {
  currentGuard = guard;
  paused = true;
  narrativeIdx = 0;
  clockModal.style.display = 'flex';
  renderNarrativeSlide();
}
function renderNarrativeSlide() {
  const s = currentGuard.slides[narrativeIdx];
  document.getElementById('clockYear').textContent = s.year;
  document.getElementById('clockText').textContent = s.text;
  const discussionEl = document.getElementById('discussionPrompt');
  const btn = document.getElementById('clockContinue');
  if (s.discussion) {
    discussionEl.style.display = 'block';
    discussionEl.textContent = s.discussion;
    btn.textContent = "Obtenir le fragment de la boussole 🧭";
  } else {
    discussionEl.style.display = 'none';
    btn.textContent = "Continuer ▶";
  }
}
document.getElementById('clockContinue').onclick = () => {
  narrativeIdx++;
  if (narrativeIdx >= currentGuard.slides.length) {
    clockModal.style.display = 'none';
    onGuardComplete(currentGuard);
  } else {
    renderNarrativeSlide();
  }
};

// ============ FRISE CHRONOLOGIQUE FINALE (récapitulative, pas un quiz) ============
const eras = [
  {icon:"⚔️", label:"Trois Royaumes", title:"Les Trois Royaumes (37 av. J.-C. – 668)", text:"Goguryeo, Baekje et Silla se disputent la péninsule, jusqu'à ce que Silla l'unifie en 668 avec l'aide de la dynastie Tang."},
  {icon:"📜", label:"Goryeo", title:"La dynastie Goryeo (935–1392)", text:"Fondée par Taejo (Wang Geon), c'est de son nom que vient « Corée ». Le Tripitaka Koreana y est créé."},
  {icon:"👑", label:"Joseon", title:"La dynastie Joseon (1392–1910)", text:"Guidée par le confucianisme, cette dynastie voit la création du Hangul par le roi Sejong le Grand."},
  {icon:"🕰️", label:"Occupation", title:"L'occupation japonaise (1910–1945)", text:"Malgré la répression, la résistance culturelle et politique se poursuit jusqu'à la Libération de 1945."},
  {icon:"🕊️", label:"Guerre", title:"La guerre de Corée (1950–1953)", text:"Le conflit s'achève par l'armistice de Panmunjom, sans traité de paix formel, la division perdure."},
  {icon:"🌉", label:"Aujourd'hui", title:"La Corée moderne (1953–aujourd'hui)", text:"Deux trajectoires très différentes, et une question toujours ouverte : celle d'une possible réunification."},
];
let visitedEras = new Set();

function startTimeline(guard) {
  currentGuard = guard;
  paused = true;
  renderTimelineEras();
  timelineModal.style.display = 'flex';
}
function renderTimelineEras() {
  const container = document.getElementById('timelineEras');
  container.innerHTML = '';
  eras.forEach((era, i) => {
    const btn = document.createElement('button');
    btn.className = 'eraBtn' + (visitedEras.has(i) ? ' visited' : '');
    btn.innerHTML = era.icon + `<span class="lbl">${era.label}</span>`;
    btn.onclick = () => {
      visitedEras.add(i);
      document.getElementById('timelineDetail').innerHTML =
        `<span class="eraTitle">${era.icon} ${era.title}</span>${era.text}`;
      renderTimelineEras();
      document.getElementById('compassBtn').disabled = visitedEras.size < eras.length;
      document.getElementById('compassHint').textContent =
        visitedEras.size < eras.length
          ? `${visitedEras.size}/${eras.length} époques revisitées.`
          : "Toutes les époques ont été revisitées : tu peux passer le quiz final !";
    };
    container.appendChild(btn);
  });
}
document.getElementById('compassBtn').onclick = () => {
  timelineModal.style.display = 'none';
  if (currentGuard && currentGuard.quizQuestions) startQuiz(currentGuard);   // quiz final : il donne le dernier fragment
  else onGuardComplete(currentGuard);
};

// ============ POP-UP DE FIN + CONFETTIS ============
const confettiCanvas = document.getElementById('confettiCanvas');
const cctx = confettiCanvas.getContext('2d');
const CONFETTI_COLORS = ['#d4af37', '#e63946', '#4a6fa5', '#5c9e6f', '#c0a8e0', '#ffe9b0', '#ff9f1c', '#2ec4b6', '#ffffff'];
let confettiParticles = [];
let confettiRunning = false;
let confettiRainUntil = 0;
let confettiBurstsLeft = [];
let confettiRaf = null;

function resizeConfetti() {
  confettiCanvas.width = window.innerWidth;
  confettiCanvas.height = window.innerHeight;
}
window.addEventListener('resize', resizeConfetti);
resizeConfetti();

function makeConfetti(x, y, vx, vy) {
  return {
    x, y, vx, vy,
    w: 6 + Math.random() * 8,
    h: 4 + Math.random() * 6,
    rot: Math.random() * Math.PI * 2,
    vr: (Math.random() - 0.5) * 0.35,
    tilt: Math.random() * Math.PI * 2,
    vtilt: 0.08 + Math.random() * 0.12,
    color: CONFETTI_COLORS[Math.floor(Math.random() * CONFETTI_COLORS.length)],
    round: Math.random() < 0.25
  };
}

// canons de confettis aux deux coins du bas, tirant vers le haut et le centre
function confettiBurst() {
  const W = confettiCanvas.width, H = confettiCanvas.height;
  for (let i = 0; i < 110; i++) {
    const speed = 12 + Math.random() * 14;
    const a = (-70 - Math.random() * 25) * Math.PI / 180;      // gauche : vers le haut-droite
    confettiParticles.push(makeConfetti(0, H, Math.cos(a) * speed, Math.sin(a) * speed));
    const b = (-110 + Math.random() * 25) * Math.PI / 180;     // droite : vers le haut-gauche
    confettiParticles.push(makeConfetti(W, H, Math.cos(b) * speed, Math.sin(b) * speed));
  }
}

function confettiRain(n) {
  const W = confettiCanvas.width;
  for (let i = 0; i < n; i++) {
    confettiParticles.push(makeConfetti(Math.random() * W, -20, (Math.random() - 0.5) * 3, 1 + Math.random() * 3));
  }
}

function confettiFrame(now) {
  if (!confettiRunning) return;
  const W = confettiCanvas.width, H = confettiCanvas.height;
  cctx.clearRect(0, 0, W, H);

  while (confettiBurstsLeft.length && now >= confettiBurstsLeft[0]) {
    confettiBurstsLeft.shift();
    confettiBurst();
  }
  if (now < confettiRainUntil) confettiRain(3);

  for (let i = confettiParticles.length - 1; i >= 0; i--) {
    const p = confettiParticles[i];
    p.vy += 0.28;           // gravité
    p.vx *= 0.985;          // frottement de l'air
    p.vy *= 0.985;
    if (p.vy > 6) p.vy = 6; // vitesse de chute limite (effet "flottant")
    p.x += p.vx + Math.sin(p.tilt) * 0.8;
    p.y += p.vy;
    p.rot += p.vr;
    p.tilt += p.vtilt;
    if (p.y > H + 30) { confettiParticles.splice(i, 1); continue; }

    cctx.save();
    cctx.translate(p.x, p.y);
    cctx.rotate(p.rot);
    cctx.fillStyle = p.color;
    if (p.round) {
      cctx.beginPath();
      cctx.arc(0, 0, p.w / 2.2, 0, Math.PI * 2);
      cctx.fill();
    } else {
      cctx.fillRect(-p.w / 2, -p.h / 2, p.w, p.h * Math.abs(Math.cos(p.tilt)) + 1); // effet de papier qui se retourne
    }
    cctx.restore();
  }

  if (confettiParticles.length === 0 && now >= confettiRainUntil && confettiBurstsLeft.length === 0) {
    confettiRunning = false;
    cctx.clearRect(0, 0, W, H);
    return;
  }
  confettiRaf = requestAnimationFrame(confettiFrame);
}

function startConfetti() {
  stopConfetti();
  const now = performance.now();
  confettiRunning = true;
  confettiRainUntil = now + 7000;
  confettiBurstsLeft = [now, now + 1400, now + 2800];
  confettiRaf = requestAnimationFrame(confettiFrame);
}

function stopConfetti() {
  confettiRunning = false;
  confettiParticles = [];
  confettiBurstsLeft = [];
  if (confettiRaf) cancelAnimationFrame(confettiRaf);
  confettiRaf = null;
  cctx.clearRect(0, 0, confettiCanvas.width, confettiCanvas.height);
}

function showEndPopup(guard) {
  document.getElementById('endFragments').textContent = fragmentsObtained;
  document.getElementById('endCoins').textContent = coinsCollected;
  document.getElementById('endCoinsTotal').textContent = originalCoins.length;
  document.getElementById('endScore').textContent = score;
  const quizChip = document.getElementById('endQuizChip');
  if (guard && guard.quizQuestions) {
    document.getElementById('endQuiz').textContent = quizCorrectCount;
    document.getElementById('endQuizTotal').textContent = guard.quizQuestions.length;
    quizChip.style.display = '';
  } else {
    quizChip.style.display = 'none';
  }
  endScreen.style.display = 'flex';
  endScreen.classList.remove('show');
  void endScreen.offsetWidth;          // relance l'animation d'apparition
  endScreen.classList.add('show');
  startConfetti();
}
document.getElementById('replayBtn').onclick = () => resetGame();

function onGuardComplete(guard) {
  fragmentsObtained++;
  score += guard.quizQuestions ? (500 + quizCorrectCount * 250) : 700;
  updateUI();
  if (!guard.isFinal) showFragmentToast(guard.toast);   // le pop-up de fin remplace le bandeau pour le dernier fragment
  if (guard.isFinal) {
    levelComplete = true;
    showEndPopup(guard);
  } else {
    // pas de blocage : le joueur continue simplement son chemin
    paused = false;
  }
}

// ============ UPDATE ============
function update() {
  if (titleActive) return;          // écran de titre affiché : le jeu attend
  if (!introShown) { showIntro(); }
  if (levelComplete) return;

  if (eJustPressed) {
    if (activeDialogue) {
      advanceDialogue();
    } else if (tooltipEl.style.display === 'block') {
      closeTooltip();
    } else if (nearbyNPC && !paused) {
      if (nearbyNPC.isGate && !puzzleSolved) {
        openPuzzle();
      } else {
        startDialogue(nearbyNPC);
      }
    }
    eJustPressed = false;
  }

  if (paused) return;

  if (keys['arrowleft'] || keys['q']) { player.vx -= 0.8; player.facing = -1; }
  if (keys['arrowright'] || keys['d']) { player.vx += 0.8; player.facing = 1; }
  player.vx *= FRICTION;
  if (Math.abs(player.vx) > player.speed) player.vx = player.speed * Math.sign(player.vx);

  if ((keys[' '] || keys['arrowup'] || keys['z'] || keys['w']) && player.onGround) {
    player.vy = -player.jumpPower;
    player.onGround = false;
  }

  player.vy += GRAVITY;
  if (player.vy > 15) player.vy = 15;

  player.x += player.vx;
  player.onGround = false;
  for (const p of level.platforms) {
    if (rectsOverlap(player, p)) {
      if (player.vx > 0) player.x = p.x - player.w;
      else if (player.vx < 0) player.x = p.x + p.w;
      player.vx = 0;
    }
  }

  player.y += player.vy;
  for (const p of level.platforms) {
    if (rectsOverlap(player, p)) {
      if (player.vy > 0) { player.y = p.y - player.h; player.vy = 0; player.onGround = true; }
      else if (player.vy < 0) { player.y = p.y + p.h; player.vy = 0; }
    }
  }

  if (player.x < 0) player.x = 0;
  if (player.x + player.w > level.width) player.x = level.width - player.w;

  if (player.y > canvas.height + 100) respawnAtCheckpoint();
  if (player.invuln > 0) player.invuln--;

  // Connaissances
  for (let i = level.coins.length - 1; i >= 0; i--) {
    const c = level.coins[i];
    const coinBox = {x:c.x, y:c.y, w:20, h:20};
    if (rectsOverlap(player, coinBox)) {
      level.coins.splice(i, 1);
      coinsCollected++;
      score += 100;
      if (c.isPhoto) hasFamilyPhoto = true;
      updateUI();
      showTooltip(c);
      break;
    }
  }

  // Ennemis (niveau 1 uniquement)
  for (const en of level.enemies) {
    if (!en.alive) continue;
    en.x += en.dir * 0.75;
    if (en.x < en.minX || en.x + en.w > en.maxX) en.dir *= -1;
    if (rectsOverlap(player, en)) {
      if (player.vy > 0 && player.y + player.h - en.y < 20) {
        en.alive = false;
        player.vy = -8;
        score += 150;
        updateUI();
      } else if (player.invuln <= 0) {
        knockback(en);
      }
    }
  }

  // Hazards (forteresse, niveau 3) — non vaincables, juste à esquiver
  for (const hz of level.hazards) {
    hz.x += hz.dir * hz.speed;
    if (hz.x < hz.minX || hz.x + hz.w > hz.maxX) hz.dir *= -1;
    if (rectsOverlap(player, hz) && player.invuln <= 0) knockback(hz);
  }

  // Patrouilles (niveau 4) — mécanique de discrétion, jamais de combat
  for (const pat of level.patrols) {
    pat.x += pat.dir * pat.speed;
    if (pat.x < pat.minX || pat.x + pat.w > pat.maxX) pat.dir *= -1;
    if (rectsOverlap(player, pat) && player.invuln <= 0) knockback(pat);
  }

  // NPC proximity (le Lettré du Roi Sejong agit comme un PNJ-porte pour le puzzle ;
  // la Mère cherchant son fils propose un dialogue spécial si la photo a été retrouvée)
  nearbyNPC = null;
  for (const npc of level.npcs) {
    const dist = Math.abs((player.x + player.w/2) - (npc.x + npc.w/2));
    if (dist < 60 && Math.abs(player.y - npc.y) < 80) {
      if (npc.name === "Lettré du Roi Sejong") {
        nearbyNPC = Object.assign({isGate:true}, npc);
      } else if (npc.name === "Mère cherchant son fils" && hasFamilyPhoto && !reunionDone) {
        nearbyNPC = {name: npc.name, lines:[
          "Cette photo... c'est... c'est la nôtre ! Où l'as-tu trouvée ?!",
          "Merci, merci infiniment. Je ne sais pas si je reverrai mon fils un jour, mais au moins j'ai ce souvenir de lui.",
          "Même au milieu de la guerre, un petit geste comme le tien redonne un peu d'espoir."
        ], onEnd: () => { reunionDone = true; score += 200; updateUI(); }};
      } else if (npc.name === "Mère cherchant son fils" && reunionDone) {
        nearbyNPC = {name: npc.name, lines:["Merci encore pour cette photo. Je la garde toujours sur moi."]};
      } else {
        nearbyNPC = npc;
      }
      break;
    }
  }

  // --- Détection de zone ---
  const newZoneIdx = zoneIndexAt(player.x);
  if (newZoneIdx !== currentZoneIdx) {
    currentZoneIdx = newZoneIdx;
    showZoneTitle(currentZoneIdx);
  }
  canvas.style.background = getZoneColor(player.x, 'sky');

  // Checkpoints à chaque frontière de zone franchie
  for (const b of ZONE_BOUNDARIES) {
    if (player.x >= b && checkpoint.x < b) checkpoint = {x: b + 20, y: 300};
  }

  // Déclenchement de l'invasion mongole (niveau 2, phase 2)
  if (player.x >= PHASE2_START && player.x < PHASE3_START && !phase2Warned) {
    phase2Warned = true;
    wave.active = true;
    startDialogue({name:"⚠️ Alerte", lines:[
      "Un moine crie : « Les Mongols arrivent par l'ouest ! »",
      "Cours vers la grotte à l'est et mets les tablettes à l'abri, sans les laisser tomber !"
    ]});
  }
  if (wave.active && !paused && player.x < PHASE3_START) {
    wave.x += 1.6;
    if (wave.x > PHASE3_START) wave.x = PHASE3_START;
    if (wave.x + wave.w >= player.x && player.invuln <= 0) {
      caughtByWave();
    }
  }

  // Bannière d'alerte générique (Mongols / forteresse / patrouilles)
  if (wave.active && player.x < PHASE3_START) {
    alertBanner.textContent = "⚠️ FUIS VERS LA GROTTE AVEC LES TABLETTES ! ⚠️";
    alertBanner.style.display = 'block';
  } else if (player.x >= JOSEON_B2 && player.x < JOSEON_B3) {
    alertBanner.textContent = "🛡️ DES ENVAHISSEURS APPROCHENT, ESQUIVE LES PROJECTILES !";
    alertBanner.style.display = 'block';
  } else if (player.x >= OFFSET4 && player.x < OCC_B2) {
    alertBanner.textContent = "🤫 Reste discret, évite les patrouilles !";
    alertBanner.style.display = 'block';
  } else {
    alertBanner.style.display = 'none';
  }

  // Gardiens (fragments de boussole)
  for (const guard of level.guards) {
    if (rectsOverlap(player, {x:guard.x, y:guard.y, w:guard.w, h:guard.h})) {
      // Un gardien attend que toutes les pièces DE SA PORTION soient récupérées.
      const stillMissing = level.coins.length > 0 && coinsCollected < guard.coinRangeEnd;
      if (!stillMissing) {
        if (!guard.__done) {
          guard.__done = true;
          if (guard.type === 'narrative') startNarrative(guard);
          else if (guard.type === 'timeline') startTimeline(guard);
          else startQuiz(guard);
        }
      } else if (!guardWarned[guard.name] && !activeDialogue && !paused) {
        startDialogue({name: guard.name, lines: guard.waitLines});
        guardWarned[guard.name] = true;
      }
    } else {
      guardWarned[guard.name] = false;
    }
  }

  cameraX = player.x - 350;
  if (cameraX < 0) cameraX = 0;
  if (cameraX > level.width - canvas.width) cameraX = level.width - canvas.width;
}

// ============ DRAW ============
function drawVerticalLabel(worldX, text, spaced) {
  ctx.strokeStyle = 'rgba(255,255,255,0.3)';
  ctx.setLineDash([10, 8]);
  ctx.lineWidth = 2;
  ctx.beginPath();
  ctx.moveTo(worldX, 0);
  ctx.lineTo(worldX, canvas.height);
  ctx.stroke();
  ctx.setLineDash([]);
  ctx.save();
  ctx.translate(worldX, 275);
  ctx.rotate(-Math.PI/2);
  ctx.fillStyle = 'rgba(255,255,255,0.55)';
  ctx.font = 'bold 11px sans-serif';
  ctx.textAlign = 'left';
  if (spaced) ctx.letterSpacing = '3px';
  ctx.fillText(text, 0, -6);
  if (spaced) ctx.letterSpacing = '0px';
  ctx.restore();
}

function drawCloud(x, y) {
  ctx.fillStyle = 'rgba(255,255,255,0.85)';
  ctx.beginPath();
  ctx.arc(x, y, 18, 0, Math.PI*2);
  ctx.arc(x+20, y-8, 20, 0, Math.PI*2);
  ctx.arc(x+40, y, 18, 0, Math.PI*2);
  ctx.fill();
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawCloud(150 - cameraX*0.3, 70);
  drawCloud(650 - cameraX*0.3, 90);
  drawCloud(1150 - cameraX*0.3, 60);
  drawCloud(1650 - cameraX*0.3, 80);
  drawCloud(2150 - cameraX*0.3, 65);
  drawCloud(2650 - cameraX*0.3, 75);
  drawCloud(3150 - cameraX*0.3, 60);

  ctx.save();
  ctx.translate(-cameraX, 0);

  // fond en dégradé fluide
  const gradSamples = 16;
  const bgGrad = ctx.createLinearGradient(cameraX, 0, cameraX + canvas.width, 0);
  for (let i = 0; i <= gradSamples; i++) {
    const t = i / gradSamples;
    const worldX = cameraX + t * canvas.width;
    bgGrad.addColorStop(t, getZoneColor(worldX, 'ground') + "33");
  }
  ctx.fillStyle = bgGrad;
  ctx.fillRect(cameraX, 0, canvas.width, canvas.height);

  // décor de la zone musée (intro)
  if (cameraX < 500) {
    ctx.fillStyle = 'rgba(80,55,25,0.5)';
    ctx.fillRect(40, 150, 60, 250);
    ctx.fillRect(340, 150, 60, 250);
  }

  // vague mongole
  if (wave.active) {
    const grad = ctx.createLinearGradient(wave.x - 120, 0, wave.x, 0);
    grad.addColorStop(0, 'rgba(80,0,0,0)');
    grad.addColorStop(1, 'rgba(139,0,0,0.6)');
    ctx.fillStyle = grad;
    ctx.fillRect(0, 0, wave.x, canvas.height);
    ctx.fillStyle = 'rgba(30,10,10,0.8)';
    for (let i = 0; i < 5; i++) {
      const hx = wave.x - 30 - i*45;
      ctx.fillRect(hx, 380, 22, 20);
      ctx.fillRect(hx+6, 365, 10, 18);
    }
  }

  // ligne du 38e parallèle (niveau 5) et de la DMZ (niveau 6) — repères historiques
  if (DMZ38 > cameraX - 50 && DMZ38 < cameraX + canvas.width + 50) {
    drawVerticalLabel(DMZ38, "38e PARALLÈLE - ligne de division de 1945", true);
  }
  if (DMZ_MODERN > cameraX - 50 && DMZ_MODERN < cameraX + canvas.width + 50) {
    drawVerticalLabel(DMZ_MODERN, "DMZ - ZONE DÉMILITARISÉE", true);
  }

  // le gouffre du puzzle Hangul (niveau 3) — repère visuel + panneau explicatif
  const bridgeVisible = (BRIDGE.x + BRIDGE.w + 150 > cameraX) && (BRIDGE.x - 150 < cameraX + canvas.width);
  if (!puzzleSolved && bridgeVisible) {
    const signX = BRIDGE.x + BRIDGE.w/2;
    const signY = 120;
    const sbob = Math.sin(Date.now()/300) * 4;
    ctx.fillStyle = 'rgba(0,0,0,0.75)';
    ctx.strokeStyle = '#d4af37';
    ctx.lineWidth = 3;
    ctx.beginPath();
    ctx.roundRect(signX - 125, signY - 44 + sbob, 250, 84, 12);
    ctx.fill();
    ctx.stroke();
    ctx.fillStyle = '#ffd700';
    ctx.font = 'bold 22px sans-serif';
    ctx.textAlign = 'center';
    ctx.fillText('🧩', signX, signY - 18 + sbob);
    ctx.fillStyle = '#fff';
    ctx.font = 'bold 12px sans-serif';
    ctx.fillText('Résous l\'énigme du Lettré', signX, signY + 6 + sbob);
    ctx.fillText('pour faire apparaître le pont', signX, signY + 22 + sbob);
  }

  // plateformes
  for (const p of level.platforms) {
    const isBridge = puzzleSolved && p.x === BRIDGE.x && p.y === BRIDGE.y;
    ctx.fillStyle = isBridge ? '#a0783c' : '#6b4a2b';
    ctx.fillRect(p.x, p.y, p.w, p.h);
    ctx.fillStyle = isBridge ? '#d4af37' : lerpColor(getZoneColor(p.x, 'ground'), '#ffffff', 0.45);
    ctx.fillRect(p.x, p.y, p.w, 10);
  }

  // pièces de connaissance (flottement)
  for (const c of level.coins) {
    const bob = Math.sin(Date.now()/300 + c.x) * 4;
    if (c.isPhoto) {
      // rendu distinctif "polaroid" pour la photo de famille
      const w = 26, h = 30;
      ctx.save();
      ctx.translate(c.x + 10, c.y + 10 + bob);
      ctx.rotate(-0.12);
      ctx.shadowColor = 'rgba(0,0,0,0.35)';
      ctx.shadowBlur = 4;
      ctx.fillStyle = '#fffdf5';
      ctx.strokeStyle = '#c9a35a';
      ctx.lineWidth = 1.5;
      ctx.beginPath();
      ctx.roundRect(-w/2, -h/2, w, h, 2);
      ctx.fill();
      ctx.stroke();
      ctx.shadowBlur = 0;
      ctx.fillStyle = '#d8c9a0';
      ctx.fillRect(-w/2 + 3, -h/2 + 3, w - 6, h - 12);
      ctx.font = '13px sans-serif';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.fillText('👪', 0, -h/2 + 3 + (h - 12) / 2);
      ctx.textBaseline = 'alphabetic';
      ctx.restore();
      continue;
    }
    ctx.fillStyle = '#ffd700';
    ctx.beginPath();
    ctx.arc(c.x + 10, c.y + 10 + bob, 9, 0, Math.PI*2);
    ctx.fill();
    ctx.strokeStyle = '#c9a300';
    ctx.lineWidth = 2;
    ctx.stroke();
    ctx.fillStyle = '#8b5a00';
    ctx.font = 'bold 11px sans-serif';
    ctx.textAlign = 'center';
    ctx.fillText('?', c.x+10, c.y+14+bob);
  }

  // hasards (forteresse, niveau 3) — projectiles à esquiver
  for (const hz of level.hazards) {
    ctx.fillStyle = '#5a3a2a';
    ctx.beginPath();
    ctx.arc(hz.x + hz.w/2, hz.y + hz.h/2, hz.w/2, 0, Math.PI*2);
    ctx.fill();
    ctx.strokeStyle = '#2a1a10';
    ctx.lineWidth = 2;
    ctx.stroke();
  }

  // patrouilles (niveau 4)
  for (const pat of level.patrols) {
    ctx.fillStyle = '#4a4a3a';
    ctx.fillRect(pat.x, pat.y, pat.w, pat.h);
    ctx.fillStyle = '#2a2a20';
    ctx.fillRect(pat.x + 4, pat.y - 6, pat.w - 8, 8);
    ctx.fillStyle = '#eee';
    ctx.fillRect(pat.x + (pat.dir>0?18:4), pat.y + 8, 5, 5);
  }

  // Gardiens
  for (const guard of level.guards) {
    ctx.fillStyle = guard.color;
    ctx.fillRect(guard.x, guard.y, guard.w, guard.h);
    ctx.strokeStyle = guard.stroke;
    ctx.lineWidth = 3;
    ctx.strokeRect(guard.x, guard.y, guard.w, guard.h);
    ctx.fillStyle = '#fff';
    ctx.font = 'bold 12px sans-serif';
    ctx.textAlign = 'center';
    ctx.fillText(guard.icon, guard.x + guard.w/2, guard.y - 8);
    if (guard.type === 'narrative' && coinsCollected >= guard.coinRangeEnd) {
      ctx.fillStyle = '#ffe9a8';
      ctx.font = 'bold 11px sans-serif';
      ctx.fillText('1945 t\'attend...', guard.x + guard.w/2, guard.y - 26);
    }
    if (guard.type === 'timeline' && coinsCollected >= guard.coinRangeEnd) {
      ctx.fillStyle = '#ffe9a8';
      ctx.font = 'bold 11px sans-serif';
      ctx.fillText('La boussole t\'attend...', guard.x + guard.w/2, guard.y - 26);
    }
  }

  // ennemis (dessinés après les gardiens : toujours visibles, jamais cachés derrière une porte)
  for (const en of level.enemies) {
    if (!en.alive) continue;
    ctx.fillStyle = en.color;
    ctx.fillRect(en.x, en.y, en.w, en.h);
    ctx.fillStyle = '#fff';
    ctx.fillRect(en.x + 6, en.y + 8, 5, 5);
    ctx.fillRect(en.x + 20, en.y + 8, 5, 5);
  }

  // PNJ
  for (const npc of level.npcs) {
    if (npc.isSign) {
      // rendu "panneau d'information" plutôt que silhouette humaine
      ctx.fillStyle = '#5a5a5a';
      ctx.fillRect(npc.x + npc.w/2 - 2, npc.y + 16, 4, npc.h - 16);
      ctx.fillStyle = npc.color;
      ctx.fillRect(npc.x, npc.y, npc.w, 20);
      ctx.strokeStyle = '#eee';
      ctx.lineWidth = 1.5;
      ctx.strokeRect(npc.x, npc.y, npc.w, 20);
      ctx.fillStyle = '#fff';
      ctx.font = 'bold 12px sans-serif';
      ctx.textAlign = 'center';
      ctx.fillText('ℹ️', npc.x + npc.w/2, npc.y + 14);
    } else {
      ctx.fillStyle = npc.color;
      ctx.fillRect(npc.x, npc.y, npc.w, npc.h);
      ctx.fillStyle = '#ffe0bd';
      ctx.beginPath();
      ctx.arc(npc.x + npc.w/2, npc.y - 4, 10, 0, Math.PI*2);
      ctx.fill();
    }

    if (npc.name === "Lettré du Roi Sejong" && !puzzleSolved) {
      const ebob = Math.sin(Date.now()/250) * 4;
      ctx.font = 'bold 20px sans-serif';
      ctx.textAlign = 'center';
      ctx.fillText('🧩', npc.x + npc.w/2, npc.y - 26 + ebob);
    }

    const dist = Math.abs((player.x + player.w/2) - (npc.x + npc.w/2));
    if (dist < 60 && Math.abs(player.y - npc.y) < 80) {
      const bx = npc.x + npc.w/2;
      const by = npc.y - 22 + Math.sin(Date.now()/200) * 3;
      ctx.fillStyle = 'rgba(0,0,0,0.6)';
      ctx.beginPath();
      ctx.roundRect(bx - 12, by - 12, 24, 24, 5);
      ctx.fill();
      ctx.strokeStyle = '#d4af37';
      ctx.lineWidth = 2;
      ctx.stroke();
      ctx.fillStyle = '#fff';
      ctx.font = 'bold 14px sans-serif';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.fillText('E', bx, by + 1);
      ctx.textBaseline = 'alphabetic';
    }
  }

  // joueur
  if (player.invuln === 0 || Math.floor(player.invuln/5)%2===0) {
    ctx.fillStyle = '#e63946';
    ctx.fillRect(player.x, player.y, player.w, player.h);
    ctx.fillStyle = '#ffe0bd';
    ctx.fillRect(player.x + (player.facing>0?14:2), player.y+6, 14, 10);
    ctx.fillStyle = '#1d3557';
    ctx.fillRect(player.x, player.y+26, player.w, 14);
  }

  ctx.restore();
}

function loop() {
  update();
  draw();
  requestAnimationFrame(loop);
}

// ============ NOM DE ZONE (fondu au centre de l'écran) ============
let zoneTitleShownAt = {};   // dernière apparition du nom de chaque zone
function showZoneTitle(idx) {
  const now = Date.now();
  // évite que le nom clignote quand on passe et repasse sur une frontière de zone
  if (zoneTitleShownAt[idx] !== undefined && now - zoneTitleShownAt[idx] < 8000) return;
  zoneTitleShownAt[idx] = now;
  zoneBannerText.textContent = zones[idx].name;
  zoneBanner.classList.remove('show');
  void zoneBanner.offsetWidth;                   // relance l'animation depuis le début
  zoneBanner.classList.add('show');
}

// Le nom de zone se met en pause dès qu'une fenêtre est ouverte (dialogue, pièce, quiz, énigme, frise...)
// et reprend exactement où il en était à la fermeture. `paused` est vrai tant qu'une fenêtre est ouverte.
(function syncZoneBannerPause() {
  zoneBanner.style.animationPlayState = paused ? 'paused' : 'running';
  requestAnimationFrame(syncZoneBannerPause);
})();

// ============ ÉCRAN DE TITRE ============
let titleActive = true;
const titleScreen = document.getElementById('titleScreen');
function startGame() {
  if (!titleActive) return;                       // sécurité : ne s'exécute qu'une fois
  titleActive = false;
  eJustPressed = false;                           // une touche E pressée sur le titre ne doit pas sauter le prologue
  for (const k in keys) keys[k] = false;
  const btn = document.getElementById('playBtn');
  if (btn) btn.blur();
  titleScreen.classList.add('leaving');
  setTimeout(function () { titleScreen.style.display = 'none'; }, 500);
}
document.getElementById('playBtn').onclick = startGame;
document.addEventListener('keydown', function (e) {
  if (titleActive && (e.key === 'Enter' || e.key === ' ')) { e.preventDefault(); startGame(); }
});

updateUI();
loop();

// ============ TAILLE D'ÉCRAN + PLEIN ÉCRAN (ajouts : aucune logique de jeu modifiée) ============
// La zone de jeu garde sa taille interne (900×450) ; elle est seulement agrandie/réduite à l'affichage.
const gameWrap = document.getElementById('wrap');
function fitGame() {
  const w = gameWrap.offsetWidth, h = gameWrap.offsetHeight;   // taille de référence, non affectée par l'échelle
  if (!w || !h) return;
  const margin = 16;
  let s = Math.min((window.innerWidth - margin) / w, (window.innerHeight - margin) / h);
  if (!(s > 0.2)) s = 0.2;
  gameWrap.style.transform = 'translate(-50%, -50%) scale(' + s + ')';
}
window.addEventListener('resize', fitGame);
window.addEventListener('load', fitGame);
['fullscreenchange', 'webkitfullscreenchange', 'MSFullscreenChange'].forEach(function (ev) {
  document.addEventListener(ev, fitGame);
});
fitGame();

function toggleFullscreen() {
  const el = document.documentElement;
  const inFs = document.fullscreenElement || document.webkitFullscreenElement || document.msFullscreenElement;
  const fn = inFs
    ? (document.exitFullscreen || document.webkitExitFullscreen || document.msExitFullscreen)
    : (el.requestFullscreen || el.webkitRequestFullscreen || el.msRequestFullscreen);
  if (!fn) return;                                  // navigateur sans plein écran : on ne fait rien
  try {
    const r = fn.call(inFs ? document : el);
    if (r && r.catch) r.catch(function () {});
  } catch (e) {}
}
document.addEventListener('keydown', function (e) {
  if ((e.key === 'f' || e.key === 'F') && !e.repeat && !e.ctrlKey && !e.metaKey && !e.altKey) toggleFullscreen();
});

window.__gameStarted = true;   // le jeu a bien démarré : le message de repli ne s'affichera pas
</script>
</body>
</html>
