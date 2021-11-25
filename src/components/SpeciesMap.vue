<style>
    .map-boundary > path {
        fill: white;
        stroke: rgba(75,0,0,.5);
        stroke-width: 1px;
    }
    .poly_text{
        fill: #545;
        font-size: 0.5vw;
        transition: fill,text-shadow .125s;
    }
    .poly_text:hover{
        fill: #00c;
        text-shadow: 0px 0px 5px #fff;
        cursor: pointer;
        font-weight: 1000;
    }
    @media screen and (max-width: 800px) {
        .poly_text{
            font-size: 3.5vw;
        }
        #map-container svg{
            height: 100%;
            width: 100%;
        }
    }
</style>

<template>
    <div id="map-container"></div>
</template>

<script>
import * as d3 from 'd3'
import districtsMap from '../assets/book_data/districts_map.json'

export default {
    name: 'SpeciesMap',
    props: ['observations'],
    data() {
        return {
            states: null,
            path: null,
            svg: {},
            projection: {},
            colors: {},
            legend: {},

            state_data: {},
            selected: 'All',
            state_max: 0,
            height: 100,
            width: 100,
            map_first_render: true,
        }
    },
    mounted() {
        this.init()
        window.addEventListener('resize', this.init)
    },
    unmounted() {
        window.removeEventListener('resize', this.init)
    },
    computed: {
        points() {
            const op = []
            this.observations.forEach((o) => {
                const l = o.location.split(',')
                op.push([l[1], l[0], o.id, o.place_guess])
            })
            return op
        },
    },
    watch: {
    },
    methods: {
        init() {
            this.width = parseInt(d3.select('#map').style('width'), 10)
            this.height = parseInt(d3.select('#map').style('height'), 10)

            this.renderMap()
        },
        renderMap() {
            if (!d3.select('#map-container svg').empty()) {
                d3.selectAll('#map-container svg').remove()
            }

            this.svg = d3.select('#map-container')
                .append('svg')
                .attr('preserveAspectRatio', 'xMinYMin meet')
                .attr('width', this.width)
                .attr('height', this.height)
                .style('background-color', 'rgb(190, 229, 235)')
                .classed('svg-content d-flex m-auto', true)

            this.projection = d3.geoMercator().scale(1).translate([0, 0])
            this.path = d3.geoPath().projection(this.projection)

            const b = this.path.bounds(districtsMap)
            const s = 0.95 / Math.max((b[1][0] - b[0][0]) / this.width, (b[1][1] - b[0][1]) / this.height)
            const t = [(this.width - s * (b[1][0] + b[0][0])) / 2, (this.height - s * (b[1][1] + b[0][1])) / 2]
            // Update the projection to use computed scale & translate.
            this.projection
                .scale(s)
                .translate(t);

            const base = this.svg.append('g')
                .classed('map-boundary', true)
                .selectAll('path').append('g')

            this.states = base.append('g').classed('states', true)

            districtsMap.features.forEach((district) => {
                const name = district.properties.district

                this.states.append('g')
                    .data([district])
                    .enter().append('path')
                    .attr('d', this.path)
                    .attr('title', name)
            })
            this.svg.append('g')
                .classed('map-points', true)
                .selectAll('circle')
                .data(this.points)
                .enter()
                .append('circle')
                .attr('cx', (d) => this.projection(d)[0])
                .attr('cy', (d) => this.projection(d)[1])
                .attr('r', '5px')
                .attr('title', (d) => d)
                .attr('stroke', 'red')
                .attr('fill', 'rgba(255,0,0,.25)')

            // this.svg.call(this.zoom)
        },
        stateID(s) {
            return s.replaceAll(' ', '_').replaceAll('&', '')
        },
    },
};
</script>
