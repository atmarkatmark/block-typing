<template>
  <div id="app">
    <img src="./assets/logo.png">
    <p v-show="done">スコア: {{ score.toFixed(0) }}</p>
    <p id="display" ref="display" v-html="diff"></p>
    <textarea v-show="!done" id="user" v-model="user" @keydown="onKeydown" @keydown.escape="init" @keydown.delete="incrementBackspaceCount" autofocus></textarea>
    <p>
      <label>速さ</label> {{ velocity.toFixed(1) }} 文字/秒
      <label>時間</label> {{ elapsedTime() !== null ? elapsedTime().toFixed(0) : '-' }} 秒
      <label>字消</label> {{ counter.backspace }}</p>
    <p>{{ judge() }}</p>
    <button @click="init">リセット(Escape)</button>
  </div>
</template>

<script>
const Diff = require('diff')

export default {
  name: 'App',
  components: {},

  data: function () {
    return {
      // 問題文
      targets: [
        '日本語入力をオンにしてください。',
        'これはテスト問題です。',
        '文章を入力してください。'
      ],
      // 問題文の現在位置
      count: 0,
      
      // ユーザの入力
      user: '',

      // 時間経過
      time: {
        start: null,
        end: null,
        diff: null,
        isRunning: false
      },
      animationFrame: null,
      
      // キーカウント
      counter: {
        chars: 0,
        backspace: 0
      }
    }
  },

  computed: {
    // 問題と入力の差分を取る
    diff: function () {
      // 終了状態の場合
      if (this.done)
        return '終了'
      
      // 問題と入力の差分を取る
      const d = Diff.diffChars(this.target, this.user)

      // 着色したspanを格納する - https://www.npmjs.com/package/diff
      let block = ''
      d.forEach(function (part) {
        // 間違いは red, 未入力は black, 入力済は grey
        let color = part.added ? 'red' : (part.removed ? 'black' : 'grey')

        block += '<span style="color: ' + color + '">' + part.value + '</span>'
      })

      return block
    },

    // 問題文
    target: function () {
      return this.count < this.targets.length ? this.targets[this.count] : 'DONE'
    },

    // 終了判定
    done: function () {
      return this.targets.length <= this.count ? true : false
    },

    // 入力速度(文字数/秒)
    velocity: function () {
      return this.counter.chars / this.elapsedTime()
    },

    // スコア計算
    score: function () {
      return this.velocity * 100 - this.counter.backspace * 10
    }
  },

  mounted: function () {
    this.init()
  },

  methods: {
    // 次画面への判定
    judge: function () {
      // 終了判定
      if (this.done) {
        this.stop()
        return '終了'
      }
      
      // 入力判定
      if (this.target + '\n' === this.user) {
        this.counter.chars += this.targets[this.count].length
        ++this.count
        this.user = ''
        return 'OK'
      }

      return this.time.start === null ? '入力してください' : '入力中'
    },

    // キー入力時の処理
    onKeydown: function (e) {
      // IME対策(確定するまでdataに反映されないから)
      this.user = e.target.value

      // 開始処理
      if (this.time.start === null) {
        this.start()
      }
    },

    // 開始時のタイマー処理 - https://webman-japan.com/vue-simple-stopwatch/#javascript
    start: function () {
      this.time.start = performance.now()

      let self = this;
      (function loop() {
        self.time.diff = performance.now() - self.time.start
        self.animationFrame = requestAnimationFrame(loop)
      }())
      this.isRunning = true
    },

    // 終了時のタイマー処理
    stop: function () {
      this.time.end = Date.now()
      this.isRunning = false
      cancelAnimationFrame(this.animationFrame)
    },

    // 初期化処理
    init: function () {
      this.stop()
      this.count = 0
      this.user = ''
      this.time.start = this.time.end = this.time.diff = null
      this.counter.chars = this.counter.backspace = 0
    },

    // 経過時間の計算
    elapsedTime: function () {
      if (this.time.start === null)
        return 0
      
      return this.time.diff / 1000
    },

    // Baskspace打鍵数のインクリメント
    incrementBackspaceCount: function () {
      ++this.counter.backspace
    }
  }
}
</script>

<style>
body {
  text-align: center;
}

label {
  background-color: lightcyan;
}

label::after {
  content: ':';
}

#display {
  background-color: lightgray;

  border: 1px solid black;
  border-radius: 0.5em;

  padding: 0.3em;

  left: auto;
  right: auto;
  
  width: 95%;
  font-size: 2em;
  text-align: left;
}

#user {
  width: 95%;
  height: 2.5em;
  font-size: 2em;
  text-align: left;
}

#diff {
  width: 95%;
  height: 5em;
}
</style>
