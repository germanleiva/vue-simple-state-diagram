<template>
    <state-diagram :states="states"></state-diagram>
</template>

<script>

import StateDiagram from './components/StateDiagram'

export default {
    name: 'app',
    components: {
        StateDiagram
    },
    data() {
        return {
            states: [
                { x: 43, y: 67, name: "first", isSelected: false, isHovered: false, transitions: [] },
                { x: 340, y: 150, name: "second", isSelected: false, isHovered: false, transitions: [] },
                { x: 200, y: 250, name: "third", isSelected: false, isHovered: false, transitions: [] },
                { x: 300, y: 320, name: "fourth", isSelected: false, isHovered: false, transitions: [] },
                { x: 50, y: 250, name: "fifth", isSelected: false, isHovered: false, transitions: [] },
                { x: 90, y: 170, name: "last", isSelected: false, isHovered: false, transitions: [] }
            ],
        }
    },
    mounted() {
        this.states[0].transitions.push({ name: 'whooo', target: this.states[1] })
    }
}

</script>

<style scoped>
/* #app {
   font-family: 'Avenir', Helvetica, Arial, sans-serif;
   -webkit-font-smoothing: antialiased;
   -moz-osx-font-smoothing: grayscale;
   text-align: center;
   color: #2c3e50;
   margin-top: 60px;
 }*/

rect.selection {
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

/* disable text selection */
svg *::selection {
    background: transparent;
}

svg *::-moz-selection {
    background: transparent;
}

svg *::-webkit-selection {
    background: transparent;
}
</style>
