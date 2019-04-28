<template>
    <div id="game"></div>
</template>

<script>
import Phaser from 'phaser'
import wall from '../assets/wall.png'
import ground from '../assets/ground.png'
import player from '../assets/player.png'
import coin from '../assets/coin.png'
import enemy from '../assets/enemy.png'

let score = 0
let scoreText

// TODO -> Detectar quan l'usuari surt del mon -> executar un die

function die (player, enemy) {
  // TODO -> MILLORAR AMB UNA ANIMACIÓ
  player.disableBody(true, true)

  // TODO -> EXECUTAR SO

  // TODO REINICIAR NIVELL
  // console.log(game)
  // console.log(game.camera)
  // console.log(player.scene)
  // console.log(player.scene.game)
  player.scene.cameras.main.shake(500)

  // TODO: Display amb el número de vidas -> Mostrar el número de vidas i si l'has hem gastat aturar el joc

  // TODO: Simular la explosió
}

// SHAKE EFFECT
// http://labs.phaser.io/edit.html?src=src/camera/shake.js

function takeCoin (player, coin) {
  // TODO -> MILLORAR AMB UNA ANIMACIÓ
  coin.disableBody(true, true)
  console.log('XIVATO')

  score = score + 10

  // this.add.text(10, 10, 'score: ' + score, { fontSize: '12px', fill: '#000' })
  scoreText.setText('Score: ' + score)
  // TODO -> EXECUTAR SO QUE PERTOCA (coin.mp3)

  // TODO -> ACTUALITZAR COMPTADOR/SCORE DE PUNTS
}

export default {
  name: 'Game',
  mounted () {
    // PHASER 2.6 -> phaser-ce
    // PHASER 3.0 -> phaser
    let config = {
      type: Phaser.AUTO,
      width: 500,
      height: 200,
      physics: {
        default: 'arcade',
        arcade: {
          gravity: { y: 200 }
        }
      },
      // NO HI HA STATES A 3.0 -> SCENES
      scene: {
        preload () {
          console.log('PRELOAD')
          // CARREGAR ASSETS -> IMAGES, PLAYERS (SPRITESHEETS), AUDIOS, VIDEOS
          this.load.image('wall', wall)
          this.load.image('ground', ground)
          this.load.spritesheet('player', player, { frameWidth: 28, frameHeight: 22 })

          this.load.image('coin', coin)
          this.load.image('enemy', enemy)

          // AUDIO
          // this.load.setBaseURL('http://labs.phaser.io')
          // this.load.audio('dust', ['assets/dead.wav', 'assets/dead.mp3'])
        },
        create () {
          // INITILIZE del nivell -> Afegirem tiles (level: pareds, terras), afegirem player/s, collectibles (coins) , enemies
          console.log('CREATED')

          this.cameras.main.backgroundColor.setTo(52, 152, 219)

          // GROUP PER DEFECTE ES DINAMIC
          // this.level = this.physics.add.group()
          this.level = this.physics.add.staticGroup()

          // this.add.image(500 / 2 - 160, 200 / 2, 'wall')
          // this.add.image(500 / 2 + 160, 200 / 2, 'wall')
          // this.ground = this.physics.add.image(500 / 2, 200 / 2 + 30, 'ground')

          this.level.create(500 / 2 - 160, 200 / 2, 'wall')
          this.level.create(500 / 2 + 160, 200 / 2, 'wall')
          this.level.create(500 / 2, 200 / 2 + 30, 'ground')

          // this.ground.body.allowGravity = false
          // PLAYER -> FÍSIQUES
          this.player = this.physics.add.sprite(500 / 2, 200 / 2 - 50, 'player')
          this.player.setBounce(0.2)
          // this.player.setCollideWorldBounds(true)

          this.physics.add.collider(this.player, this.level)

          // DEFINE SOUNDS
          // this.dustSound = this.add.audio('dust', 0.1)

          // cursors
          this.cursors = this.input.keyboard.createCursorKeys()

          // ANIMATE PLAYER
          this.anims.create({
            key: 'idle',
            frames: this.anims.generateFrameNumbers('player', { start: 3, end: 5 }),
            frameRate: 5,
            repeat: -1
          })

          // ADD COINS
          this.coins = this.physics.add.group()

          this.coins.create(140, 200 / 2, 'coin')
          this.coins.create(170, 200 / 2, 'coin')
          this.coins.create(200, 200 / 2, 'coin')

          this.physics.add.collider(this.coins, this.level)
          this.physics.add.overlap(this.player, this.coins, takeCoin, null, this)

          // SCORE
          scoreText = this.add.text(10, 10, 'Score: 0', { fontSize: '12px', fill: '#000' })

          // ENEMIES

          this.enemies = this.physics.add.group()
          this.enemies.create(500 / 2 + 130, 200 / 2, 'enemy')

          this.physics.add.collider(this.enemies, this.level)
          this.physics.add.overlap(this.player, this.enemies, die, null, this)

          // PREPARE LOSER TEXT
          this.loserText = this.add.text(500 / 2 - 50, 200 / 2 - 50, 'LOOSER!', { fontSize: '30px', fill: '#000' }).setVisible(false)

          this.cameras.main.on('camerashakestart', () => {
            this.loserText.setVisible(true)
          })

          this.cameras.main.on('camerashakecomplete', () => {
            this.loserText.setVisible(false)
          })
        },
        update () {
          this.player.anims.play('idle', true)

          // INPUT EVENTS
          if (this.cursors.left.isDown) {
            this.player.setVelocityX(-160)
            this.player.setFrame(2)
          } else if (this.cursors.right.isDown) {
            this.player.setVelocityX(160)
            this.player.setFrame(1)
          } else {
            this.player.setVelocityX(0)
          }

          // JUMP
          if (this.cursors.up.isDown && this.player.body.touching.down) {
            this.player.setVelocityY(-160)
          }
          // MULTIPLE JUMP (JUMP ON THE AIR)
          // if (this.cursors.up.isDown) {
          //   this.player.setVelocityY(-160)
          // }

          // console.log('UPDATED')
          // ESTE EL QUE S'executa continuament al Game loop -> 60 vegades per segon o FPS

          // if (this.player.body.touching.down && this.player.y > 10) {
          //   this.dustSound.play()
          // }
        }
      }
    }
    // eslint-disable-next-line
      new Phaser.Game(config)
  }
}
</script>
