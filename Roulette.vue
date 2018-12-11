<template lang="pug">
  div.spinner-box
    div#spinner-items.spin(ref="tileContainer")
      .loader-wrap(v-if="spinner_items.length == 0")
        .loader
      div(v-else v-for="(item, index) in spinner_items" class="spinner-item" :style="'border-bottom: 2px solid ' + item.color" :key="index")
        img(:src="item.image['300px']")
        div.wear {{ itemWear(item.name) }}
        div.name {{ item.name.split('|')[0] }} | 
          span(:style="'color: ' + item.color") {{ item.name.split('|')[1].split('(')[0] }}
    div.pointer
    div.shadow-right
    div.shadow-left
</template>
<script>
export default {
  props: ['unboxed_skin', 'spinner_items', 'itemsLength', 'callback'],
  data() {
    return {
      tileWidth: null,
      winningPosition: null,
      offsetWidth: null,
      randomOffsetIntoTileSeed: null,
      randomOffsetIntoWinningTile: null,
      visibilityChangeProperty: null,
      hiddenProperty: null
    }
  },
  methods: {
    itemWear(itemName) {
      if (itemName.includes('Factory New')) return 'FN';
      else if (itemName.includes('Minimal Wear')) return 'MW';
      else if (itemName.includes('Field-Tested')) return 'FT';
      else if (itemName.includes('Well-Worn')) return 'WW';
      else if (itemName.includes('Battle-Scarred')) return 'BS';
      else return '';
    },
    calcOffsetIntoTile() {
      this.randomOffsetIntoWinningTile = (this.tileWidth / 10) * this.randomOffsetIntoTileSeed
      this.randomOffsetIntoWinningTile += (this.tileWidth / 10) / 2
    },
    determineWinningTilePosition() {
      // length of two loops into the roulette
      // find the winner tile index
      var startIndex = this.itemsLength * 2 - 1
      let winningIndex
      for(var i=startIndex; i < this.itemsLength * 3; i++){
        var item = this.spinner_items[i];
        if(this.unboxed_skin.name == item.name){
          winningIndex = i
          console.log('found the winningIndex show the unboxedSkin', this.unboxed_skin)
          break;
        }
      }
      return winningIndex * this.tileWidth
    },
    roulette() {
      // find tile width
      let spinnerTile = document.querySelector('.spinner-item')
      this.tileWidth = spinnerTile.offsetWidth
      this.winningPosition = this.determineWinningTilePosition()
      this.$refs.tileContainer.classList.add('spin')
      this.offsetWidth = Math.floor(document.querySelector('.spinner-box').offsetWidth / 2)

      this.offsetWidth = this.offsetWidth - this.winningPosition

      // split the winning tile into horizontal tenths, find a random tenth to land on except the first or last
      // rolls between 2-8, deduct one from roll we want to land the tenth before
      this.randomOffsetIntoTileSeed = ((Math.floor(Math.random() * (8 - 2 + 1)) + 2) - 1)
      this.calcOffsetIntoTile()
  
      this.offsetWidth -= this.randomOffsetIntoWinningTile
      this.$refs.tileContainer.addEventListener('transitionend', this.rouletteEnd)
      let translate = 'translate3d('+ (this.offsetWidth) + 'px, 0, 0)'
      this.$refs.tileContainer.style.transform = translate
    },
    rouletteEnd(e) {
      this.$refs.tileContainer.removeEventListener('transitionend', this.rouletteEnd);
      this.snapTileToWinning()
    },
    snapEnd(e) {
      this.$refs.tileContainer.removeEventListener('transitionend', this.snapEnd);
      this.moveRouletteToStart()
    },
    moveRouletteToStart() {
      // always get current tileWidth
      var spinnerTile = document.querySelector('.spinner-item')
      this.tileWidth = spinnerTile ? spinnerTile.offsetWidth : null
      this.calcOffsetIntoTile()

      this.$refs.tileContainer.classList.remove('snap')
      var startIndex = this.itemsLength - 1
      let winningIndex
      for(var i=startIndex; i < this.itemsLength * 2; i++){
        var item = this.spinner_items[i];
        if(this.unboxed_skin.name == item.name){
          winningIndex = i
          break;
        }
      }
      this.offsetWidth = Math.floor(document.querySelector('.spinner-box').offsetWidth / 2) - (winningIndex * this.tileWidth)
      this.offsetWidth -= this.tileWidth / 2

      let translate = 'translate3d('+ (this.offsetWidth) + 'px, 0, 0)'
      this.$refs.tileContainer.style.transform = translate
      this.callback()
    },
    snapTileToWinning() {
      this.$refs.tileContainer.addEventListener('transitionend', this.snapEnd)
      this.$refs.tileContainer.classList.add('snap')
      this.$refs.tileContainer.classList.remove('spin')
      let snapPosition = (Math.floor(document.querySelector('.spinner-box').offsetWidth / 2)) - this.winningPosition
      let translate = 'translate3d('+ (snapPosition - this.tileWidth / 2) + 'px, 0, 0)'
      this.$refs.tileContainer.style.transform = translate
    },
    completeStop() {
      this.$refs.tileContainer.removeEventListener('transitionend', this.rouletteEnd);
      this.$refs.tileContainer.removeEventListener('transitionend', this.snapEnd);
      this.moveRouletteToStart()
    },
    handleVisibilityChange() {
      // animation events dont fire in the background
      // so start a timer that checks if we should end the animation manually based on elapsed time
      // clear interval if the user returns to the tab while the animation is still running it should play out normally
      if(window.document[this.hiddenProperty]){
        // when user goes off tab remove animation
        if(this.running) {
        // stop the animation at current position
          this.$refs.tileContainer.removeEventListener('transitionend', this.rouletteEnd);
          this.$refs.tileContainer.removeEventListener('transitionend', this.snapEnd);

          var style = 'transform: translate3d('+ this.$refs.tileContainer.getBoundingClientRect().x + 'px, 0, 0);'
          this.$refs.tileContainer.setAttribute("style", style)
          
          var timeleft = (this.animationPredictedEnd + 1100) - Date.now()
          
          if(timeleft >= 0){
            this.timerId = setTimeout(()=>{
                this.completeStop()
            }, timeleft)
          } else {
            this.completeStop()
          }
        }
      } else {
        clearTimeout(this.timerId)
        if(this.running) {
          this.catchUp = true
          this.update()
        }
        this.timerId = null
      }
    }
  },
  mounted() {
    // Set the name of the hidden property and the change event for visibility
    if (typeof window.document.hidden !== "undefined") { // Opera 12.10 and Firefox 18 and later support 
      this.hiddenProperty = "hidden";
      this.visibilityChangeProperty = "visibilitychange";
    } else if (typeof window.document.msHidden !== "undefined") {
      this.hiddenProperty = "msHidden";
      this.visibilityChangeProperty = "msvisibilitychange";
    } else if (typeof window.document.webkitHidden !== "undefined") {
      this.hiddenProperty = "webkitHidden";
      this.visibilityChangeProperty = "webkitvisibilitychange";
    }
    window.document.addEventListener(this.visibilityChangeProperty, this.handleVisibilityChange, false);
  },
  beforeDestroy() {
    window.document.removeEventListener(this.visibilityChangeProperty, this.handleVisibilityChange, false);
  }
}
</script>
<style lang="stylus">
.unbox-stats-container
  display: flex;
  justify-content: space-between
  align-items center
  margin-top: 34px;
  .left-info
    display flex
  .right-info
    display flex
    align-items center
.loader-wrap
  height 100%
  display flex
  align-items center
#spinner-items
  &.spin
    will-change: transform
    transition-timing-function: cubic-bezier(0.12, 0.8, 0.38, 1)
    transition-duration: 10s
  &.snap
    transition all 1s ease-in 0.0s;
.spinner
  padding: 25px 0
  width: 100%
  position: relative
  text-align: center
.spinner-box
  height: 164px
  white-space: nowrap
  overflow: hidden
  position: relative
  border-radius: 2px
  background-color: #151722
#spinner-items
  height: 150px
  margin-top: 6.5px
.spinner-item:nth-of-type(odd)
  background-color #191b27
.spinner-item:nth-of-type(even)
  background-color #151722
.spinner-item
  display inline-block
  box-sizing border-box
  height 100%
  width 157px
  border-right 1px solid #292D3D
  position relative
  font-size 14px
.spinner-item img
  max-height 100%
  max-width 100%
.spinner-item .name
  font-size 14px
  position absolute
  bottom 10px
  width 90%
  margin-left 5%
  text-align center
  overflow hidden
.spinner-item .wear
  font-size 12px
  position absolute
  top 9px
  left 13px
  color #9194A7
.pointer
  height 100%
  width 2px
  background-color #ffffff
  position absolute
  left calc(50% - 1px)
  top 0
.shadow-left
  height 100%
  width 15%
  z-index 999
  position absolute
  left 0
  top 0
  background linear-gradient(to right, rgba(21,23,34,0.95) 0%,rgba(21,23,34,0.95) 3%,rgba(21,23,34,0) 100%)
.shadow-right
  height 100%
  width 16%
  z-index 999
  position absolute
  right 0
  top 0
  background linear-gradient(to left, rgba(21,23,34,0.95) 0%,rgba(21,23,34,0.95) 3%,rgba(21,23,34,0) 100%)
</style>