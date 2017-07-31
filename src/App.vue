<template>
    <state-diagram></state-diagram>
</template>

<script>

import Vue from 'vue'
import VueD3 from 'vue-d3'
Vue.use(VueD3)
import StateDiagram from './components/StateDiagram'

export default {
    name: 'app',
    components: {
        StateDiagram
    },
    data() {
        return {
        }
    },
    mounted() {

    }
}

</script>

<style>
body {
  background: #384d54;
  color: white;
}
text {
    fill: white;
    font-family: 'Open Sans';
}
circle {
  stroke: white;
  stroke-width: 1.5px;
}
path.link, path.textpath {
    fill: none;
    stroke: #cccccc;
    stroke-width: 0.5px;
}
path.invis {
    fill: none;
    stroke-width: 0;
}
.textClass {
    font-size: 25px;
}
</style>
