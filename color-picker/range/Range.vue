<template>
   <div>
      <div class="range__line" :style="{ background: 'linear-gradient(to right, #0099ff ' + dotLeft + '%, #ccc 0%)' }" ref="rangeLine" @click="setValue">
         <div class="dot" :style="{ left: dotLeft + '%' }" @mousedown="dotMouseDown" ref="dot" @click="stopProp"></div>
         <div class="value__block">
            <div class="overlay"></div>
            <svg>
               <circle class="circle_1" cx="50%" cy="50%" r="18"></circle>
               <circle class="circle_2" cx="50%" cy="50%" r="18" :stroke-dashoffset="-(113 - (113 / 100 * dotLeft))"></circle> 
            </svg>
            <div class="value">{{ Math.round(dotLeft) }}</div>   
         </div>
      </div>
   </div>
</template>

<style scoped>
   .range__line {
      max-width: var(--rangeMaxWidth, 100%);
      cursor: pointer;
      position: relative;
      height: var(--rangeHeight, 4px);
      border-radius: 5px;
      box-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5) inset; 
   }

   .dot {
      position: absolute;
      cursor: pointer;
      width: var(--thumbWidth, 20px);
      height: var(--thumbHeight, 20px);
      top: 50%;
      transform: translate(-50%, -50%);
      background: #0099ff;
      box-shadow: var(--thumbBoxShadow, -2px 4px 9px rgba(0,0,0,.5));
      border-radius: 50%;
   }

   .value__block {
      position: absolute;
      display: flex;
      align-items: center;
      justify-content: center;
      width: 50px;
      height: 50px;
      top: 50%;
      right: -62px;
      font-size: 16px;
      text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
      transform: translateY(-50%);
      pointer-events: none;
   }

   .overlay {
      position: absolute;
      border-radius: 50%;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      width: 70%;
      height: 70%;
      background: rgba(255, 255, 255, .7);
   }

   .value {
      position: relative;
      z-index: 5;
      right: 1px;
   }

   svg {
      position: absolute;
      display: block;
      width: 100%;
      height: 100%;
   }

   .circle_1,
   .circle_2 {
      stroke-width: 3px;
      fill: none;
   }

   .circle_1 {
      stroke: rgba(0, 0, 0, .1);
   }

   .circle_2 {
      stroke: #0099ff;
      transform: rotate(-90deg);
      stroke-dasharray: 1000;
      transform-origin: center;
   }
</style>

<script>
   export default {
      props: ['rangeValue'],
      data() {
         return {
            dotLeft: '',
            moveDot(e) {
               let line = this.$refs.rangeLine
               let dist = 100 / line.offsetWidth * (e.clientX - line.getBoundingClientRect().left)

               if(dist <= 0) {
                  this.dotLeft = 0
               }else if(dist >= 100) {
                  this.dotLeft = 100
               }else {
                  this.dotLeft = dist
               }
            }
         }
      },
      watch: {
         dotLeft(value) {
            this.$emit('onRange', value);
         }
      },
      methods: {
         stopProp(e) {
            e.stopPropagation()
         },
         dotMouseDown() {
            window.addEventListener('mousemove', this.moveDot);
         },
         setValue(e) {
            let line = this.$refs.rangeLine
            this.dotLeft = (100 / line.offsetWidth) * e.offsetX
         }
      },
      created() {
         this.moveDot = this.moveDot.bind(this);
      },
      mounted() {
         window.addEventListener('mouseup', () => {
            window.removeEventListener('mousemove', this.moveDot);
         });

         this.dotLeft = this.rangeValue
      }
   }
</script>