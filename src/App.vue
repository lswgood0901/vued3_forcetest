<template>
  <div id="app">
  </div>
</template>

<script>
import * as d3 from 'd3' // eslint-disable-line no-unused-vars

export default {
  name: 'App',
  components: {
  },
  data() {
    return {
    svg : null,
    svgMap : null,
    svgmini : null,
    miniMap : null,
    minimapScatter : null,
    width : 300,
    height: 300,
    simulation : null,
    miniSimulation: null,
    newSimulation: null,
    svgBrush: null,
    k : 1,
    tx : 0,
    ty :0,
    scatter: null,
    node: [
      {"name":"A",
      "id":"A0",
      x : 20,
      y : 40},
      {"name": "B",
      "id":"B0",
      x : 100,
      y : 100},
      {"name": "C",
      "id":"C0",
      x : 300,
      y : 300},
      {"name": "D",
      "id":"D0",
      x : 250,
      y : 10},
      {"name": "E",
      "id":"E0",
      x : 10,
      y : 300},
      {"name": "F",
      "id":"F0",
      x : 48,
      y : 12},
      {"name": "G",
      "id":"G0",
      x : 48,
      y : 178},
      {"name": "H",
      "id":"H0",
      x : 90,
      y : 178},
      {"name": "I",
      "id":"I0",
      x : 67,
      y : 117},
      {"name": "J",
      "id":"J0",
      x : 242,
      y : 105}
      ],
    arrayK : [0,0,0],
    arrayTx:[0,0,0,0,0],
    arrayTy:[0,0,0,0,0],
    zooms: null,
    brush: null,
    i : 0,
    bbox: null,
    arrayBbox: [],
    signal : null,
    eventType : null,
    tFlag: null,
    bFlag: true,
    userActionStatus: [0,0,0,0,0],
    previousUserStatus: null,
    }
  },
  methods: {
    
    isBrushed: (brushCoords, cx, cy)=>{ // eslint-disable-line no-unused-vars
      const x0 = brushCoords[0][0];
      const x1 = brushCoords[1][0];
      const y0 = brushCoords[0][1];
      const y1 = brushCoords[1][1];
      return x0 > cx || cx > x1 || y0 > cy || cy > y1;
    },
    zoomOut: (x,y,k,svg,zoom)=>{
      if(k[k.length-1] < k[k.length-2])
        {
          svg.call(
            zoom.transform,
            d3.zoomIdentity.translate(x[x.length-1], y[y.length-1]).scale(k[k.length-1]))
        } svg.call(
            zoom.transform,
            d3.zoomIdentity.translate(x[x.length-2], y[y.length-2]).scale(k[k.length-2]))
          // .on('mousedown.zoom', null)
    },
    zoomIn: (x,y,k,svg,zoom)=>{
      if(k[k.length-1] > k[k.length-2])
        {
          svg.call(
            zoom.transform,
            d3.zoomIdentity.translate(x[x.length-1], y[y.length-1]).scale(k[k.length-1]))
        } svg.call(
            zoom.transform,
            d3.zoomIdentity.translate(x[x.length-2], y[y.length-2]).scale(k[k.length-2]))
          // .on('mousedown.zoom', null)
    },
    panning: (x,y,k,svg,zoom)=>{
      console.log(k[k.length-1] == k[k.length-2] || (k[0]==1 && k[1]== undefined))
      if(k[k.length-1] == k[k.length-2] || (k[0]==1 && k[1]== undefined))
        {
        svg.call(
          zoom.transform,
          d3.zoomIdentity.translate(x[x.length-1], y[y.length-1]).scale(k[k.length-1]))
          // svg.on('wheel.zoom', (event)=>{
          //   event.preventDefault();
          // })
        } svg.call(
            zoom.transform,
            d3.zoomIdentity.translate(x[x.length-2], y[y.length-2]).scale(k[k.length-2]))
    },
    
  },
  
  watch: {

    i(){
        let d3length = d3.selectAll('.inbox')._groups[0].length
        let eachCx = []
        let eachCy = []
        for(let step = 0; step < d3length; step++){
          eachCx.push(d3.selectAll('.inbox')._groups[0][step].cx.baseVal.value)
          eachCy.push(d3.selectAll('.inbox')._groups[0][step].cy.baseVal.value)
        }
        let selectedBBox = [[Math.min(...eachCx), Math.min(...eachCy)], [Math.max(...eachCx), Math.max(...eachCy)]]
        let bboxX = selectedBBox[0][0] - 10
        let bboxY = selectedBBox[0][1] - 10
        let bboxHeight = Math.max(selectedBBox[1][0]-selectedBBox[0][0], selectedBBox[1][1]-selectedBBox[0][1]) + 20 // eslint-disable-line no-unused-vars
        const scaleX = (k)=>{ // eslint-disable-line no-unused-vars
          return d3.scaleLinear([0,this.width],[0,this.width * k])
        }
        const scaleY = (k)=>{ // eslint-disable-line no-unused-vars
          return d3.scaleLinear([0,this.height],[0,this.height * k])
        }

        this.bFlag = false
        this.gCam.attr("transform", "translate("+ scaleX(this.k)(-bboxX)+"," +scaleY(this.k)(-bboxY) +")" +"scale("+this.k+")")
        this.svgMap.call(
        this.zooms.transform,
        d3.zoomIdentity.translate(scaleX(this.width / bboxHeight)(-bboxX), scaleY(this.width / bboxHeight)(-bboxY)).scale(this.width / bboxHeight)
        )
        this.bFlag = true
        this.arrayK=[0,0,0]
        eachCx = []
        eachCy = []
        selectedBBox = []
        console.log("보정")
        console.log('aa')
    }
  },



  mounted() {

    setInterval(()=>{
      this.userActionStatus.unshift(0)
      this.userActionStatus.pop()
      
      for(let step = 1; step < 3; step++){
        if(this.userActionStatus[step]==1){
          this.tFlag = true          
       }
      }
      if(this.tFlag == true && this.userActionStatus[0]==0 && 
        this.userActionStatus[1]==0 && this.userActionStatus[2]==0 &&
        this.userActionStatus[3]==0 && this.userActionStatus[4]==0){
        this.bFlag = true
        this.i = this.i + 1
        this.tFlag = false
        console.log('시간보정')
      }
    }, 300)


    this.svg = d3.select('#app')
      .append('svg')
      .attr('width', 500)
      .attr('height', 500)
    this.svgmini = d3.select('#app')
      .append('svg')
      .attr('width', 500)
      .attr('height', 500)
    this.svgMap = this.svg
      .append('svg')
      .attr('width', this.width)
      .attr('height', this.height)
      .attr('class', 'bbox')
    this.miniMap = this.svgmini
      .append('svg')
      .attr('width', this.width)
      .attr('height', this.height)
      .attr('class', 'miniBbox')
    
    let startTime = Date.now()
    let endTime = startTime + 1000

    const ticked = ()=>{
      if(Date.now() < endTime){
        return this.scatter
        .attr('cx', (d)=>{ 
          return d.x
        })
        .attr('cy', (d)=> {
          return d.y
        })
        .on("click", (d)=>{
          console.log("hi", d.target)
        })
      } else {
        this.simulation.stop()
      }     
    }
    
    const miniMapTicked = ()=>{ // eslint-disable-line no-unused-vars
      if(Date.now() < endTime){
        return this.minimapScatter
        .attr('cx', (d)=>{ 
          return d.x
        })
        .attr('cy', (d)=> {
          return d.y
        })
        .on("click", (d)=>{
          console.log("hi", d.target)
        })
      } else {
        this.miniSimulation.stop()
      }
    }

    this.gCam = this.svgMap.append('g')
    this.scatter = this.gCam
      .selectAll('circle')
      .data(this.node)
      .join('circle')
      .attr("r", 10)
      .attr("id", (d)=>{
        return d.id
      })

    this.minimapScatter = this.miniMap // eslint-disable-line no-unused-vars
      .selectAll('g')
      .data(this.node)
      .join('g')
      .append('circle')
      .attr("r", 10)
      .attr("id", (d)=>{
        return d.id
      })

    this.simulation = d3.forceSimulation(this.node) // eslint-disable-line no-unused-vars
      .force('charge', d3.forceManyBody().strength(90))
      .force('center', d3.forceCenter(this.width / 2, this.height / 2))
      .force('collision', d3.forceCollide().radius(15))
      .on('tick', ticked)

    this.miniSimulation =  d3.forceSimulation(this.node) // eslint-disable-line no-unused-vars
      .force('charge', d3.forceManyBody().strength(90))
      .force('center', d3.forceCenter(this.width / 2, this.height / 2))
      .force('collision', d3.forceCollide().radius(15))
      .on('tick', miniMapTicked)
    
    this.onZoom = (event)=>{
      if ((event.type == 'brush')) 
        {return null}

      this.k = event.transform.k // eslint-disable-line no-unused-vars
      this.tx = event.transform.x // eslint-disable-line no-unused-vars
      this.ty = event.transform.y // eslint-disable-line no-unused-vars
      if(this.bFlag == true){
        this.userActionStatus.unshift(1)
        this.userActionStatus.pop()
        this.arrayK.unshift(this.k)
        this.arrayK.pop()
        console.log()
        if(this.arrayK[2]>this.arrayK[1] && this.arrayK[1]>this.arrayK[0] == false &&
          this.arrayK[0] !=0 && this.arrayK[1] != 0 && this.arrayK[2] != 0){
          console.log(this.arrayK[0])
          console.log('줌아웃 변환 보정')
          this.i = this.i + 1
        }
        else if(this.arrayK[2]<this.arrayK[1] && this.arrayK[1]<this.arrayK[0] == false &&
          this.arrayK[0] !=0 && this.arrayK[1] != 0 && this.arrayK[2] != 0){
          console.log(this.arrayK[0])
          console.log('줌인 변환 보정')
          this.i = this.i + 1
        }
        else if(this.arrayK[2]==this.arrayK[1] && this.arrayK[1]!=this.arrayK[0] &&
          this.arrayK[0] !=0 && this.arrayK[1] != 0 && this.arrayK[2] != 0){
          console.log(this.arrayK[0])
          console.log('패닝 변환 보정')
          this.i = this.i + 1
        }
      }

  
      // if(this.arrayK[2]<this.arrayK[1] && this.arrayK[1]<this.arrayK[0] == false){
      //     console.log('변환 보정')
      //     this.i = this.i + 1
      //   }
      // if(this.arrayK[2]==this.arrayK[1] && this.arrayK[1]==this.arrayK[0] == false){
      //     console.log('변환 보정')
      //     this.i = this.i + 1
      //   }
       
      this.gCam.attr('transform', event.transform)
      
      const scaleX = (k)=>{ // eslint-disable-line no-unused-vars
        return d3.scaleLinear([0,this.width],[0,this.width * k])
      }
      const scaleY = (k)=>{ // eslint-disable-line no-unused-vars
        return d3.scaleLinear([0,this.height],[0,this.height * k])
      }
      this.brush.move(this.svgBrush, [
        [scaleX(this.k).invert(-this.tx), scaleY(this.k).invert(-this.ty)],
        [scaleX(this.k).invert(-this.tx + this.width), scaleY(this.k).invert(-this.ty + this.height)],
      ])
    }

    this.zooms = d3.zoom()
      .on('zoom', this.onZoom)
      
    this.svgBrush = this.miniMap
      .append('g')

    this.onBrush = (event)=>{
      if (event.type == 'zoom') 
        {return null;}

      if (Array.isArray(event.selection)){
        this.brushX = event.selection[0][0]
        this.brushY = event.selection[0][1]
        this.minimapScatter.classed("outofBox", (d)=>{ // eslint-disable-line no-unused-vars
          return this.isBrushed(event.selection, d.x, d.y)
        })
        this.minimapScatter.classed("inbox", (d)=>{ // eslint-disable-line no-unused-vars
          return !this.isBrushed(event.selection, d.x, d.y)
        })
        this.scatter.classed("clientGrey", (d)=>{ // eslint-disable-line no-unused-vars
          return this.isBrushed(event.selection, d.x, d.y)
        })
        this.zoomscale = d3.zoomTransform(this.svgMap.node()).k
   
        // const scaleX = (k)=>{ // eslint-disable-line no-unused-vars
        // return d3.scaleLinear([0,this.width],[0,this.width * k])
        // }
        // const scaleY = (k)=>{ // eslint-disable-line no-unused-vars
        // return d3.scaleLinear([0,this.height],[0,this.height * k])
        // }
        // this.svgMap.call(
        //   this.zooms.transform,
        //   d3.zoomIdentity.translate(scaleX(this.zoomscale)(-this.brushX), scaleY(this.zoomscale)(-this.brushY)).scale(this.zoomscale)
        // )
        // this.gCam.attr("transform", "translate("+ scaleX(this.zoomscale)(-this.brushX)+"," +scaleY(this.zoomscale)(-this.brushY) +")" +"scale("+this.zoomscale+")")
       
      }
    }

    this.brush = d3.brush()
      .extent([[0,0], [this.width, this.height]])
      .on("brush", this.onBrush)

    this.svgMap.call(this.zooms)
      .on("dblclick.zoom", null)
      
    this.svgBrush.call(this.brush)
    this.brush.move(this.svgBrush, [
      [0,0],
      [this.width * this.k, this.height * this.k]
    ])
    this.miniMap.selectAll('.handle').remove()
    this.miniMap.selectAll('.overlay').remove()
 
  },

 
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
circle {
  fill: cadetblue;
}
.outofBox {
  fill: red;
  opacity: 1;
}
.inbox {
  fill: rgb(0, 183, 255);
  opacity: 1;
}
.clientGrey{
  fill: grey;
  opacity: 0.7;
}

</style>
