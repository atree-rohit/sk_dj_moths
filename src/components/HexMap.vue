<style scoped>
    #mapid{height: 40em;width: 100%;z-index: 1}
</style>

<template>
    <div>
        <l-map id="mapid" :zoom="zoom" :center="center" class="svg-container bg-light col">

            <l-control-layers position="topright"  ></l-control-layers>
            <l-tile-layer
                v-for="tileProvider in tileProviders"
                :key="tileProvider.name"
                :name="tileProvider.name"
                :visible="tileProvider.visible"
                :url="tileProvider.url"
                :attribution="tileProvider.attribution"
                layer-type="base"/>
            <l-polygon v-for="(hex, id) in h3Hex" 
                :key="id"
                :lat-lngs="h3_coordinates(id)"
                :fillColor="color(hex.species)"
                color="#000000"
                :fillOpacity="0.5"
                @click="openPopUp(getLatLon(hex),hex)"
            >
            </l-polygon>

            <l-layer-group ref="features" >
                <l-popup >
                    <div class="card">
                        {{ caller.species }} Observations                        
                    </div>
                    
                </l-popup>
            </l-layer-group>
        </l-map>

    </div>
</template>

<script>
import {LMap, LTileLayer,LLayerGroup, LPopup, LControlLayers, LPolygon} from 'vue2-leaflet';
import {scaleLinear} from 'd3';
import {geoToH3, h3ToGeoBoundary, h3ToGeo} from 'h3-js';

export default {
    name: 'cs-hex-map',
    components: {
        LMap,
        LTileLayer,
        LLayerGroup,
        LControlLayers,
        LPopup,
        LPolygon
    },
    props: ["map_data", "hexSize", "observation_link", "links"],
    data () {
        return {
            svgWidth : window.innerWidth / 2,
            svgHeight : window.innerHeight - 30,
            maxZoomValue : 15,
            minZoomValue : 5,
            url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
            zoom: 8,
            center: [27.5, 88.5],
            obs_points: [],
            tileProviders: [
                {
                name: 'OpenStreetMap',
                visible: true,
                attribution:
                    '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
                url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
                },
                {
                name: 'OpenTopoMap',
                visible: false,
                url: 'https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png',
                attribution:
                    'Map data: &copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>, <a href="http://viewfinderpanoramas.org">SRTM</a> | Map style: &copy; <a href="https://opentopomap.org">OpenTopoMap</a> (<a href="https://creativecommons.org/licenses/by-sa/3.0/">CC-BY-SA</a>)',
                },
            ],
            h3Hex : [],
            filterSpecies : "ALL",
            svg : 0,
            path : 0,
            h3Zoom : 5,
            species_list : [],
            source: "none",
            row: [],
            speciesMaxValue:0,
            caller:[],
        }
    },
    created(){
        this.map_data.map((object, index) => {
            if(this.links != "")
            {
                this.links.forEach((link,i)=>{
                    if(object.photo_url != undefined && object.photo_url.substring(0,1) == i)
                    {
                        this.map_data[index].mergedUrl = link + object.photo_url.substring(1, object.photo_url.length)
                    }
                    this.map_data[index].url = this.observation_link + object.id;
                })
            }
            else
            {
                this.map_data[index].mergedUrl = object.photo_url;
                this.map_data[index].url = this.observation_link + object.id;
            }
        })
        
        this.renderHex();
    },
    watch: {
        hexSize:{
            immediate:false,
            handler(newVal, oldVal){
                this.h3Zoom = newVal;
                this.renderHex();
            }
        }
    },
    computed: {
        color:function(){
            return scaleLinear().domain([1, this.speciesMaxValue/2, this.speciesMaxValue]).range(["red",  "darkgreen"])
        }
    },
    methods: {
        h3_coordinates(index){
            var op = [];
            this.h3Hex[index].coordinates.features[0].geometry.coordinates[0].forEach(c => {
                op.push([c[1], c[0]]);
            });

            return op;
        },
        renderHex()
        {
            this.h3Hex = [];
            this.speciesMaxValue = 0;

            this.map_data.forEach((p, index) => {
                const h3Address = geoToH3(p.lat, p.long, this.h3Zoom)
                var matchFlag = false;
                var matchID = null;   
                var id = null;
                this.h3Hex.forEach((h,i) => {
                    if (h.hexID == h3Address) {
                        matchFlag = true
                        matchID = i
                        id = h.hexID
                    }
                })
                // source = "none";
                if(p.taxa_name != null || p.taxa_name != "")
                    this.row = this.parseData(p, this.source)

                // console.log(index, matchFlag, matchID, h3Address+" = "+id, p.taxa_name, this.parseData(p, this.source))
                if(this.row.length > 0){
                    if (matchFlag == false) {
                        var h3Boundary = h3ToGeoBoundary(h3Address, true)
                        var h3Geo = this.hexFeatures(h3Boundary)
                        this.h3Hex.push({ "hexID": h3Address, "coordinates": h3Geo, "rows": this.row })
                    }
                    else {                        
                        this.h3Hex[matchID].rows = this.h3Hex[matchID].rows.concat(this.row)
                    }
                }
            })

            var uniqueSpecies = null;
            var total_individuals = 0;
            var unique_observers = null;
            var observers = "";
            
            this.h3Hex.forEach((h,i) => {
                uniqueSpecies = h.rows.map(function(value,index) {
                    return value["scientific_name"];
                }).filter(function(v, i, self){
                    return i == self.indexOf(v);
                });                
                // h.rows.forEach(r => {
                //     total_individuals += parseInt(r.individuals)
                // })
                unique_observers = h.rows.map(function(value,index) { 
                    return value["names"] + "-" + value["source"]
                }).filter(function(v, i, self){
                    return i == self.indexOf(v)
                });
                                
                // unique_observers.forEach(o => {
                //     if(o.includes("ibp"))
                //         observers += '<div class="col text-nowrap border border-warning table-warning text-center rounded m-1">'+o.replace("-ibp", "")+'</div>'
                //     else if (o.includes("inat"))
                //         observers += '<div class="col text-nowrap border border-success table-success text-center rounded m-1">'+o.replace("-inat", "")+'</div>'
                //     else
                //         observers += '<div class="col text-nowrap border border-info table-info text-center rounded m-1">'+o.replace("-butterfly_counts", "")+'</div>'
                // })

                this.h3Hex[i].species = uniqueSpecies.length
                // this.h3Hex[i].total_individuals = total_individuals
                this.h3Hex[i].unique_observers = unique_observers.length
                this.h3Hex[i].observers = unique_observers

                if(this.speciesMaxValue < this.h3Hex[i].species)
                    this.speciesMaxValue = this.h3Hex[i].species;
            });

            // this.h3Hex.forEach((h,i) => {

            //     function onEachFeature(feature, layer) {
            //         var num = Number(h.species)
            //         layer.bindTooltip(num.toString());
            //         layer.bindPopup(num.toString());
            //         layer.on('click', function (e) {
            //             display_data(i);
            //         });

            //     }

            //     var hexagons = L.geoJSON(h.coordinates.features,{
            //                 style: function (feature) {
            //                     return {color: myColor(h.species)};
            //                 },
            //                 onEachFeature: onEachFeature
            //             // }).addTo(mymap);
            //             });
            //     return hexagons;
            // });

        },
        parseData(data, source){
            if(this.filterSpecies == "ALL" || this.filterSpecies == data.taxa_id){
                this.row = [{
                    "id": data.id,
                    "scientific_name": data.taxa_id,
                    "common_name": "",
                    "individuals": 1,
                    // "names": data.login,
                    // "source": data.source,
                    "image": data.img_url,
                    "uri": data.url
                }]
            }
            return this.row
        },
        hexFeatures(array) {
            const geoJSON = {
                "type": "FeatureCollection",
                "features": [{
                    "type": "Feature",
                    "geometry": {
                        "type": "Polygon",
                        "coordinates": [array.reverse()]
                    }
                }]
            }
            return geoJSON;
        },
        getLatLon(obs){
            return(h3ToGeo(obs.hexID));
        },
        openPopUp (latLng, caller) {
            this.caller = caller;
            this.$refs.features.mapObject.openPopup(latLng);
            this.$emit('clicked-hexagon', caller);
        }
    }
};
</script>
