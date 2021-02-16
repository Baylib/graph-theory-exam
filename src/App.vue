<template>
  <div id="app">
    <div class="md:flex grid grid-rows-3 grid-flow-col gap-4">
      <div class="controls row-span-3 ...">
        <div>
          <p>Toggle edit mode : {{ shiftKeySymbol }}</p>
          <p>Add node by clicking</p>
          <p>Click and drag to add an edge</p>
          <p>Delete Node : Delete</p>
        </div>
        <div>
            <label>Adjust width</label>
            <input type="range" v-model="settings.width" min="0" max="100" />
        </div>
        <div>
          <input type="checkbox" id="checkbox-directed" v-model="directed">
          <label v-if="directed" for="checkbox"> Undirected </label>
          <label v-else for="checkbox"> Directed </label>
          
        </div>
        <div>
          <button v-on:click="resetGraph">Reset Graph</button>
        </div>
      </div>
      <div class="svg-container col-span-2 ..." :style="{width: settings.width + '%'}">
        <svg pointer-events="all" viewBox="0 0 960 600" preserveAspectRatio="xMinYMin meet">
          <g :id="links"></g>
          <g :id="nodes"></g>
        </svg>
      </div>
    <div id="svg-adjancency-matrix" class="svg-container row-span-2 col-span-2" :style="{width: settings.width + '%'}">
    <GraphVue/>
    </div>
  </div>
  </div>
</template>

<script>
import * as d3 from 'd3'

import GraphVue from './components/GraphVue.vue'

export default {
  name: 'App',
  components: {
    GraphVue
  },
  data: function () {
                return {
                    graph: null,
                    simulation: null,
                    settings: {
                        strokeColor: "#29B5FF",
                        width: 100,
                        svgWigth: 960,
                        svgHeight: 600
                    },
                    selectedNode: null,
                    selectedLink:null,
                    selectedTargetNode : null,
                    edit_mode : false,
                    transformation : "",
                    directed:false,
                    shiftKeySymbol : `<shift>`
                }
            },
  mounted: function () {
    console.log("mounted");
    d3.select(window)
      .on("keydown", keydown)
    var svg = d3.select("svg");
    d3.select("#svg-adjancency-matrix").append("svg").style("pointer-events","all").attr("viewBox","0 0 960 600").attr("preserveAspectRatio","xMinYMin meet")
    var that = this;
    var graph={
        "nodes": [
            
        ],
        "links": [
            
        ]
    }
    that.graph = graph;
    var bigrect = svg.append("g")
      .append("rect")
      .attr("width",that.settings.svgWigth)
      .attr("height",that.settings.svgHeight)
      .attr("fill","#F5F5F5")
      .attr("stroke-width",that.settings.svgWigth/75)
      .attr("stroke","#DC143C")
      .attr("opacity",0)
      .on("mousedown", function () {
          if (that.edit_mode){ 
            var pointer = d3.pointer(event,svg.node());
            console.log("pointer :", pointer[0], pointer[1])
            that.transformation = d3.select(".nodes").attr("transform")
            console.log("transfo :::", that.transforms)
            var newNode = {
              x : (pointer[0] - that.transforms.translateX) /that.transforms.scaleX,
              y: (pointer[1] - that.transforms.translateY) / that.transforms.scaleY, 
              id: that.graph.nodes.length,
              "name": that.graph.nodes.length
            }
            that.graph.nodes.push(newNode);
          }
      })
      .call(d3.zoom()
        .on('zoom', (event) => {
          if (!that.edit_mode){
          svg.selectAll(".links").attr('transform', event.transform);
          svg.selectAll(".nodes").attr('transform', event.transform);
          }
        })
        .scaleExtent([-200, 200]))
      svg.append("g")
        .attr("class", "links")
      svg.append("g")
        .attr("class","nodes")
    

    that.simulation = d3.forceSimulation(that.graph.nodes)
        .force("link", d3.forceLink(that.graph.links).distance(50).strength(0.1))
        .force("charge", d3.forceManyBody())
        .force("center", d3.forceCenter(that.settings.svgWigth / 2, that.settings.svgHeight / 2)).stop();
        
      // });
    var editText = svg.append("text").attr("x", (that.settings.svgWigth / 2))             
              .attr("y", 0 + that.settings.svgWigth/75 + 25 )
              .attr("text-anchor", "middle")
              .style("fill","#DC143C") 
              .style("font-size", "2em") 
              .text("")
    function keydown(event) {
      console.log("keydown", event.keyCode)
      switch (event.keyCode) {
        case 16: { // shift
          that.edit_mode = !that.edit_mode;
          if (that.edit_mode){
            bigrect.attr("opacity",1)
            editText.text("EditMode")
              
          } else {
            bigrect.attr("opacity",0)
            editText.text("")
          }
          break;
        }
        case 46: {
          var i;
          if (that.selectedNode) { // deal with nodes
            i = that.graph.nodes.indexOf(that.selectedNode);
            that.graph.nodes.splice(i, 1);
            // find links to/from this node, and delete them too
            var new_links = [];
            that.graph.links.forEach(function(l) {
              if (l.source !== that.selectedNode && l.target !== that.selectedNode) {
                new_links.push(l);
              }
            });
            that.graph.links = new_links;
            that.graph.selectedNode = that.graph.nodes.length ? that.graph.nodes[i > 0 ? i - 1 : 0] : null;
            that.selectedNode = null
          } else if (that.selectedLink) { // deal with links
            i = that.graph.links.indexOf(that.selectedLink);
            that.graph.links.splice(i, 1);
            that.selectedLink = that.graph.links.length ? that.graph.links[i > 0 ? i - 1 : 0] : null;
          }
        }
      }
    }
    
    
  },
  computed: {
    
      nodes: function () {
          var that = this
          console.log("calculatednodes")
          if (that.graph) {
              d3.selectAll("circle").remove()
              return d3.select(".nodes")
                  .selectAll("circle")
                  .data(that.graph.nodes)
                  .enter().append("circle")
                  .attr("r", 5)
                  .attr("fill", "#5F9EA0")
                  .attr("stroke", "red")
                  .attr("stroke-opacity", function(d){if (d==that.selectedNode) {return "1" }else {return  "0"} })
                  .on("mousedown", function(event,d) {
                    console.log("on mouse down", event, d)
                    if (that.edit_mode) {
                      that.selectedNode = d
                      if (!d.active) that.simulation.alphaTarget(0).restart()
                      console.log("selected :", d)
                    }
                  })
                  .on("mouseover",function(event,d){
                    console.log("mouseovernode",event,d)
                    // pop up if not drag
                    if (that.edit_mode){
                      that.selectedTargetNode = d
                    }
                  })
                  .on("mouseout", function(){
                    if (that.edit_mode){
                    that.selectedTargetNode = null;
                    }
                  })
                  .call(d3.drag()
                      .on("start", function dragstarted(event, d) {
                        console.log("dragstart", d)
                        if (!that.edit_mode){
                          
                          if (!d.active) that.simulation.alphaTarget(0.3).restart()
                          d.fx = d.x
                          d.fy = d.y
                        } 
                      })
                      .on("drag", function dragged(event, d) {
                        //console.log("drag", event, d)
                        if (!that.edit_mode){ 
                          d.fx = event.x
                          d.fy = event.y
                        }
                      })
                      .on("end", function dragended(event, d) {
                        console.log("end", event, d)
                        if (!that.edit_mode){
                          
                          if (!d.active) that.simulation.alphaTarget(0)
                          d.fx = null
                          d.fy = null
                        } else {
                          if (d!= that.selectedTargetNode){
                            console.log("end drag", that.selectedNode)
                            if (that.selectedTargetNode == null){
                              var pointer = d3.pointer(event,d3.select("svg").node());
                              console.log("pointer :", pointer[0], pointer[1])
                              that.transformation = d3.select(".nodes").attr("transform")
                              console.log("transfo :::", that.transforms)
                              var newNode = {
                                x : (pointer[0] - that.transforms.translateX) /that.transforms.scaleX,
                                y: (pointer[1] - that.transforms.translateY) / that.transforms.scaleY, 
                                id: that.graph.nodes.length,
                                "name": that.graph.nodes.length
                              }
                              that.graph.nodes.push(newNode)
                              that.graph.links.push({"source":that.selectedNode,"target":newNode,"weight":1})
                            } else {
                                that.graph.links.push({"source":that.selectedNode,"target":that.selectedTargetNode,"weight":1})
                              console.log(that.graph.links)
                            }
                            that.selectedNode = null;
                            
                          }
                        }
                      })).on("mouseup",function(){console.log("on mouse up!")})
                      
          }
          return {}
      },
      links: function () {
          console.log("calculatedlinks")
          var that = this
          if (that.graph) {
              d3.selectAll("line").remove()
              return d3.select(".links")
                  .selectAll("line")
                  .data(that.graph.links)
                  .enter().append("line")
                  .attr("stroke-width", function (d) { return Math.sqrt(d.value); })
                  .attr("stroke-opacity", 0.6)
                  .attr("stroke", "#999")
          }
          return {}
      },
      transforms: function(){
        // Create a dummy g for calculation purposes only. This will never
        // be appended to the DOM and will be discarded once this function 
        // returns.
        var that = this;
        if (that.transformation == null || that.transformation == ""){
          return {
            translateX: 0,
            translateY: 0,
            rotate: 0,
            skewX: 0,
            scaleX: 1,
            scaleY: 1
          }
        }
        
        var g = document.createElementNS("http://www.w3.org/2000/svg", "g");
        
        // Set the transform attribute to the provided string value.
        g.setAttributeNS(null, "transform", that.transformation);
        
        // consolidate the SVGTransformList containing all transformations
        // to a single SVGTransform of type SVG_TRANSFORM_MATRIX and get
        // its SVGMatrix. 
        var matrix = g.transform.baseVal.consolidate().matrix;
        
        // Below calculations are taken and adapted from the private function
        // transform/decompose.js of D3's module d3-interpolate.
        var {a, b, c, d, e, f} = matrix;   // ES6, if this doesn't work, use below assignment
        // var a=matrix.a, b=matrix.b, c=matrix.c, d=matrix.d, e=matrix.e, f=matrix.f; // ES5
        var scaleX, scaleY, skewX;
        scaleX = Math.sqrt(a * a + b * b)
        skewX = a * c + b * d
        scaleY = Math.sqrt(c * c + d * d)
        if (scaleX){
          a /= scaleX, b /= scaleX;
        } 
        if (skewX) c -= a * skewX, d -= b * skewX;
        if (scaleY) c /= scaleY, d /= scaleY, skewX /= scaleY;
        if (a * d < b * c) a = -a, b = -b, skewX = -skewX, scaleX = -scaleX;
        return {
          translateX: e,
          translateY: f,
          rotate: Math.atan2(b, a) * 180 / Math.PI,
          skewX: Math.atan(skewX) * 180 / Math.PI,
          scaleX: scaleX,
          scaleY: scaleY
        };
      },
      //https://bl.ocks.org/KingOfCramers/32bbcd8c360e6d8aa0d5b7a50725fe73
      adjacencyMatrix: function(){
        let that = this
        console.log("calc adj matrix")
        d3.select("#svg-adjancency-matrix").select("svg").selectAll("g").remove()
        console.log("not recalc?", that.graph.nodes, that.graph.edges)
        
        let edgeHash = {};
        if (!that.graph.nodes){ return null}
        that.graph.links.forEach(edge =>{
          var id = edge.source.id + "-" + edge.target.id
          edgeHash[id] = edge
        })
            
        let matrix = []
        that.graph.nodes.forEach((source, a) => {
          that.graph.nodes.forEach((target, b) => {
            var grid = {id: source.id + "-" + target.id, x: b, y: a, weight: 0};
            
            if(edgeHash[grid.id]){
              var linkWeight =  edgeHash[grid.id].weight
              grid.weight = linkWeight;
              if (!that.directed) {
                var gridundirected = {id: target.id + "-" + source.id, x: a, y: b, weight: 0};
                gridundirected.weight = linkWeight;
                matrix.push(gridundirected)
              }
            }
            matrix.push(grid)
            
           })
        })
        console.log(matrix)
        return matrix
      }
  },
  updated: function () {
      console.log("updated")
      //console.log(that.links)
      var that = this;
      that.simulation.nodes(that.graph.nodes).on('tick', function ticked() {
          that.links
              .attr("x1", function (d) { return d.source.x; })
              .attr("y1", function (d) { return d.source.y; })
              .attr("x2", function (d) { return d.target.x; })
              .attr("y2", function (d) { return d.target.y; });

          that.nodes
              .attr("cx", function (d) { return d.x; })
              .attr("cy", function (d) { return d.y; });
      });
      that.simulation.alphaTarget(0).restart()


      //
  console.log("addmatrix in  update :", that.adjacencyMatrix);
  if (that.adjacencyMatrix){
    var svgAdjMatrix = d3.select("#svg-adjancency-matrix").select("svg") 
    console.log(svgAdjMatrix)
    svgAdjMatrix.append("g")
      .attr("transform","translate(50,50)")
      .attr("id","adjacencyG")
      .selectAll("rect")
      .data(that.adjacencyMatrix)
      .enter()
      .append("rect")
      .attr("class","grid")
      .attr("width",35)
      .attr("height",35)
      .attr("x", d=> d.x*35)
      .attr("y", d=> d.y*35)
      .style("fill-opacity", d=> d.weight * .2)
      .style("stroke","#9A8B7A")
      .style("stroke-width","1px")
      .style("fill","#CF7D1C")
      
    svgAdjMatrix.append("g")
      .attr("transform","translate(50,45)")
      .selectAll("text")
      .data(that.graph.nodes)
      .enter()
      .append("text")
      .attr("x", (d,i) => i * 35 + 17.5)
      .text(d => d.id)
      .style("text-anchor","middle")
      .style("font-size","10px")
      
      svgAdjMatrix.append("g").attr("transform","translate(45,50)")
      .selectAll("text")
      .data(that.graph.nodes)
      .enter()
      .append("text")
      .attr("y",(d,i) => i * 35 + 17.5)
      .text(d => d.id)
      .style("text-anchor","end")
      .style("font-size","10px")
      .attr("fill","#EBD8C1")
    }
  },
  methods:{
    resetGraph: function(){
      this.graph = {"nodes":[], "links":[]}
      d3.select("#svg-adjancency-matrix").select("svg").selectAll("g").remove()
    }
  },
  provide:{
    graphdata : 'Adjancency Matrix :'
  }
}

</script>

<style src="./assets/styles.css"></style>