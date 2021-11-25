<style>
    #map-container{
        background-color: rgb(180,180,200);
    }
    .district-boundary {
        fill: #fff;
        stroke: rgba(75,0,0,.5);
        stroke-width: 1px;
        transition: all .75s;
    }
    .district-boundary:hover {
        fill: #ffa;
    }
    .map-point {
        stroke: red;
        transition: all .25s;
        fill: rgba(255,0,0,.25);
    }
    .map-point:hover {
        fill: rgba(255,0,0,.5);
    }
    .map-tooltip {
        position: absolute;
        text-align: center;
        padding: 8px;
        margin-top: -20px;
        font: 10px sans-serif;
        background: #000;
        z-index: 30000;
        pointer-events: none;
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
    mounted() {
        window.addEventListener('resize', this.renderMap)
        this.renderMap()
    },
    unmounted() {
        window.removeEventListener('resize', this.renderMap)
    },
    computed: {
        points() {
            return this.observations.map(
                (o) => [o.location.split(',')[1], o.location.split(',')[0], o.id, o.place_guess],
            )
        },
    },
    methods: {
        renderMap() {
            if (!d3.select('#map-container svg').empty()) {
                d3.selectAll('#map-container svg').remove()
            }

            const width = parseInt(d3.select('#map').style('width'), 10)
            const height = parseInt(d3.select('#map').style('height'), 10)
            const projection = d3.geoMercator().scale(1).translate([0, 0])
            const path = d3.geoPath().projection(projection)
            const tooltip = d3.select('body')
                .append('div')
                .attr('class', 'map-tooltip')
                .style('opacity', 0)

            const b = path.bounds(districtsMap)
            const s = 0.95 / Math.max((b[1][0] - b[0][0]) / width, (b[1][1] - b[0][1]) / height)
            const t = [(width - s * (b[1][0] + b[0][0])) / 2, (height - s * (b[1][1] + b[0][1])) / 2]

            projection
                .scale(s)
                .translate(t);

            const mouseover = () => tooltip.transition()
                .duration(200)
                .style('opacity', 0.9)

            const mousemove = (event, d) => tooltip.html(d.properties.district)
                .style('left', (event.pageX - 50) + 'px')
                .style('top', (event.pageY - 10) + 'px')

            const mouseout = () =>  tooltip.transition()
                .duration(500)
                .style('opacity', 0)

            const svg = d3.select('#map-container')
                .append('svg')
                .attr('preserveAspectRatio', 'xMinYMin meet')
                .attr('width', width)
                .attr('height', height)

            svg.append('g')
                .classed('map-boundary', true)
                .selectAll('path')
                .data(districtsMap.features)
                .enter()
                .append('path')
                .attr('d', path)
                .classed('district-boundary', true)
                .attr('title', (d) => d.properties.district)
                .on('mouseover', mouseover)
                .on('mousemove', (event, d) => mousemove(event, d))
                .on('mouseout', mouseout)

            svg.append('g')
                .classed('map-points', true)
                .selectAll('circle')
                .data(this.points)
                .enter()
                .append('circle')
                .classed('map-point', true)
                .attr('cx', (d) => projection(d)[0])
                .attr('cy', (d) => projection(d)[1])
                .attr('r', '5px')
                .attr('title', (d) => d)
        },
    },
}
</script>
