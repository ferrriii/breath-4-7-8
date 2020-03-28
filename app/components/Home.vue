<template>
  <Page class="page" actionBarHidden="true" @loaded="appLoaded">
    <FlexboxLayout flexDirection="column" backgroundColor="#36163a" justifyContent="space-around" height="100%">
      <FlexboxLayout justifyContent="center">
        <StackLayout class="reset" @tap="restart">
          <Label text="restart" color="white"/>
        </StackLayout>
      </FlexboxLayout>
      <Label class="action" :text="action" textAlignment="center" ref="action"></Label>
      <Label class="count" :text="count" key="cnt" textAlignment="center" ref="counter"></Label>
	  <Label @tap="openHomePage" class="about" text="⌂" textAlignment="center" ref="about"></Label>
    </FlexboxLayout>
  </Page>
</template>

<script>
  import * as app from 'tns-core-modules/application'
  import * as utilsModule from 'tns-core-modules/utils/utils'
  export default {
    data() {
      return {
        count: 4,
        timer: undefined,
        actionIndex: 0,
        actionCounts: [
          {name: 'Inhale', count: 4, vibration: [0, 50]},
          {name: 'Hold', count: 7, vibration: [0, 50, 100, 50]},
          {name: 'Exhale', count: 8, vibration: [0, 200]}
        ]
      }
    },
    computed: {
      action() {
        return this.actionCounts[this.actionIndex].name
      }
    },
    destroyed() {
      clearInterval(this.timer)
    },
    methods: {
      openHomePage() {
        utilsModule.openUrl('https://github.com/ferrriii/breath-4-7-8')
	  },
      onButtonTap() {
        let srv = app.android.context.getSystemService(android.content.Context.VIBRATOR_SERVICE)
        srv.vibrate(2000)
      },
      restart() {
        clearInterval(this.timer)
        this.timer = setInterval(this.tick, 1000)
        this.actionIndex = 0
        this.vibrate(this.actionCounts[this.actionIndex].vibration)
        this.count = this.actionCounts[this.actionIndex].count
      },
      appLoaded() {
        if (!this.timer)
          this.timer = setInterval(this.tick, 1000)
      },
      vibrate(param) {
        let srv = app.android.context.getSystemService(android.content.Context.VIBRATOR_SERVICE)
        if (!srv.hasVibrator()) return
        // Create the pattern array
        let pattern = Array.create('long', param.length)
        // Assign the pattern values
        param.forEach((value, index) => { pattern[index] = value; })
        // Vibrate pattern
        srv.vibrate(pattern, -1)
      },
      tick() {
        this.count--
        if (this.count == 0)
        {
          //switch to next action
          this.actionIndex++
          if (this.actionIndex >= this.actionCounts.length) this.actionIndex = 0
          this.vibrate(this.actionCounts[this.actionIndex].vibration)
          this.count = this.actionCounts[this.actionIndex].count

          let actionLayout = this.$refs.action.nativeView // Needed to avoid masking child components on Android
          var hide = actionLayout.createAnimation({ opacity: 0, duration: 300 })
          var show = actionLayout.createAnimation({ opacity: 1, duration: 500 })
          hide.play().then(function () {
            return show.play()
          })
        }

        let counterLayout = this.$refs.counter.nativeView // Needed to avoid masking child components on Android

        var grow = counterLayout.createAnimation({scale: { x: 1.5, y: 1.5}, duration: 10})
        var shrink = counterLayout.createAnimation({scale: { x: 1, y: 1}, duration: 980})
        grow.play()
          .then(function () { return shrink.play() })
          .catch(function (e) { })
      }
    }
  };
</script>

<style scoped lang="scss">

// Custom styles
.action {
  color: white;
  font-size: 60vmin;
  padding-left: 10vw;
  padding-right: 10vw;
  padding-top: 20vh;
  padding-bottom: 60vh;
}

.count {
  color: white;
  font-size: 30vmin;
}
  
.about {
  font-size: 15vmin;
  color: white;
}

.reset {
  width: 80vw;
  height: 50vw;
  border-color: white;
  border-width: 2px;
  border-radius: 50vw;
  vertical-align: center;
  text-align: center;
  font-size: 15vmin;
}
</style>
