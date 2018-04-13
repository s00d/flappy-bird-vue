<template>
  <div id="app">
    <audio loop="loop" src='./static/sound/bgm.mp3' preload='preload' autoplay="autoplay"></audio>
    <div class="bkg-main" @click="upperBird">
      <div class="score">Record: {{ game.score }}</div>
      <div class="score-max">Max: {{ max }}</div>
      <div class="bkg" :style="styleBgr">
        <div class="pipe-group">
          <div v-for="(pipe, id) in game.pipes" :key="id" class="pipe" :style="{left:pipe.position+'px'}">
             <div class="pipe-top" :style="{height:pipe.topPipeHeight+'px'}"></div>
                <div class="space" style="height:100px"></div>
                <div class="pipe-bottom" :style="{height:bottomPipeHeight(pipe)+'px'}"></div>
          </div>
        </div>

        <div class="bird" :class="'bird-'+game.bird.wingState" :style="styleBird"></div>
      </div>
    </div>

  </div>
</template>

<script>

export default {
  name: 'App',
  data () {
    return {
      max:0,
      eventCallback: {

      },
      defoult: {
        score: 0,
        pipes: [],


        bird: {
          vBird: 1,
          uBird: 10,
          dt: 0.025,

          xBird: 100,
          yBird: 100,

          g: 300,

          wingState: 1,
        },

        bgr: {
          width: 1000
        }
      },
      game: {}
    };
  },
  computed: {
      styleBird() {
        return {
          width: "33px",
          height: "24px",
          left: this.game.bird.xBird + 'px',
          top: this.game.bird.yBird + 'px'
        }
      },
      styleBgr() {
        return {
          width: this.game.bgr.width+'px',
          right: 0
        }
      },
  },
  methods: {
      gameLoop() {
        this.game.bird.vBird += this.game.bird.dt * this.game.bird.g;
        this.game.bird.yBird += this.game.bird.dt * this.game.bird.vBird;
        if(this.game.bird.yBird > 480) this.game.bird.vBird = -100;
        if(this.game.bird.yBird < 0) this.game.bird.vBird = 100;

        // this.bird.uBird += this.bird.dt * this.bird.g
        this.game.bird.xBird += this.game.bird.uBird;

        this.game.bgr.width += this.game.bird.uBird;
        this.checkDue();
      },
      upperBird() {
        this.eventCallback['fly']();
        this.game.bird.vBird = -200;
        this.game.bird.wingState = 2
        setTimeout(() => {
          this.game.bird.wingState = 0
          setTimeout(() => {
            this.game.bird.wingState = 1
          },500);
        },500);
      },
      addPipe() {
        this.game.pipes.push({
          position: this.game.bgr.width,
          topPipeHeight: 100 + Math.random() * 100

        })
      },
      bottomPipeHeight (pipe) {
        return 505 - pipe.topPipeHeight - 100;
      },
      checkDue() {
        if (this.game.pipes.length > 10) delete this.$delete(this.game.pipes, 0);

        this.game.pipes.forEach((pipe, i) => {
          if(this.game.bird.xBird+33 > pipe.position && this.game.bird.xBird < pipe.position + 100) {
            if(this.game.bird.yBird < pipe.topPipeHeight || this.game.bird.yBird+24 > pipe.topPipeHeight+100){
              if(this.max < this.game.score) this.max = this.game.score;
              this.$set(this, 'game', JSON.parse(JSON.stringify(this.defoult)))
              this.eventCallback['hit']();
            }
          }
          if(this.game.score < i+1 && this.game.bird.xBird > pipe.position + 100){
            this.eventCallback['score']();
            this.game.score = i+1
          }
        });
      }
  },
  created () {
    this.$set(this, 'game', JSON.parse(JSON.stringify(this.defoult)))
  },
  mounted () {
    this.addPipe();
    setInterval(this.gameLoop, 30);
    setInterval(this.addPipe, 1000);

     let events = ['fly', 'die', 'score', 'hit']
     events.forEach(event => {
        let el = document.createElement('audio');
        el.src = './static/sound/' + event + '.mp3';
        el.preload = 'preload';
        this.eventCallback[event] = function () {
          el.pause();
          el.currentTime = 0;
          el.play();
        };
     });
  }
}
</script>

<style>
.bkg-main {
  width: 100%;
  height: 505px;
  overflow: hidden;
}
.bkg {
  /* width: 100%; */
  height: 505px;
  background-image: url(./assets/background.png);
  border: 3px solid green;
  position: absolute;
}

.bird {
  position: absolute;
  z-index: 20;
  width: 33px;
  height: 24px;

  transition: transform 0.5s;
}

.bird-0 {
    background: url(./assets/bird1.png) 50% 50% no-repeat;
}
.bird-1 {
    background: url(./assets/bird2.png) 50% 50% no-repeat;
}
.bird-2 {
    background: url(./assets/bird3.png) 50% 50% no-repeat;
}

.pipe-top {
  background: url(./assets/pipedown.png) bottom no-repeat;
  background-size: cover;
}

.pipe-bottom {
  background: url(./assets/pipeup.png) no-repeat;
  background-size: cover;
}

.pipe-group {
  width: 100%;
  height: 100%;
  padding: 0;
  margin: 0;
  z-index: 20;
  overflow: hidden;
}
.pipe-group li {
    text-decoration: none;
}

.pipe {
  width: 100px;

  position: absolute;
}

.space {
  height: 20px;
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.score, .score-max {
  position: absolute;
  left: 20px;
  top: 20px;
  font-size: 2em;
  font-weight: bolder;
}
.score-max {
  left: 200px;
}
</style>
