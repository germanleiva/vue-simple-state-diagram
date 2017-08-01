<template>
    <!-- <svg id="svg" width="960px" height="500px" @mousedown="canvasMouseDown" @mousemove="canvasMouseMove" @mouseup="canvasMouseUp" @dblclick="canvasDoubleClick"> -->
    <svg id="svg" width="1024px" height="900px" @dblclick.prevent="canvasDoubleClick">
        <!--define arrow markers for graph links-->
        <defs>
             <marker id="arrowhead" :viewBox="`0 -${arrowSize/2} ${arrowSize} ${arrowSize}`" :refX="nodeRadius * 2.4" :refY="nodeRadius/20" orient="auto" :markerWidth="arrowSize" :markerHeight="arrowSize" markerUnits="userSpaceOnUse" overflow="visible">
                <path :d="`M0,-${arrowSize/2}L${arrowSize},0L0,${arrowSize/2}z`" fill="#ccc">
                </path>
            </marker>
             <marker id="arrowhead-selected" :viewBox="`0 -${arrowSize/2} ${arrowSize} ${arrowSize}`" :refX="nodeRadius * 2.4" :refY="nodeRadius/20" orient="auto" :markerWidth="arrowSize" :markerHeight="arrowSize" markerUnits="userSpaceOnUse" overflow="visible">
                <path :d="`M0,-${arrowSize/2}L${arrowSize},0L0,${arrowSize/2}z`" fill="red">
                </path>
            </marker>
             <marker id="selfarrowhead" :viewBox="`0 -${arrowSize/2} ${arrowSize} ${arrowSize}`" :refX="nodeRadius * 2.4" :refY="nodeRadius/5 - 9" orient="-110" :markerWidth="arrowSize" :markerHeight="arrowSize" markerUnits="userSpaceOnUse" overflow="visible">
                <path :d="`M0,-${arrowSize/2}L${arrowSize},0L0,${arrowSize/2}z`" fill="#ccc">
                </path>
            </marker>
             <marker id="selfarrowhead-selected" :viewBox="`0 -${arrowSize/2} ${arrowSize} ${arrowSize}`" :refX="nodeRadius * 2.4" :refY="nodeRadius/5 - 9" orient="-110" :markerWidth="arrowSize" :markerHeight="arrowSize" markerUnits="userSpaceOnUse" overflow="visible">
                <path :d="`M0,-${arrowSize/2}L${arrowSize},0L0,${arrowSize/2}z`" fill="red">
                </path>
            </marker>
        </defs>
        <!--line displayed when dragging new nodes-->
        <path v-for="eachLink in links" class="link" :class="{selected:eachLink.isSelected}" :marker-end="arrowHeadMaker(eachLink)" :d="arcPath(true, eachLink)" @click.prevent="toggleLink(eachLink)"></path>
        <path v-for="eachLink in links" :id="invisiblePath(eachLink)" class="invis" :d="arcPath(eachLink.source.x < eachLink.target.x,eachLink)"></path>
        <g v-for="eachLink in links">
            <text class="linkLabel" :class="{selected:eachLink.isSelected}" dy="-5" @click.prevent="toggleLink(eachLink)">
                <textPath startOffset="50%" text-anchor="middle" :href="'#'+invisiblePath(eachLink)" syle="fill:#cccccc;font-size:50px">{{eachLink.name}}</textPath>
            </text>
        </g>
        <g v-for="eachNode in nodes" :key="eachNode.id" :transform="'translate(' + eachNode.x + ',' + eachNode.y + ')'">
            <!-- <circle r="50" class="outer" @mousedown="mouseDownOnOuterCircle(eachNode)"></circle> -->

            <!-- <circle r="40" class="inner" @mousedown="mouseDownOnInnerCircle(eachNode,$event)" @mouseup="mouseUpOnInnerCircle(eachNode,$event)" @mouseover="eachNode.isHovered = true" @mouseout="eachNode.isHovered = false"></circle> -->
            <circle :id="'Node;'+eachNode.id" :r="nodeRadius" class="node" :class="{selected:eachNode.isSelected}" @click.prevent="toggleNode(eachNode)"></circle>
            <text class="nodeTextClass" x="20" y=".31em" @click.prevent="toggleNode(eachNode)">{{eachNode.name}}</text>

            <!-- <text text-anchor="middle" y="4">{{eachNode.name}}</text> -->
            <!-- <title>{{eachNode.name}}</title> -->
        </g>
        <path class="dragline" v-if="startState != undefined" :d="dragLinePath"></path>

        <!-- <rect v-if="selectionRectangle != undefined" class="selection" :rx="selectionRectangle.rx"  :ry="selectionRectangle.ry"  :x="selectionRectangle.x"  :y="selectionRectangle.y"  :width="selectionRectangle.width" :height="selectionRectangle.height"></rect> -->
    </svg>
</template>

<script>

var linkDistance = 300;

export default {
    name: 'state-machine',
    props: {
      nodes: {
        type: Array,
        default: []
      },
      links: {
        type: Array,
        default: []
      }
    },
    components: {
    },
    data() {
        return {
            simulation: undefined,
            startState: undefined,
            dragLineEnd: undefined,
            nodeRadius: 20,
            arrowSize: 30
        }
    },
    computed: {
        dragLinePath() {
            return 'M' + this.startState.x + ',' + this.startState.y + 'L' + this.dragLineEnd.x + ',' + this.dragLineEnd.y
        }
    },
    mounted() {
        let vm = this;

        this.simulation = this.$d3.forceSimulation(this.nodes)
            .force("x",this.$d3.forceX(window.innerWidth/2))
            .force("y",this.$d3.forceY(window.innerHeight/2))
            .force("collide",this.$d3.forceCollide(this.nodeRadius + 1))
            .force("charge",this.$d3.forceManyBody().strength(-20))
            .force("link",this.$d3.forceLink(this.links)
                .id(function(aNode) {
                    return aNode.id
                })
                .distance(linkDistance)
            )
            // .on("tick",function(e) {
            //     vm.$forceUpdate
            // })

        // simulation.nodes(this.nodes)
        // simulation.force("link").links(this.links)

        let container = document.querySelector("svg");

        function dragsubject() {
            let e = this.$d3.event
            //nodeRadius constraint drag to the nodes and not the rest of the container (e.g edges)
            let result = this.simulation.find(e.x, e.y,this.nodeRadius);
            return result;
        }
        function dragstarted() {
            let e = this.$d3.event
            if (e.sourceEvent.shiftKey) {
                this.startState = e.subject
                this.dragLineEnd = {x:e.sourceEvent.pageX,y:e.sourceEvent.pageY}
                return
            }
            if (!e.active) this.simulation.alphaTarget(0.3).restart();
            e.subject.fx = e.subject.x;
            e.subject.fy = e.subject.y;
        }

        function dragged() {
            let e = this.$d3.event
            if (this.startState) {
                this.dragLineEnd.x = event.pageX
                this.dragLineEnd.y = event.pageY
                return
            }
            e.subject.fx = e.x;
            e.subject.fy = e.y;
        }

        function dragended() {
            let e = this.$d3.event

            if (this.startState) {
                let endState = this.simulation.find(e.x, e.y,this.nodeRadius);

                if (endState) {
                    let newLink = {source: this.startState.id, target: endState.id, name: 'unnamed '+this.links.length, isSelected:false}
                    this.links.push(newLink)

                    //To force an update
                    this.simulation.force("link").links(this.links)
                }
                this.dragLineEnd = undefined
                this.startState = undefined
                return;
            }
            if (!e.active) this.simulation.alphaTarget(0);
            e.subject.fy = null;
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
        toggleNode(aNode) {
            for (let eachLink of this.links) {
                eachLink.isSelected = false
            }
            for (let eachNode of this.nodes) {
                eachNode.isSelected = eachNode == aNode
            }
        },
        toggleLink(aLink) {
            for (let eachNode of this.nodes) {
                eachNode.isSelected = false
            }
            for (let eachLink of this.links) {
                eachLink.isSelected = eachLink == aLink
            }
        },
        arrowHeadMaker(eachLink) {
            let name = eachLink.source == eachLink.target ? '#selfarrowhead' : '#arrowhead'
            let suffix = ""
            if (eachLink.isSelected) {
                suffix = "-selected"
            }
            return `url(${name}${suffix})`
        },
        arcPath(leftHand, d) {
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
                siblings = this.siblingLinks(d.source, d.target),
                siblingCount = siblings.length,
                xRotation = 0,
                largeArc = 0;

                if (dr === 0) {
                    sweep = 0;
                    xRotation = -130;
                    largeArc = 1;
                    drx = 60;
                    dry = 50;
                    x2 = x2 + 1;
                    y2 = y2 + 1;
                }

                if (siblingCount > 1) {
                    // console.log(siblings);
                    var arcScale = this.$d3.scalePoint()
                                        .domain(siblings)
                                        .range([1, siblingCount]);
                    drx = drx/(1 + (1/siblingCount) * (arcScale(d.name) - 1));
                    dry = dry/(1 + (1/siblingCount) * (arcScale(d.name) - 1));
                }

            return "M" + x1 + "," + y1 + "A" + drx + ", " + dry + " " + xRotation + ", " + largeArc + ", " + sweep + " " + x2 + "," + y2;
        },
        siblingLinks(source, target) {
          var siblings = [];
          for(var i = 0; i < this.links.length; ++i){
              if( (this.links[i].source.id == source.id && this.links[i].target.id == target.id) || (this.links[i].source.id == target.id && this.links[i].target.id == source.id) )
                  siblings.push(this.links[i].name);
          };
          return siblings;
        },
        invisiblePath(aLink) {
            return "invis_" + aLink.source.id + "-" + aLink.name + "-" + aLink.target.id;
        },
        canvasDoubleClick(event) {
            let newState = {id: `${this.nodes.length}`, name: 'new state', x:event.pageX, y:event.pageY};
            this.nodes.push(newState);

            //To force an update
            this.simulation.nodes(this.nodes)
        }
    }
}

</script>

<style scoped>

text {
    fill: white;
    font-family: 'Open Sans';
    user-select: none;
}

.node {
    stroke: white;
    stroke-width: 1.5px;
    fill:#0db7ed;
}

.node.selected {
    fill:red;
}

path.link, path.textpath {
    fill: none;
    stroke: #cccccc;
    stroke-width: 1px;
}

path.link.selected{
    stroke-width: 4px;
    stroke: red
}

path.invis {
    fill: none;
    stroke-width: 0;
}
.nodeTextClass {
    font-size: 30px;
}
.linkLabel {
    font-size: 20px;
    user-select: none;
}
.linkLabel.selected {
    font-size: 30px;
    fill: red
}
.dragline {
    fill: none;
    stroke: #000;
    stroke-width: 1px;
    /*cursor: default;*/
    /*marker-end: url(#arrowhead);*/
}

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
