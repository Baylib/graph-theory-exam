<template>
  <div id="app">
    <div class=flex>
      <div class="controls flex-none">
        <div>
            <label>Adjust width</label>
            <input type="range" v-model="settings.width" min="0" max="100" />
        </div>
      </div>
      <div class="svg-container flex-grow" :style="{width: settings.width + '%'}">
        <svg id="svg" pointer-events="all" viewBox="0 0 960 600" preserveAspectRatio="xMinYMin meet">
          <g :id="links"></g>
          <g :id="nodes"></g>
        </svg>
      </div>
    </div>
  </div>
</template>

<script>
import * as d3 from 'd3'

export default {
  name: 'App',
  components: {
  },
  data: function () {
                return {
                    graph: null,
                    simulation: null,
                    color: d3.scaleOrdinal(d3.schemeCategory10),
                    settings: {
                        strokeColor: "#29B5FF",
                        width: 100,
                        svgWigth: 960,
                        svgHeight: 600
                    },
                    selectedNode: null,
                    edit_mode : false,
                };
            },
            mounted: function () {
              console.log("mounted");
              d3.select(window)
                .on("keydown", keydown)
              var svg = d3.select("svg");
              var that = this;
              var graph={
                  "nodes": [
                      {
                          "id": "Alice"
                      },
                      {
                          "id": "Bob"
                      },
                      {
                          "id": "Carol"
                      }
                  ],
                  "links": [
                      {
                          "source": 0,
                          "target": 1
                      },
                      {
                          "source": 1,
                          "target": 2
                      }
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
                      var trans = getTransformation(d3.select(".nodes").attr("transform"))
                      var transX = trans.translateX
                      var transY = trans.translateY
                      var scaleX = trans.scaleX
                      var scaleY = trans.scaleY
                      //console.log(transX,transY,scaleY,scaleX)
                      //console.log((pointer[0] - transX) /scaleX);
                      //that.graph.nodes.push({x : pointer[0], y: pointer[1], id: that.graph.nodes.length});
                      that.graph.nodes.push({x : (pointer[0] - transX) /scaleX, y: (pointer[1] - transY) / scaleY, id: that.graph.nodes.length});
                      //that.graph.links.push({});
                      //that.graph.links.pop();
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
                  .force("link", d3.forceLink(that.graph.links).distance(100).strength(0.1))
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
                }
              }

              //https://stackoverflow.com/questions/38224875/replacing-d3-transform-in-d3-v4
              function getTransformation(transform) {
                  // Create a dummy g for calculation purposes only. This will never
                  // be appended to the DOM and will be discarded once this function 
                  // returns.
                  var g = document.createElementNS("http://www.w3.org/2000/svg", "g");
                  
                  // Set the transform attribute to the provided string value.
                  g.setAttributeNS(null, "transform", transform);
                  
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
                            .attr("fill", function (d ,i) {
                                return that.color(i);
                            })
                            .on("mousedown", function(event,d) {
                              if (that.edit_mode) {
                                that.selected_node = d;
                              }
                              if (!that.edit_mode) {
                                //event.stopPropagation();
                                //that.drawing_line = true;
                              }
                            })
                            .call(d3.drag()
                                .on("start", function dragstarted(d, event) {
                                  if (!that.edit_mode){
                                    console.log("dragstarted")
                                    if (!event.active) that.simulation.alphaTarget(0.3).restart()
                                    d.subject.fx = d.subject.x
                                    d.subject.fy = d.subject.y
                                  }
                                })
                                .on("drag", function dragged(d) {
                                  if (!that.edit_mode){ 
                                    console.log("dragcontinued")
                                    d.subject.fx = d.x
                                    d.subject.fy = d.y
                                  }
                                })
                                .on("end", function dragended(d, event) {
                                  if (!that.edit_mode){
                                    console.log("dragended")
                                    if (!event.active) that.simulation.alphaTarget(0)
                                    d.subject.fx = null
                                    d.subject.fy = null
                                  }
                                }))
                    }
                    return {}
                },
                links: function () {
                    console.log("calculatedlinks")
                    var that = this
                    if (that.graph) {
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

            },
            updated: function () {
                console.log("updated")
                
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
                
            },
            }

</script>

<style src="./assets/styles.css"></style>