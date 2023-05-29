<template>
  <div>
    <div class="wrapper" ref="canvas"></div>
  </div>
</template>

<style>

  .wrapper{
    width: 32px;
    height: 32px;
  }
  
  canvas{
    width:  100%;
    height:  100%;
    background : transparent;
    image-rendering: pixelated; 
    image-rendering: crisp-edges;
  }

</style>


<script>

import { ref } from "vue"
import params from './params.json'

export default {
  props: {
    address: {
      type: String,
      required: true,
    },
  },

  components: {

  },

  data() {
    return {
      params : params,
    };
  },

  mounted() {
    this.draw()

  },

  methods: {

    draw(){

      console.log("draw head for ", this.address)

      const seed = this.md5(this.address)

      const data = {
        "address" : this.address,
        "seed": seed,

        "colors" : {
          "skin" : {
            "hue" :         this.processParam(this.params.colors.skin.hue,        this.pick(seed, 0) ),
            "saturation" :  this.processParam(this.params.colors.skin.saturation,   this.pick(seed, 1) ),
            "lightness" :   this.processParam(this.params.colors.skin.lightness,    this.pick(seed, 2) ),
            "variation" :   this.processParam(this.params.colors.skin.variation,    this.pick(seed, 3) ),
          },
          "hair" : {
            "hue" :         this.processParam(this.params.colors.hair.hue,      this.pick(seed, 1) ),
            "saturation" :  this.processParam(this.params.colors.hair.saturation,   this.pick(seed, 2) ),
            "lightness" :   this.processParam(this.params.colors.hair.lightness,    this.pick(seed, 3) ),
            "variation" :   this.processParam(this.params.colors.hair.variation,    this.pick(seed, 4) ),
          },

          "eyes" : {
            "hue" :         this.processParam(this.params.colors.eyes.hue,      this.pick(seed, 2) ),
            "saturation" :  this.processParam(this.params.colors.eyes.saturation,   this.pick(seed, 3) ),
            "lightness" :   this.processParam(this.params.colors.eyes.lightness,    this.pick(seed, 4) ),
            "variation" :   this.processParam(this.params.colors.eyes.variation,    this.pick(seed, 5) ),
          },      

          "pupils" : {
            "hue" :         this.processParam(this.params.colors.pupils.hue,      this.pick(seed, 3) ),
            "saturation" :  this.processParam(this.params.colors.pupils.saturation,   this.pick(seed, 4) ),
            "lightness" :   this.processParam(this.params.colors.pupils.lightness,  this.pick(seed, 5) ),
            "variation" :   this.processParam(this.params.colors.pupils.variation,  this.pick(seed, 6) ),
          },    

        },

        "layers" : {
          "skin" :          this.processParam({"min":0, "max": this.params.layers.skins.length },       this.pick(seed, 1) ),
          "hairstyle" :     this.processParam({"min":0, "max": this.params.layers.hairstyles.length },    this.pick(seed, 2) ),
          "eyes" :          this.processParam({"min":0, "max": this.params.layers.eyes.length },        this.pick(seed, 3) ),
          "pupils" :        this.processParam({"min":0, "max": this.params.layers.pupils.length },      this.pick(seed, 4) ),
        }
        
      }

      const canvas = document.createElement("canvas");
      const ctx = canvas.getContext("2d");
      ctx.canvas.width  = 8
      ctx.canvas.height = 8
      
      this.drawLayer(ctx, this.params.layers.skins[data.layers.skin], data.colors.skin , seed)
      this.drawLayer(ctx, this.params.layers.eyes[data.layers.eyes], data.colors.eyes , seed)
      this.drawLayer(ctx, this.params.layers.pupils[data.layers.pupils], data.colors.pupils , seed)
      this.drawLayer(ctx, this.params.layers.hairstyles[data.layers.hairstyle], data.colors.hair , seed)

      this.$refs.canvas.appendChild(canvas);

    },


    // draws a Layer ( 8x8 ) from pixel data in a color
    drawLayer(ctx, data, color, seed){
      const rows = data.split(",");
      rows.forEach( (row, r) => {
        const pixels = row.split("", 8);
        pixels.forEach( (pixel, p) => {
          if (pixel == 1) { this.drawPixel(ctx, p, r, color, seed ); }
        });
      });

      return ctx;
    },

    drawPixel(ctx, x, y, color, seed){

      let rnd1 = this.pick(seed, 1)
      let rnd2 = this.pick(seed, 2)
      let variation =  ( x + rnd1 + y + rnd2 ) * color.variation

      let hsl = "hsl(" +   ( (color.hue + variation) )  + "," + color.saturation + "%, " + color.lightness + "%)"
      ctx.beginPath();
      ctx.rect(x, y, 1, 1);
      ctx.fillStyle = hsl;
      ctx.fill();
      ctx.closePath();

    },

    pick( seed, pos, lenght = 3 ){
      return this.hex2dec(  seed.substring( pos, pos + lenght ) )
    },

    hex2dec(hex){
      return parseInt(hex, 16)
    },

    processParam(param, value) {
      return param.min + (value % ( param.max - param.min ));
    },

    md5(inputString) {
        var hc="0123456789abcdef";
        function rh(n) {var j,s="";for(j=0;j<=3;j++) s+=hc.charAt((n>>(j*8+4))&0x0F)+hc.charAt((n>>(j*8))&0x0F);return s;}
        function ad(x,y) {var l=(x&0xFFFF)+(y&0xFFFF);var m=(x>>16)+(y>>16)+(l>>16);return (m<<16)|(l&0xFFFF);}
        function rl(n,c)            {return (n<<c)|(n>>>(32-c));}
        function cm(q,a,b,x,s,t)    {return ad(rl(ad(ad(a,q),ad(x,t)),s),b);}
        function ff(a,b,c,d,x,s,t)  {return cm((b&c)|((~b)&d),a,b,x,s,t);}
        function gg(a,b,c,d,x,s,t)  {return cm((b&d)|(c&(~d)),a,b,x,s,t);}
        function hh(a,b,c,d,x,s,t)  {return cm(b^c^d,a,b,x,s,t);}
        function ii(a,b,c,d,x,s,t)  {return cm(c^(b|(~d)),a,b,x,s,t);}
        function sb(x) {
            var i;var nblk=((x.length+8)>>6)+1;var blks=new Array(nblk*16);for(i=0;i<nblk*16;i++) blks[i]=0;
            for(i=0;i<x.length;i++) blks[i>>2]|=x.charCodeAt(i)<<((i%4)*8);
            blks[i>>2]|=0x80<<((i%4)*8);blks[nblk*16-2]=x.length*8;return blks;
        }
        var i,x=sb(inputString),a=1732584193,b=-271733879,c=-1732584194,d=271733878,olda,oldb,oldc,oldd;
        for(i=0;i<x.length;i+=16) {olda=a;oldb=b;oldc=c;oldd=d;
            a=ff(a,b,c,d,x[i+ 0], 7, -680876936);d=ff(d,a,b,c,x[i+ 1],12, -389564586);c=ff(c,d,a,b,x[i+ 2],17,  606105819);
            b=ff(b,c,d,a,x[i+ 3],22,-1044525330);a=ff(a,b,c,d,x[i+ 4], 7, -176418897);d=ff(d,a,b,c,x[i+ 5],12, 1200080426);
            c=ff(c,d,a,b,x[i+ 6],17,-1473231341);b=ff(b,c,d,a,x[i+ 7],22,  -45705983);a=ff(a,b,c,d,x[i+ 8], 7, 1770035416);
            d=ff(d,a,b,c,x[i+ 9],12,-1958414417);c=ff(c,d,a,b,x[i+10],17,     -42063);b=ff(b,c,d,a,x[i+11],22,-1990404162);
            a=ff(a,b,c,d,x[i+12], 7, 1804603682);d=ff(d,a,b,c,x[i+13],12,  -40341101);c=ff(c,d,a,b,x[i+14],17,-1502002290);
            b=ff(b,c,d,a,x[i+15],22, 1236535329);a=gg(a,b,c,d,x[i+ 1], 5, -165796510);d=gg(d,a,b,c,x[i+ 6], 9,-1069501632);
            c=gg(c,d,a,b,x[i+11],14,  643717713);b=gg(b,c,d,a,x[i+ 0],20, -373897302);a=gg(a,b,c,d,x[i+ 5], 5, -701558691);
            d=gg(d,a,b,c,x[i+10], 9,   38016083);c=gg(c,d,a,b,x[i+15],14, -660478335);b=gg(b,c,d,a,x[i+ 4],20, -405537848);
            a=gg(a,b,c,d,x[i+ 9], 5,  568446438);d=gg(d,a,b,c,x[i+14], 9,-1019803690);c=gg(c,d,a,b,x[i+ 3],14, -187363961);
            b=gg(b,c,d,a,x[i+ 8],20, 1163531501);a=gg(a,b,c,d,x[i+13], 5,-1444681467);d=gg(d,a,b,c,x[i+ 2], 9,  -51403784);
            c=gg(c,d,a,b,x[i+ 7],14, 1735328473);b=gg(b,c,d,a,x[i+12],20,-1926607734);a=hh(a,b,c,d,x[i+ 5], 4,    -378558);
            d=hh(d,a,b,c,x[i+ 8],11,-2022574463);c=hh(c,d,a,b,x[i+11],16, 1839030562);b=hh(b,c,d,a,x[i+14],23,  -35309556);
            a=hh(a,b,c,d,x[i+ 1], 4,-1530992060);d=hh(d,a,b,c,x[i+ 4],11, 1272893353);c=hh(c,d,a,b,x[i+ 7],16, -155497632);
            b=hh(b,c,d,a,x[i+10],23,-1094730640);a=hh(a,b,c,d,x[i+13], 4,  681279174);d=hh(d,a,b,c,x[i+ 0],11, -358537222);
            c=hh(c,d,a,b,x[i+ 3],16, -722521979);b=hh(b,c,d,a,x[i+ 6],23,   76029189);a=hh(a,b,c,d,x[i+ 9], 4, -640364487);
            d=hh(d,a,b,c,x[i+12],11, -421815835);c=hh(c,d,a,b,x[i+15],16,  530742520);b=hh(b,c,d,a,x[i+ 2],23, -995338651);
            a=ii(a,b,c,d,x[i+ 0], 6, -198630844);d=ii(d,a,b,c,x[i+ 7],10, 1126891415);c=ii(c,d,a,b,x[i+14],15,-1416354905);
            b=ii(b,c,d,a,x[i+ 5],21,  -57434055);a=ii(a,b,c,d,x[i+12], 6, 1700485571);d=ii(d,a,b,c,x[i+ 3],10,-1894986606);
            c=ii(c,d,a,b,x[i+10],15,   -1051523);b=ii(b,c,d,a,x[i+ 1],21,-2054922799);a=ii(a,b,c,d,x[i+ 8], 6, 1873313359);
            d=ii(d,a,b,c,x[i+15],10,  -30611744);c=ii(c,d,a,b,x[i+ 6],15,-1560198380);b=ii(b,c,d,a,x[i+13],21, 1309151649);
            a=ii(a,b,c,d,x[i+ 4], 6, -145523070);d=ii(d,a,b,c,x[i+11],10,-1120210379);c=ii(c,d,a,b,x[i+ 2],15,  718787259);
            b=ii(b,c,d,a,x[i+ 9],21, -343485551);a=ad(a,olda);b=ad(b,oldb);c=ad(c,oldc);d=ad(d,oldd);
        }
        return rh(a)+rh(b)+rh(c)+rh(d);
    },


  },

}

</script>