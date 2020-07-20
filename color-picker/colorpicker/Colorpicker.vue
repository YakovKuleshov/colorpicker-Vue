<template>
   <div>
      <div class="wrapper__row">
         <div class="canv__wrapper">
            <canvas id="color__block" height="200" width="300" ref="canvas2" @mousedown="moveDot"></canvas>
            <div class="pointer" :style="{ top: top + 'px', left: left + 'px' }"></div>
         </div>
         <div class="wrapper" @mousedown="setColor">
            <canvas id="color__strip" height="200" width="35" ref="canvas"></canvas>
            <div class="arrow" :style="{ top: arrowTop + 'px' }"></div>
         </div>
      </div>
      <div class="value" title="Скопировать" @click="copy">{{ this.color }}
         <div class="done__arrow"></div>      
         <div class="final__color" :style="{ background: color }"></div>   
      </div> 
      <div class="value" title="Скопировать" @click="copy">{{ this.rgbaColor }}
         <div class="done__arrow"></div>      
         <div class="final__color" :style="{ background: rgbaColor }"></div>   
      </div>  
      <div class="opacity__range">Opacity
         <Range id="op__range" :rangeValue="alpha * 100" @onRange="setAlpha"/>   
      </div>    
   </div>
</template>

<style scoped>

   * {
      box-sizing: border-box;
   }
   
   .wrapper__row {
      display: flex;
      background: #fff;
   }

   #color__strip {
      cursor: pointer;
      height: 100%;
   }

   .arrow {
      cursor: pointer;
      position: absolute;
      width: 110%;
      height: 9px;
      border: 2px solid #333;
      border-radius: 3px;
      left: 50%;
      transform: translate(-50%, -50%);
   }

   .wrapper {
      position: relative;
   }

   .canv__wrapper {
      position: relative;
      margin-right: 5px;
   }

   #color__block {
      height: 100%;
      width: 100%;
   }

   .pointer {
      position: absolute;
      width: 10px;
      height: 10px;
      border-radius: 50%;
      border: 2px solid #333;
      box-sizing: border-box;
      transform: translate(-50%, -50%);
      background: #fff;
   }

   .value {
      display: flex;
      align-items: center;
      position: relative;
      cursor: pointer;
      font-weight: bold;
      color: #333;
      margin-top: 5px;
      user-select: auto;
      text-shadow: 1px 1px 2px rgba(0,0,0,.3);
   }

   .done__arrow {
      width: 7px;
      border-radius: 2px;
      height: 13px;
      border-bottom: 3px solid #0087FF;
      border-right: 3px solid #0087FF;
      transition: .3s;
      margin-left: 10px;
      transform-origin: bottom right;
      transform: rotate(45deg) scale(0);
   } 

   .final__color {
      width: 100px;
      height: 30px;
      margin-left: auto;
   }

   .opacity__range {
      font-size: 14px;
      font-weight: bold;
      display: flex;
      align-items: center;
      padding: 25px 0 20px;
   }

   #op__range {
      flex-grow: 1;
      margin-left: 16px;
      --rangeMaxWidth: 75%;
      --thumbWidth: 17px;
      --thumbHeight: 17px;
      --rangeHeight: 3px;
      --thumbBoxShadow: -1px 3px 5px rgba(0,0,0,.6);
   }

   .active .done__arrow {
      transform: rotate(45deg) scale(1);
   }
  
   </style>

<script>
import Range from '../range/Range'
   export default {
      components: {
         Range
      },
      data() {
         return {
            toHex(n) {
               n = parseInt(n,10);
               if (isNaN(n)) return "00";
               n = Math.max(0,Math.min(n,255));
               return "0123456789ABCDEF".charAt((n-n%16)/16)  + "0123456789ABCDEF".charAt(n%16);
            },
            colorsInit() {
               let canvas = this.$refs.canvas
               let ctx = canvas.getContext('2d');
               let width = canvas.width;
               let height = canvas.height;

               let canvas2 = this.$refs.canvas2
               let ctx2 = canvas2.getContext('2d');
               let width2 = canvas2.width;
               let height2 = canvas2.height;

               let grdWhite = ctx2.createLinearGradient(0, 0, width2, 0);
               let grdBlack = ctx2.createLinearGradient(0, 0, 0, height2);
                        
               ctx.drawImage(this.img, 0, 0, width, height);
                                           
               grdWhite.addColorStop(0, 'rgba(255,255,255,1)');
               grdWhite.addColorStop(1, 'rgba(255,255,255,0)');
               ctx2.fillStyle = grdWhite;
               ctx2.fillRect(0, 0, width2, height2);
               
               grdBlack.addColorStop(0, 'rgba(0,0,0,0)');
               grdBlack.addColorStop(1, 'rgba(0,0,0,1)');
               ctx2.fillStyle = grdBlack;
               ctx2.fillRect(0, 0, width2, height2);
            },
            color: '',
            rgbaColor: '',
            alpha: .3,
            top: 9,
            left: 290,
            arrowTop: 136
         }
      },
      watch: {
         rgbaColor(value) {
            this.$emit('setColor', value)
         }
      },
      methods: {
         setAlpha(value) {
            this.alpha = (value / 100).toFixed(2)
            this.finalColor()
         },
         moveDot(e) {
            this.left = e.offsetX
            this.top = e.offsetY
            this.finalColor(e)
            window.addEventListener('mousemove', this.finalColor);
         },
         setColor(e) {
            this.arrowTop = e.clientY - e.currentTarget.getBoundingClientRect().top
            window.addEventListener('mousemove', this.moveArrow)
            this.chooseColor()
            this.finalColor()
         },
         touchArrow(e) {
            
         },
         moveArrow(e) {
            let canvas = this.$refs.canvas
            let distTop = e.clientY - canvas.getBoundingClientRect().top

            if(distTop <= 4) {
               this.arrowTop = 4
            }else if (distTop >= canvas.offsetHeight - 5) {
               this.arrowTop = canvas.offsetHeight - 5
            }else {
               this.arrowTop = distTop
            }
            this.chooseColor()
            this.finalColor()
         },
         chooseColor(e) {
            let canvas2 = this.$refs.canvas2
            let ctx2 = canvas2.getContext('2d');
            let colorBlock = this.$refs.canvas;
            let ctx = colorBlock.getContext('2d');
            if(e) {
               this.arrowTop = e.offsetY;
            }
            let imageData = ctx.getImageData(0, this.arrowTop, 1, 1).data;
            // let rgbColor = 'rgba(' + imageData[0] + ',' + imageData[1] + ',' + imageData[2] + ',1)';
            let hexColor = '#' + this.toHex(imageData[0]) + this.toHex(imageData[1]) + this.toHex(imageData[2]);
            let grdBlack = ctx2.createLinearGradient(0, 0, 0, canvas2.height);    
            let grdWhite = ctx2.createLinearGradient(0, 0, canvas2.width, 0);
            
            ctx2.fillStyle = hexColor
            ctx2.fillRect(0, 0, canvas2.width, canvas2.height);
               // Градиент для левого блока
            grdWhite.addColorStop(0, 'rgba(255,255,255,1)');
            grdWhite.addColorStop(1, 'rgba(255,255,255,0)');
            ctx2.fillStyle = grdWhite;
            ctx2.fillRect(0, 0, canvas2.width, canvas2.height);
            
            grdBlack.addColorStop(0, 'rgba(0,0,0,0)');
            grdBlack.addColorStop(1, 'rgba(0,0,0,1)');
            ctx2.fillStyle = grdBlack;
            ctx2.fillRect(0, 0, canvas2.width, canvas2.height);           
         },
         finalColor(e) {
                       
            let colorBlock = this.$refs.canvas2;
            let ctx = colorBlock.getContext('2d');
            
            if(e) {
               let distTop = e.clientY - colorBlock.getBoundingClientRect().top
               let distLeft = e.clientX - colorBlock.getBoundingClientRect().left

               if(distLeft <= 0) {
                  this.left = 4
               }else if(distLeft >= colorBlock.offsetWidth) {
                  this.left = colorBlock.offsetWidth - 4 
               }else {
                  this.left = distLeft;
               }

               if(distTop <= 0) {
                  this.top = 4
               }else if(distTop >= colorBlock.offsetHeight) {
                  this.top = colorBlock.offsetHeight - 4
               }else {
                  this.top = distTop;
               }   
            }                   
          
            let imageData = ctx.getImageData(this.left, this.top, 1, 1).data;
            let rgbColor = 'rgba(' + imageData[0] + ',' + imageData[1] + ',' + imageData[2] + ',' + this.alpha + ')';
            let hexColor = '#' + this.toHex(imageData[0]) + this.toHex(imageData[1]) + this.toHex(imageData[2]);

            if(this.top == 4 && this.left == 4) {
               this.color = '#FFFFFF'
               this.rgbaColor = 'rgba(255,255,255,' + this.alpha + ')'
            }else {
               this.color = hexColor 
               this.rgbaColor = rgbColor
            }                        
         },
         copy(e) {
            let valueBlock = e.currentTarget
            let tmp = document.createElement("textarea");
            document.querySelector("body").append(tmp);
            tmp.value = valueBlock.innerText
            tmp.select()
            document.execCommand("copy");
            tmp.remove();
            
            valueBlock.classList.add('active');
            setTimeout(() => {
               valueBlock.classList.remove('active');
            }, 1000)
         }      
      },
      created() {
         this.finalColor = this.finalColor.bind(this);
         this.moveArrow = this.moveArrow.bind(this);
      },
      mounted() {
         this.img = new Image();
         this.img.src = '/src/elements/colorpicker/img/gradient.png';
       
         this.img.onload = () => {
            this.colorsInit()             
            this.chooseColor()
            this.finalColor()
         }
                  
         window.addEventListener('mouseup', e => {
            window.removeEventListener('mousemove', this.finalColor);
            window.removeEventListener('mousemove', this.moveArrow)          
         })
      }
   }
</script>