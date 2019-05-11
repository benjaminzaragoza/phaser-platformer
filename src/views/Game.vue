
<template>
    <div id="game">
        <h1>This is an game page</h1>
    </div>
</template>

<script>
import Phaser from 'phaser'
import wall from '../assets/wall.png'
import ground from '../assets/ground.png'
import player from '../assets/player.png'
import coin from '../assets/coin.png'
import enemy from '../assets/enemy.png'
import soundDead from '../assets/dead.mp3'
import soundCoin from '../assets/coin.mp3'
let score = 0
let scoreText
function takeCoin (player, coin) {
  // todo -> millorar amb una animacio i executar so al agafar la moneda
  coin.disableBody(true, true)
  score = score + 10
  scoreText.setText('Score: ' + score)
  this.sound.play('soundCoin')
}
function die (player, enemy) {
  player.disableBody(true, true)
  player.scene.cameras.main.shake(500)
  this.sound.play('soundDead')
}
export default {
  name: 'Game',
  created () {
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
      scene: {
        preload () {
          this.load.image('wall', wall)
          this.load.image('ground', ground)
          this.load.image('coin', coin)
          this.load.image('enemy', enemy)
          this.load.spritesheet('player', player, { frameWidth: 28, frameHeight: 22 })
          this.load.audio('soundDead', soundDead)
          this.load.audio('soundCoin', soundCoin)
          // AUDIO
          // this.load.setBaseURL('http://labs.phaser.io')
          // this.load.audio('dust', ['assets/dead.wav', 'assets/dead.mp3'])
        },
        create () {
          this.sound.add('soundDead')
          this.sound.add('soundCoin')
          this.cameras.main.backgroundColor.setTo(52, 152, 219)
          this.level = this.physics.add.staticGroup()
          // this.add.image(500 / 2 - 160, 200 / 2, 'wall')
          // this.add.image(500 / 2 + 160, 200 / 2, 'wall')
          // this.add.image(500 / 2, 200 / 2 + 30, 'ground')
          this.level.create(500 / 2 - 160, 200 / 2, 'wall')
          this.level.create(500 / 2 + 160, 200 / 2, 'wall')
          this.level.create(500 / 2, 200 / 2 + 30, 'ground')
          // PLAYER
          this.player = this.physics.add.sprite(500 / 2, 200 / 2 - 50, 'player')
          // this.player.setCollideWorldBounds(true)
          this.player.setBounce(0.2)
          this.physics.add.collider(this.player, this.level)
          // DEFINE SOUNDS
          // this.dustSound = this.add.audio('dust', 0.1)
          this.cursors = this.input.keyboard.createCursorKeys()
          this.anims.create({
            key: 'idle',
            frames: this.anims.generateFrameNumbers('player', { start: 3, end: 5 }),
            frameRate: 5,
            repeat: -1
          })
          this.coins = this.physics.add.group()
          this.coins.create(140, 200 / 2, 'coin')
          this.coins.create(170, 200 / 2, 'coin')
          this.coins.create(200, 200 / 2, 'coin')
          this.physics.add.collider(this.coins, this.level)
          this.physics.add.overlap(this.coins, this.player, takeCoin, null, this)
          // ENEMIES
          this.enemies = this.physics.add.group()
          this.enemies.create(500 / 2 + 130, 200 / 2, 'enemy')
          this.physics.add.collider(this.enemies, this.level)
          this.physics.add.overlap(this.player, this.enemies, die, null, this)
          // SCORE
          scoreText = this.add.text(20, 20, 'Score: 0', { fontSize: '18px', fill: '#000' })
          // LOOSER TEXT
          this.loserText = this.add.text(500 / 2 - 50, 200 / 2 - 50, 'YOU DIED', { fontSize: '30px', fill: '#830' }).setVisible(false)
          this.cameras.main.on('camerashakestart', () => {
            this.loserText.setVisible(true)
          })
          this.cameras.main.on('camerashakecomplete', () => {
            this.loserText.setVisible(false)
          })
        },
        update () {
          // sexecuta continuament al game loop -> 60 vegades per segon
          this.player.anims.play('idle', true)
          if (this.cursors.left.isDown) {
            this.player.setVelocityX(-160)
            this.player.setFrame(2)
          } else if (this.cursors.right.isDown) {
            this.player.setVelocityX(160)
            this.player.setFrame(1)
          } else {
            this.player.setVelocityX(0)
          }
          if (this.cursors.up.isDown && this.player.body.touching.down) {
            this.player.setVelocityY(-160)
          }
          // if (this.player.body.touching.down && this.player.y > 30) {
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
