<template>
    <!-- <svg id="svg" width="960px" height="500px" @mousedown="canvasMouseDown" @mousemove="canvasMouseMove" @mouseup="canvasMouseUp" @dblclick="canvasDoubleClick"> -->
    <svg id="svg" width="1024px" height="900px">
        <!--define arrow markers for graph links-->
        <defs>
             <marker id="arrowhead" :viewBox="`0 -${arrowSize/2} ${arrowSize} ${arrowSize}`" :refX="nodeRadius * 3.5" :refY="nodeRadius/5" orient="auto" :markerWidth="arrowSize" :markerHeight="arrowSize" markerUnits="strokeWidth" overflow="visible">
                <path :d="`M0,-${arrowSize/2}L${arrowSize},0L0,${arrowSize/2}z`" fill="#ccc">
                </path>
            </marker>
             <marker id="selfarrowhead" :viewBox="`0 -${arrowSize/2} ${arrowSize} ${arrowSize}`" :refX="nodeRadius * 3.5" :refY="nodeRadius/5 - 20" orient="-105" :markerWidth="arrowSize" :markerHeight="arrowSize" markerUnits="strokeWidth" overflow="visible">
                <path :d="`M0,-${arrowSize/2}L${arrowSize},0L0,${arrowSize/2}z`" fill="#ccc">
                </path>
            </marker>
        </defs>
        <!--line displayed when dragging new nodes-->
        <!-- <path class="dragline" v-if="startState != undefined" :d="'M' + startState.x + ',' + startState.y + 'L' + dragLineEnd.x + ',' + dragLineEnd.y"></path> -->
        <path v-for="eachLink in links" class="link" :marker-end="arrowHeadMaker(eachLink)" :d="arcPath(true, eachLink)"></path>
        <path v-for="eachLink in links" :id="textPathHref(eachLink)" class="invis" :d="arcPath2(eachLink)"></path>
        <g v-for="eachLink in links">
            <text class="pathLabel">
                <textPath startOffset="50%" text-anchor="middle" :href="'#'+textPathHref(eachLink)" syle="fill:#cccccc;font-size:20">{{eachLink.name}}</textPath>
            </text>
        </g>
        <g v-for="eachState in nodes" :key="eachState.id" class="node" :transform="'translate(' + eachState.x + ',' + eachState.y + ')'">
            <!-- <circle r="50" class="outer" @mousedown="mouseDownOnOuterCircle(eachState)"></circle> -->

            <!-- <circle r="40" class="inner" @mousedown="mouseDownOnInnerCircle(eachState,$event)" @mouseup="mouseUpOnInnerCircle(eachState,$event)" @mouseover="eachState.isHovered = true" @mouseout="eachState.isHovered = false"></circle> -->
            <circle :id="'Node;'+eachState.id" :r="nodeRadius" class="nodeStrokeClass" fill="#0db7ed"></circle>
            <text class="textClass" x="20" y=".31em">{{eachState.name}}</text>

            <!-- <text text-anchor="middle" y="4">{{eachState.name}}</text> -->
            <!-- <title>{{eachState.name}}</title> -->
        </g>
        <!-- <rect v-if="selectionRectangle != undefined" class="selection" :rx="selectionRectangle.rx"  :ry="selectionRectangle.ry"  :x="selectionRectangle.x"  :y="selectionRectangle.y"  :width="selectionRectangle.width" :height="selectionRectangle.height"></rect> -->
    </svg>
</template>

<script>

var linkDistance = 300;

export default {
    name: 'state-machine',
    props: {
      // states: {
      //   type: Array,
      //   default: []
      // }
    },
    components: {
    },
    data() {
        return {
            nodes: [
                { id: 'idle', name: 'Idle', x:20, y:30},
                { id: 'moving', name: 'Moving', x:30, y:50}
            ],
            links: [
                { source: 'idle', target: 'moving', name: 'touchstart'},
                { source: 'moving', target: 'moving', name: 'touchmove'},
                { source: 'moving', target: 'idle', name: 'touchend'},
            ]
        }
    },
    computed: {
        nodeRadius() {
            return 20;
        },
        arrowSize() {
            return 30;
        },
        transitions: function () {
            return this.states.reduce(function (initial, state) {
                return initial.concat(
                    state.transitions.map(function (transition) {
                        return { source: state, target: transition.target };
                    })
                );
            }, []);
        }
    },
    mounted() {
        let vm = this;

        let simulation = this.$d3.forceSimulation()
            .force("x",this.$d3.forceX(window.innerWidth/2))
            .force("y",this.$d3.forceY(window.innerHeight/2))
            .force("collide",this.$d3.forceCollide(this.nodeRadius + 1))
            .force("charge",this.$d3.forceManyBody().strength(-20))
            .force("link",this.$d3.forceLink()
                .id(function(aNode) {
                    return aNode.id
                })
                .distance(linkDistance)
            )
            // .on("tick",function(e) {
            //     vm.$forceUpdate
            // })

        simulation.nodes(this.nodes)
        simulation.force("link").links(this.links)

        let container = document.querySelector("svg");

        function dragsubject() {
            let result = simulation.find(this.$d3.event.x, this.$d3.event.y,this.nodeRadius);
            // debugger;
            return result;
        }
        function dragstarted() {
          if (!this.$d3.event.active) simulation.alphaTarget(0.3).restart();
          this.$d3.event.subject.fx = this.$d3.event.subject.x;
          this.$d3.event.subject.fy = this.$d3.event.subject.y;
        }

        function dragged() {
          this.$d3.event.subject.fx = this.$d3.event.x;
          this.$d3.event.subject.fy = this.$d3.event.y;
        }

        function dragended() {
          if (!this.$d3.event.active) simulation.alphaTarget(0);
          this.$d3.event.subject.fx = null;
          this.$d3.event.subject.fy = null;
        }

        this.$d3.select(container).call(this.$d3.drag()
          .container(container)
          .subject(dragsubject.bind(this))
          .on("start", dragstarted.bind(this))
          .on("drag", dragged.bind(this))
          .on("end", dragended.bind(this)));

        //keep nodes on top
        // for (var each of document.body.getElementsByClassName("nodeStrokeClass")) {
        //     var gNode = each.parentNode;
        //     gNode.parentNode.appendChild(gNode);
        // };
    },
    methods: {
        arrowHeadMaker(eachLink) {
            return eachLink.source == eachLink.target ? 'url(#selfarrowhead)' : 'url(#arrowhead)'
        },
      siblingLinks(source, target) {
          var siblings = [];
          for(var i = 0; i < this.links.length; ++i){
              if( (this.links[i].source.id == source.id && this.links[i].target.id == target.id) || (this.links[i].source.id == target.id && this.links[i].target.id == source.id) )
                  siblings.push(this.links[i].name);
          };
          return siblings;
      },
      arcPath2(d) {
        // let sourceNode = this.nodes.find(aNode => aNode.id == d.source);
        // let targetNode = this.nodes.find(aNode => aNode.id == d.target)
// debugger;
        return this.arcPath(d.source.x < d.target.x,d)
      },
        arcPath(leftHand, d) {
            // debugger;

            // let sourceNode = this.nodes.find(aNode => aNode.id == d.source)
            // let targetNode = this.nodes.find(aNode => aNode.id == d.target)

            if (!d.source || !d.target) {
                debugger
            }

            var x1 = leftHand ? d.source.x : d.target.x,
                y1 = leftHand ? d.source.y : d.target.y,
                x2 = leftHand ? d.target.x : d.source.x,
                y2 = leftHand ? d.target.y : d.source.y,
                dx = x2 - x1,
                dy = y2 - y1,
                dr = Math.sqrt(dx * dx + dy * dy),
                drx = dr,
                dry = dr,
                sweep = leftHand ? 0 : 1,
                siblingCount = this.siblingLinks(d.source, d.target).length,
                xRotation = 0,
                largeArc = 0;

                if (dr === 0) {
                    sweep = 0;
                    xRotation = -130;
                    largeArc = 1;
                    drx = 50;
                    dry = 40;
                    x2 = x2 + 1;
                    y2 = y2 + 1;
                }

                if (siblingCount > 1) {
                    var siblings = this.siblingLinks(d.source, d.target);
                    // console.log(siblings);
                    var arcScale = this.$d3.scaleOrdinal()
                                        .domain(siblings)
                                        .range([1, siblingCount]);
                    drx = drx/(1 + (1/siblingCount) * (arcScale(d.name) - 1));
                    dry = dry/(1 + (1/siblingCount) * (arcScale(d.name) - 1));
                }

            return "M" + x1 + "," + y1 + "A" + drx + ", " + dry + " " + xRotation + ", " + largeArc + ", " + sweep + " " + x2 + "," + y2;
        },
        textPathHref: function(aLink) {
            return "invis_" + aLink.source.id + "-" + aLink.name + "-" + aLink.target.id;
        }
        // // http://www.dashingd3js.com/svg-paths-and-d3js
        // computeTransitionPath: /*d3.svg.diagonal.radial()*/function (d) {
        //     var deltaX = d.target.x - d.source.x,
        //         deltaY = d.target.y - d.source.y,
        //         dist = Math.sqrt(deltaX * deltaX + deltaY * deltaY),
        //         normX = deltaX / dist,
        //         normY = deltaY / dist,
        //         sourcePadding = radius + 2,//d.left ? 17 : 12,
        //         targetPadding = radius + 6,//d.right ? 17 : 12,
        //         sourceX = d.source.x + (sourcePadding * normX),
        //         sourceY = d.source.y + (sourcePadding * normY),
        //         targetX = d.target.x - (targetPadding * normX),
        //         targetY = d.target.y - (targetPadding * normY);
        //     return 'M' + sourceX + ',' + sourceY + 'L' + targetX + ',' + targetY;
        // },
        // mouseDownOnOuterCircle(aState) {
        //     if (aState.isSelected) {
        //       this.startState = aState;
        //       this.dragLineEnd = {x:aState.x,y:aState.y}
        //       this.endState = undefined;
        //     }
        // },
        // mouseDownOnInnerCircle(aState,event) {
        //     aState.isSelected = true

        //     if (this.startState) {
        //         return;
        //     }

        //     this.previousPosition.x = event.pageX
        //     this.previousPosition.y = event.pageY

        //     this.draggedState = aState

        //     //This is to move the state at the end of the list, thus putting on top of all the other states
        //     //Remove element
        //     var index = this.states.indexOf(aState);
        //     if (index > -1) {
        //         this.states.splice(index, 1);
        //     }
        //     //Add element at the end
        //     this.states.push(aState)
        // },
        // mouseUpOnInnerCircle(aState,event) {
        //     this.previousPosition.x = 0
        //     this.previousPosition.y = 0

        //     if( this.startState && this.endState) {
        //         this.startState.transitions.push( { name : "transition name 1", target : this.endState});
        //     }

        //     this.startState = undefined
        //     this.dragLineEnd = undefined
        //     this.draggedState = undefined
        // },
        // canvasMouseDown(event) {
        //     if( !event.shiftKey) {
        //         for (let eachState of this.states) {
        //             if (eachState != this.draggedState)
        //                 eachState.isSelected = false
        //         }
        //         return
        //     }

        //     this.selectionRectangle = {rx:6,ry:6,x:event.pageX,y:event.pageY,width:0,height:0}
        // },
        // canvasMouseMove(event) {
        //     if( this.selectionRectangle) {
        //         let move = {x: event.pageX - this.selectionRectangle.x , y: event.pageY - this.selectionRectangle.y }

        //         if( move.x < 1 || (move.x*2<this.selectionRectangle.width)) {
        //             this.selectionRectangle.x = event.pageX;
        //             this.selectionRectangle.width -= move.x;
        //         } else {
        //             this.selectionRectangle.width = move.x;
        //         }

        //         if( move.y < 1 || (move.y*2<this.selectionRectangle.height)) {
        //             this.selectionRectangle.y = event.pageY;
        //             this.selectionRectangle.height -= move.y;
        //         } else {
        //             this.selectionRectangle.height = move.y;
        //         }

        //         // deselect all temporary selected state objects
        //         for (let eachState of this.states) {
        //             // inner circle inside selection frame
        //             eachState.isSelected = false

        //             //xInside && yInside
        //             if (eachState.x-radius>=this.selectionRectangle.x && eachState.x+radius<=this.selectionRectangle.x+this.selectionRectangle.width &&
        //                 eachState.y-radius>=this.selectionRectangle.y && eachState.y+radius<=this.selectionRectangle.y+this.selectionRectangle.height) {
        //                 eachState.isSelected = true
        //             }
        //         }
        //     } else if (this.startState) {
        //         //update drag line
        //         this.dragLineEnd.x = event.pageX
        //         this.dragLineEnd.y = event.pageY

        //         for (let eachState of this.states) {
        //             if (eachState.isHovered) {
        //                 this.endState = eachState;
        //                 continue;
        //             }
        //         }

        //     } else if (this.draggedState) {
        //         this.draggedState.x += event.pageX - this.previousPosition.x;
        //         this.draggedState.y += event.pageY - this.previousPosition.y;

        //         this.previousPosition.x =  event.pageX
        //         this.previousPosition.y =  event.pageY
        //     }
        // },
        // canvasMouseUp(event) {
        //     this.selectionRectangle = undefined
        // },
        // canvasDoubleClick(event) {
        //     this.states.push( { x : event.pageX, y : event.pageY, name : "tst", isSelected:false, isHovered: false, transitions : [] });
        // }
    }
}

</script>

<style >
/* #app {
   font-family: 'Avenir', Helvetica, Arial, sans-serif;
   -webkit-font-smoothing: antialiased;
   -moz-osx-font-smoothing: grayscale;
   text-align: center;
   color: #2c3e50;
   margin-top: 60px;
 }*/

/*rect.selection {
    stroke: gray;
    stroke-dasharray: 4px;
    stroke-opacity: 0.5;
    fill: transparent;
}

g.state circle {
    stroke: gray;
    cursor: pointer;
}

g.state circle.inner {
    fill: white;
}

g.state.hover circle.inner {
    fill: aliceblue;
}

g.state circle.outer {
    stroke-width: 0px;
    stroke-dasharray: 4px;
    stroke-opacity: 0.5;
    fill: transparent;
}

g.state.selected circle.outer {
    stroke-width: 1px;
}

g.state text {
    font: 12px sans-serif;
    font-weight: bold;
    pointer-events: none;
}

path.transition,
path.dragline {
    fill: none;
    stroke: #000;
    stroke-width: 1px;
    cursor: default;
    marker-end: url(#end-arrow);
}

path.dragline {
    pointer-events: none;
}

// disable text selection
svg *::selection {
    background: transparent;
}

svg *::-moz-selection {
    background: transparent;
}

svg *::-webkit-selection {
    background: transparent;
}*/
</style>
