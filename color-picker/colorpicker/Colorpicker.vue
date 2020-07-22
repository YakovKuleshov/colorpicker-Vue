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
      <div class="value" title="Скопировать">
         <input class="hex__input" type="text" v-bind:value="colorValue ? colorValue : color" maxlength="7" @input="writeColor">
         <div class="done__arrow"></div>      
         <div class="final__color" :style="{ background: color[0] == '#' ? color : '#' + color }"></div>   
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

   .hex__input {
      outline: none;
      border: none;
      color: #333;
      font-size: 16px;
      font-weight: bold;
      max-width: 90px;
      text-shadow: 1px 1px 2px rgba(0,0,0,.3);
      text-transform: uppercase;
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
      props: ['comesColor'],
      components: {
         Range
      },
      data() {
         return {
            colorValue: '',
            color: '',
            rgbaColor: '',
            alpha: .3,
            top: '',
            left: '',
            arrowTop: '',
            coordsByColor(color) {
               let canvas2 = this.$refs.canvas2
               let canvas = this.$refs.canvas
               let rgb = this.hexToRgb(color)                                                         
               let hsv = this.rgbToHsv(rgb.r, rgb.g, rgb.b);
               
               if((canvas2.width * hsv.s) == 0) {
                  this.left = 3
               }else {
                  this.left = (canvas2.width * hsv.s)
               }

               this.top = canvas2.height * (1 - hsv.v)

               if((canvas.height * (1 - hsv.h / 360)) == 0) {
                  this.arrowTop = 5
               }else if( (canvas.height * (1 - hsv.h / 360)) == canvas.height) {
                  this.arrowTop = 199
               }else {
                  this.arrowTop = (canvas.height * (1 - hsv.h / 360))
               }
               this.finalColor()
            },
            toHex(n) {
               n = parseInt(n,10);
               if (isNaN(n)) return "00";
               n = Math.max(0,Math.min(n,255));
               return "0123456789ABCDEF".charAt((n-n%16)/16) + "0123456789ABCDEF".charAt(n%16);
            },
            hexToRgb(hex) {
               let result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
               return result ? {
                  r: parseInt(result[1], 16),
                  g: parseInt(result[2], 16),
                  b: parseInt(result[3], 16)
               } : null;
            },
            rgbToHsv(r, g, b) {
               r /= 255, g /= 255, b /= 255;
               let max = Math.max(r, g, b), min = Math.min(r, g, b);
               let h, s, v = max;
               let d = max - min;
               s = max == 0 ? 0 : d / max;

               if (max == min) {
                  h = 0;
               } else {
                  switch (max) {
                     case r: h = (g - b) / d + (g < b ? 6 : 0); break;
                     case g: h = (b - r) / d + 2; break;
                     case b: h = (r - g) / d + 4; break;
                  }
                  h /= 6;
               }
               return { h: h * 360, s: s, v: v };
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

               ctx.rect(0, 0, width, height);
               var grd1 = ctx.createLinearGradient(0, 0, 0, height);
               
               grd1.addColorStop(0,    "rgb(255,   0,   0)");
               grd1.addColorStop(0.15, "rgb(255,   0, 255)");
               grd1.addColorStop(0.33, "rgb(0,     0, 255)");
               grd1.addColorStop(0.50, "rgb(0,   255, 255)");
               grd1.addColorStop(0.67, "rgb(0,   255,   0)");
               grd1.addColorStop(0.81, "rgb(255, 255,   0)");
               grd1.addColorStop(1,    "rgb(255,   0,   0)");
            
               ctx.fillStyle = grd1;
               ctx.fill();
                                                     
               grdWhite.addColorStop(0, 'rgba(255,255,255,1)');
               grdWhite.addColorStop(1, 'rgba(255,255,255,0)');
               ctx2.fillStyle = grdWhite;
               ctx2.fillRect(0, 0, width2, height2);
               
               grdBlack.addColorStop(0, 'rgba(0,0,0,0)');
               grdBlack.addColorStop(1, 'rgba(0,0,0,1)');
               ctx2.fillStyle = grdBlack;
               ctx2.fillRect(0, 0, width2, height2);
            }
         }
      },
      watch: {
         rgbaColor(rgba) {
            this.$emit('onRgbaColor', rgba)
         }
      },
      methods: {
         writeColor(e) {
            this.colorValue = e.target.value[0] == '#' ? e.target.value : '#' + e.target.value
            let result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(this.colorValue);

            if(result && result.length) {
               this.coordsByColor(this.colorValue)
            }       
            this.$emit('onInputColor', this.colorValue);
         },
         setAlpha(value) {
            this.alpha = (value / 100).toFixed(2)
            this.finalColor()
         },
         moveDot(e) {
            this.colorValue = ''
            this.left = e.offsetX
            this.top = e.offsetY
            this.finalColor(e)
            window.addEventListener('mousemove', this.finalColor);
         },
         setColor(e) {
            this.colorValue = ''
            this.arrowTop = e.clientY - e.currentTarget.getBoundingClientRect().top
            window.addEventListener('mousemove', this.moveArrow)
            this.finalColor()
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
            this.chooseColor()
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
         this.colorValue = this.comesColor ? this.comesColor : '#2B6DCF'
         this.colorsInit()           
         this.coordsByColor(this.comesColor ? this.comesColor : this.colorValue) 
                                                                 
         window.addEventListener('mouseup', e => {
            window.removeEventListener('mousemove', this.finalColor);
            window.removeEventListener('mousemove', this.moveArrow)          
         })
      }
   }
</script>
