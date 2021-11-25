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
</style>

<template>
    <div id="map-container"></div>
</template>

<script>
import * as d3 from 'd3'
import { geoToH3, h3ToGeoBoundary } from 'h3-js';

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
    watch: {
        observations() {
            this.renderMap()
        },
    },
    computed: {
        points() {
            return this.observations.map(
                (o) => [o.location.split(',')[1], o.location.split(',')[0], o.id, o.place_guess],
            )
        },
        color() {
            const max = d3.max(this.polygons.map((p) => p.ids.length))
            return d3.scaleLinear().domain([0, max]).range(['#206020', '#22BB22'])
        },
        polygons() {
            const op = {}
            this.points.forEach((point) => {
                const h3Address = geoToH3(point[1], point[0], 6)
                if (op[h3Address] !== undefined) {
                    op[h3Address].ids.push(point[2])
                } else {
                    const h3Geo = this.hexFeatures(h3ToGeoBoundary(h3Address, true))
                    op[h3Address] = {
                        ids: [point[2]],
                        polygon: h3Geo.features[0],
                    }
                }
            })
            return Object.values(op)
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

            let tooltip = null

            if (!d3.select('.tooltip').empty()) {
                tooltip = d3.select('.tooltip')
            } else {
                tooltip = d3.select('body')
                    .append('div')
                    .attr('class', 'tooltip')
                    .style('opacity', 0)
            }

            const b = path.bounds(districtsMap)
            const s = 0.95 / Math.max((b[1][0] - b[0][0]) / width, (b[1][1] - b[0][1]) / height)
            const t = [(width - s * (b[1][0] + b[0][0])) / 2, (height - s * (b[1][1] + b[0][1])) / 2]

            projection
                .scale(s)
                .translate(t);

            const mouseover = () => tooltip.transition()
                .duration(200)
                .style('opacity', 0.9)

            const mousemoveDistrict = (event, d) => tooltip.html(d.properties.district)
                .style('left', `${(event.pageX - 50)}px`)
                .style('top', `${(event.pageY - 10)}px`)

            const mousemoveHexagon = (event, d) => tooltip.html(`${d.ids.length} observation${(d.ids.length > 1) ? 's' : ''}`)
                .style('left', `${(event.pageX - 50)}px`)
                .style('top', `${(event.pageY - 10)}px`)

            const mouseout = () => tooltip.transition()
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
                .on('mousemove', (event, d) => mousemoveDistrict(event, d))
                .on('mouseout', mouseout)

            svg.append('g')
                .classed('map-points', true)
                .selectAll('path')
                .data(this.polygons)
                .enter()
                .append('path')
                .classed('polygon', true)
                .attr('d', (d) => path(d.polygon))
                .attr('fill', (d) => this.color(d.ids.length))
                .on('mouseover', mouseover)
                .on('mousemove', (event, d) => mousemoveHexagon(event, d))
                .on('mouseout', mouseout)
        },
        hexFeatures(array) {
            return {
                type: 'FeatureCollection',
                features: [{
                    type: 'Feature',
                    geometry: {
                        type: 'Polygon',
                        coordinates: [array.reverse()],
                    },
                }],
            }
        },
    },
}
</script>
