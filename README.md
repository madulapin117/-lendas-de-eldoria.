# -lendas-de-eldoria.
Meu primeiro jogo de aventura 2D.



<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Lendas de Eldoria</title>
<script src="https://cdn.jsdelivr.net/npm/phaser@3.90.0/dist/phaser.min.js"></script>
<style>
* { box-sizing: border-box; }
body {
  margin: 0;
  background: #101b19;
  font-family: Arial, sans-serif;
  overflow: hidden;
}
#game {
  width: 100vw;
  height: 100dvh;
}
.hud {
  position: fixed;
  top: 12px;
  left: 12px;
  color: white;
  background: #142b22dd;
  padding: 12px 16px;
  border: 2px solid #789b62;
  border-radius: 12px;
  z-index: 5;
}
.controls {
  position: fixed;
  bottom: 25px;
  left: 0;
  right: 0;
  display: flex;
  justify-content: space-between;
  padding: 0 25px;
  z-index: 10;
  pointer-events: none;
}
.pad {
  display: grid;
  grid-template-columns: repeat(3, 55px);
  gap: 5px;
}
.controls button {
  width: 55px;
  height: 55px;
  border: 2px solid #b8d39a;
  border-radius: 14px;
  background: #203b2bdc;
  color: white;
  font-size: 23px;
  touch-action: none;
  pointer-events: auto;
}
.controls button:active {
  background: #72994d;
}
.empty { visibility: hidden; }
</style>
</head>
<body>

<div class="hud">
  <strong>🌿 LENDAS DE ELDORIA</strong><br>
  <small>Explore a floresta misteriosa</small>
</div>

<div id="game"></div>

<div class="controls">
  <div class="pad">
    <button class="empty"> </button>
    <button data-dir="up">▲</button>
    <button class="empty"> </button>
    <button data-dir="left">◀</button>
    <button data-dir="down">▼</button>
    <button data-dir="right">▶</button>
  </div>
</div>

<script>
const held = {
  up:false, down:false, left:false, right:false
};

document.querySelectorAll("[data-dir]").forEach(btn => {
  const dir = btn.dataset.dir;
  btn.addEventListener("pointerdown", e => {
    e.preventDefault();
    held[dir] = true;
    btn.setPointerCapture(e.pointerId);
  });
  ["pointerup","pointercancel","lostpointercapture"].forEach(event => {
    btn.addEventListener(event, () => held[dir] = false);
  });
});

class Forest extends Phaser.Scene {
  constructor() {
    super("Forest");
  }

  create() {
    const W = 1600, H = 1200;

    this.physics.world.setBounds(0,0,W,H);

    this.add.rectangle(W/2,H/2,W,H,0x315c3d);

    // Grama em pixel art
    const grass = this.add.graphics();
    for(let x=0;x<W;x+=32) {
      for(let y=0;y<H;y+=32) {
        const n = (x*13+y*7)%5;
        grass.fillStyle(
          [0x3b7045,0x477b4a,0x35653d,0x527e48,0x416d42][n]
        );
        grass.fillRect(x,y,32,32);
        if(n===1) {
          grass.fillStyle(0x71934e);
          grass.fillRect(x+9,y+12,3,3);
        }
      }
    }

    // Caminho da floresta
    const path = this.add.graphics();
    path.fillStyle(0xb49a68);
    path.fillPoints([
      new Phaser.Math.Vector2(0,550),
      new Phaser.Math.Vector2(0,650),
      new Phaser.Math.Vector2(550,650),
      new Phaser.Math.Vector2(800,800),
      new Phaser.Math.Vector2(1100,700),
      new Phaser.Math.Vector2(1600,720),
      new Phaser.Math.Vector2(1600,630),
      new Phaser.Math.Vector2(1100,610),
      new Phaser.Math.Vector2(800,700),
      new Phaser.Math.Vector2(550,550)
    ],true);

    // Árvores pixeladas
    for(let i=0;i<65;i++) {
      const x=40+((i*197)%1500);
      const y=40+((i*137)%1100);
      if(y>520 && y<810 && x<1100) continue;
      this.tree(x,y);
    }

    // Personagem
    this.player = this.add.container(350,600);
    const p = this.add.graphics();

    // Sombra
    p.fillStyle(0x172b20,0.45);
    p.fillEllipse(0,17,30,10);

    // Pernas
    p.fillStyle(0x45352e);
    p.fillRect(-10,5,8,13);
    p.fillRect(3,5,8,13);

    // Corpo
    p.fillStyle(0x277b53);
    p.fillRect(-13,-9,26,18);

    // Cabeça
    p.fillStyle(0xf1bd87);
    p.fillRect(-10,-25,20,18);

    // Cabelo
    p.fillStyle(0x493322);
    p.fillRect(-11,-27,22,7);
    p.fillRect(-11,-20,4,8);

    // Olhos
    p.fillStyle(0x28251e);
    p.fillRect(4,-18,3,3);

    // Capa
    p.fillStyle(0x9a493c);
    p.fillRect(-15,-9,5,17);

    this.player.add(p);
    this.physics.add.existing(this.player);
    this.player.body.setSize(20,25);
    this.player.body.setCollideWorldBounds(true);

    // Câmera
    this.cameras.main.setBounds(0,0,W,H);
    this.cameras.main.startFollow(this.player);
    this.cameras.main.setZoom(1.2);

    this.cursors = this.input.keyboard.createCursorKeys();
    this.keys = this.input.keyboard.addKeys("W,A,S,D");
  }

  tree(x,y) {
    const t=this.add.graphics();
    t.fillStyle(0x483728);
    t.fillRect(x-7,y,14,30);
    t.fillStyle(0x1d472e);
    t.fillRect(x-25,y-25,50,35);
    t.fillStyle(0x28603a);
    t.fillRect(x-19,y-39,38,24);
    t.fillStyle(0x47804b);
    t.fillRect(x-9,y-43,18,12);
    t.fillStyle(0x75a45a);
    t.fillRect(x-12,y-31,7,5);
  }

  update() {
    const speed=170;
    let vx=0,vy=0;
    const c=this.cursors, k=this.keys;

    if(c.left.isDown || k.A.isDown || held.left) vx=-1;
    if(c.right.isDown || k.D.isDown || held.right) vx=1;
    if(c.up.isDown || k.W.isDown || held.up) vy=-1;
    if(c.down.isDown || k.S.isDown || held.down) vy=1;

    const length=Math.hypot(vx,vy)||1;
    this.player.body.setVelocity(
      vx/length*speed,
      vy/length*speed
    );
  }
}

const config = {
  type: Phaser.AUTO,
  parent: "game",
  backgroundColor: "#315c3d",
  pixelArt: true,
  physics: {
    default: "arcade",
    arcade: { debug:false }
  },
  scale: {
    mode: Phaser.Scale.RESIZE,
    width: window.innerWidth,
    height: window.innerHeight
  },
  scene: Forest
};

new Phaser.Game(config);
</script>
</body>
</html>

