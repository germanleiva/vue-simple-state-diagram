<template>
    <state-diagram :nodes="states" :links="edges"></state-diagram>
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
            states: [
                { id: 'idle', name: 'Idle', x:0, y:0, isSelected: false},
                { id: 'moving', name: 'Moving', x:0, y:0, isSelected: false}
            ],
            edges: [
                { source: 'idle', target: 'moving', name: 'touchstart', isSelected: false},
                { source: 'moving', target: 'moving', name: 'touchmove', isSelected: false},
                { source: 'moving', target: 'idle', name: 'touchend', isSelected: false},
            ]
        }
    },
    mounted() {

    }
}

</script>

<style>
body {
  background: #384d54;
}

</style>
