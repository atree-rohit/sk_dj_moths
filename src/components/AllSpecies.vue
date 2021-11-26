<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
    #species-page{
        margin:0 10px;
    }
    .n-card{
        background-color: rgb(50,50,60);
    }
    .family-row{
        background-color: rgb(125,125,150);
        color: rgb(250,250,160);
        margin:  10px 0;
        padding: 10px 5px;
        transition: all .25s;
        font-size: 1.125vw;
        border-radius: 15px 15px 0 0;
    }
    .family-row:hover{
        cursor: pointer;
        background: #069;
        color: #fa0;
    }
    .genus-row{
        padding: 0 20px;
        display: grid;
        grid-template-columns: 1fr 1fr 1fr 1fr 1fr;
        grid-gap: 10px 5px;
        text-align: center;
    }
    .species-card{
        position: relative;
        border: 1px solid #ccc;
        border-radius: 10px;
        box-shadow: 1px 1px 1px #ccc;
        transition: all .3s;
    }
    .species-card:hover{
        background: #ffa;
        cursor: pointer;
        border: 1px solid #cc0;
        box-shadow: 4px 4px 4px #999;
    }

    .species-card:hover .species-name{
        background-color: rgba(200,200,225,1);
    }
    .species-card img{
        height:auto;
        width:auto;
        max-height:calc(17.5vw / 3 * 2);
        max-width: 17.5vw;
        object-fit: cover;
        transition: all .3s;
    }
    .species-name-div{
        transition: all .5s;
        position: absolute;
        bottom: 5px;
        width: 100%;
    }
    .species-name{
        background-color: rgba(200,200,225,.6);
        border-radius: 4px;
        padding: 2px 5px;
        font-style: italic;
    }

    .n-card > .n-card-header{
        padding: 0 20px;
        font-size: 2rem;
        text-align: center;
    }
    .n-card > .n-card-header > .n-card-header__main{
        color: #777 !important;
    }
    .n-card .n-card__content p{
        text-indent: 50px;
        color: #555;
        margin: 10px 0;
    }

    /* The Close Button */
    .close {
      color: #aaa;
      float: right;
      font-size: 28px;
      font-weight: bold;
    }

    .close:hover,
    .close:focus {
      color: black;
      text-decoration: none;
      cursor: pointer;
    }
    #species-modal{
        display: grid;
        width: 100%;
        height: 95%;
        /*max-height: 75vh;*/
        grid-template-columns: 1fr 1fr 20vw;
        grid-template-rows: 25vh 25vh 25vh 5vh;
        grid-template-areas: "image image map"
                            "image image map"
                            "image image chart"
                            "details details details";
        /*grid-template-rows: 2fr 1fr 1fr;*/
        /*grid-template-columns: 1fr 1fr;*/
    }
    #species-modal > div{
        /*border:  1px solid red;*/
        /*margin: 10px;*/
        /*padding: 10px;*/
    }
    #species-modal #image{
        grid-area: image;
        text-align: center;
    }
    #species-modal #image img{
        max-width: 67vw;
        max-height: 75vh;
        height: 100%;
        width: 100%;
        object-fit: contain;
        margin: auto;
    }
    #species-modal #map{
        grid-area: map;
        text-align: center;
    }
    #species-modal #chart{
        grid-area: chart;
        text-align: center;
    }
    #species-modal #details{
        grid-area: details;
        display: flex;
        justify-content: space-between;
        text-align: center;
    }
    .n-collapse-item__content-wrapper{
        transition: all .5s !important;
    }
    @media only screen and (max-width: 480px){
        .genus-row{
            grid-template-columns: 1fr 1fr;
            grid-gap: 2px;
        }
        .species-card img{
            max-height:calc(40vw / 3 * 2);
            max-width: 40vw;
        }
        #species-modal{
            grid-template-areas: "image"
                                "map"
                                "chart"
                                "details";
        }
        .n-modal-container .n-card{
            width: 96% !important;
        }
    }
    .tooltip {
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
    <div id="species-page">
        <n-auto-complete
            :options="searchAutoCompleteValues"
            v-model:value="searchValue"
            placeholder="Search"
        :clearable="true" />
        <n-card :bordered='false'
            content-style="padding:0;"
            v-for="(superfamilyData, superfamily) in filteredTree"
        :key="superfamily">
            <template v-for="(familyData, family) in superfamilyData" :key="family">
                <n-card :bordered='false'
                    class="family-row"
                    v-text="cardFamilyTitle(superfamily, family)"
                    @click="showFamilyModal(family)"
                    ></n-card>
                <div class="genus-row">
                    <template v-for="(subfamilyData, subfamily) in familyData" :key="subfamily">
                        <template v-for="(tribeData, tribe) in subfamilyData" :key="tribe">
                            <template v-for="(genusData, genus) in tribeData" :key="genus">
                                <div v-for="(speciesData, species) in genusData"
                                    class="species-card"
                                    :key="species"
                                    @click='showSpeciesModal(species)'
                                >
                                    <template v-for="observation in speciesData"
                                    :key="observation.id">
                                        <div v-if="observation.img != null">
                                            <img :src="require(`@/assets/book_data/medium/${observation.img}`)">
                                            <div class="species-name-div">
                                                <span class="species-name">
                                                    {{ speciesId(species) }}. {{species}}
                                                </span>
                                            </div>
                                        </div>
                                    </template>
                                </div>
                            </template>
                        </template>
                    </template>
                </div>
            </template>
        </n-card>
    </div>

    <n-modal v-model:show="familyModalDisplay" transform-origin="center">
        <n-card style="width: 80%;" :bordered="false" size="huge">
            <template #header>Family {{selectedFamily}}</template>
            <template #header-extra>
                <span class="close" @click="familyModalDisplay = false">&times;</span>
            </template>
            <p v-for="(p, k) in familyModalText" v-text="p" :key="k"></p>
        </n-card>
    </n-modal>
    <n-modal v-model:show="speciesModalDisplay" transform-origin="center" :on-after-leave="closeModal()" content-style="color:red !important;">
        <n-card style="width: 80%;" :bordered="false" size="huge">
            <template #header>
                <n-button quaternary circle #icon v-if="speciesId(selectedSpecies) > 1" @click='previousSpecies'>
                    <n-icon>
                        <PreviousOutline />
                    </n-icon>
                </n-button>
                <span>{{ speciesId(selectedSpecies) }}. {{selectedSpecies}}</span>
                  <n-button strong quaternary circle #icon v-if="speciesId(selectedSpecies) < allSpeciesList.length" @click="nextSpecies">
                    <n-icon>
                        <NextOutline />
                    </n-icon>
                </n-button>
            </template>
            <template #header-extra> <span class="close" @click="speciesModalDisplay = false">&times;</span> </template>
            <div id="species-modal">
                <div id="image">
                    <img :src="require(`@/assets/book_data/medium/${selectedImageObservation.img}`)" alt=''>
                </div>
                <div id="map">
                    <SpeciesMap :observations="selectedSpeciesData" />
                </div>
                <div id="chart">
                    <DateChart :observations="selectedSpeciesData"/>
                </div>
                <div id="details">
                    <span v-text="speciesImageObserver"></span> &nbsp; &nbsp; &nbsp;
                    <span v-text="speciesImageDate"></span>
                    <span>{{ selectedSpeciesData.length }} Observations</span>
                    <!-- <w-button class="ma1" bg-color="success" color="white" lg @click="gotoSpeciesPage(selectedSpecies)">Go to Species Page</w-button> -->
                </div>
            </div>

        </n-card>
    </n-modal>

</template>

<script>
import moment from 'moment'

import { NCard, NModal, NAutoComplete, NIcon, NButton } from 'naive-ui'
import { PreviousOutline, NextOutline } from '@vicons/carbon'
import SpeciesMap from './SpeciesMap.vue'
import DateChart from './DateChart.vue'

import BookData from '../assets/book_data/book_data.json'
import dataTree from '../assets/book_data/data_tree.json'
import taxonomyData from '../assets/book_data/taxonomy_data.json'
import userData from '../assets/book_data/user_data.json'
import speciesDescriptions from '../assets/book_data/descriptions.json'

export default {
    name: 'AllSpecies',
    components: {
        SpeciesMap, DateChart, NCard, NModal, NAutoComplete, NIcon, NButton, PreviousOutline, NextOutline
    },
    data() {
        return {
            book_data: BookData,
            data_tree: dataTree,
            user_data: userData,
            taxonomy_data: taxonomyData,
            descriptions: speciesDescriptions,
            tree: {},
            searchValue: '',
            allSpeciesList: [],
            allSearchAutoCompleteValues: [],
            familyModalDisplay: false,
            speciesModalDisplay: false,
            modalIsOpen: false,
            selectedFamily: '',
            selectedSpecies: '',
        }
    },
    created() {
        this.init()
        /*
        const sp_obv = {}
        let max = 0
        let max_sp = ""
        Object.keys(this.tree).forEach((sf) => {
            Object.keys(this.tree[sf]).forEach((f) => {
                Object.keys(this.tree[sf][f]).forEach((subf) => {
                    Object.keys(this.tree[sf][f][subf]).forEach((t) => {
                        Object.keys(this.tree[sf][f][subf][t]).forEach((g) => {
                            Object.keys(this.tree[sf][f][subf][t][g]).forEach((s) => {
                                sp_obv[s] = this.tree[sf][f][subf][t][g][s].length
                                if (sp_obv[s] > max && ['Maruca vitrata', 'Spoladea recurvalis', 'Sirinopteryx rufivinctata'].indexOf(s) == -1) {
                                    max = sp_obv[s]
                                    max_sp = s
                                }
                            })
                        })
                    })
                })
            })
        })
        console.log(sp_obv, max, max_sp)
        */
    },
    computed: {
        familyModalText() {
            let op = ''
            if (this.selectedFamily !== '') {
                op = this.descriptionParagraphs(this.selectedFamily)
            }
            return op
        },
        searchAutoCompleteValues() {
            return this.allSearchAutoCompleteValues.filter((str) => str.toLowerCase().includes(this.searchValue.toLowerCase()))
        },
        filteredTree() {
            let op = this.tree
            if (this.allSearchAutoCompleteValues.includes(this.searchValue)) {
                const words = this.searchValue.split(' ')
                if (['Superfamily', 'Family', 'Subfamily', 'Tribe', 'Genus'].includes(words[0])) {
                    op = this.searchTree(words[0], words[1])
                } else {
                    op = this.searchTree('Species', this.searchValue)
                }
            }
            return op
        },
        selectedSpeciesData() {
            let op = []
            Object.keys(this.tree).forEach((sf) => {
                Object.keys(this.tree[sf]).forEach((f) => {
                    Object.keys(this.tree[sf][f]).forEach((subf) => {
                        Object.keys(this.tree[sf][f][subf]).forEach((t) => {
                            Object.keys(this.tree[sf][f][subf][t]).forEach((g) => {
                                Object.keys(this.tree[sf][f][subf][t][g]).forEach((s) => {
                                    if (s == this.selectedSpecies) {
                                        op = this.tree[sf][f][subf][t][g][s]
                                    }
                                })
                            })
                        })
                    })
                })
            })
            return op
        },
        selectedImageObservation() {
            let op = {}
            if (this.selectedSpeciesData.length > 0) {
                this.selectedSpeciesData.forEach((o) => {
                    if (o.img != null) {
                        op = o
                    }
                })
            }
            return op
        },
        speciesImage() {
            return this.selectedImageObservation.img
        },
        speciesImageObserver() {
            let op = 'Observed by: '
            if (this.user_data[this.selectedImageObservation.user_id][0] != null) {
                op += this.user_data[this.selectedImageObservation.user_id][0]
            } else {
                op += this.selectedImageObservation.user_id
            }
            return op
        },
        speciesImageDate() {
            return `Observed on: ${moment(this.selectedImageObservation.observed_on).format('D MMM, YY')}`
        },
    },
    watch: {
        modalIsOpen() {
            document.body.style.overflow = (this.modalIsOpen === true) ? 'hidden' : 'auto'
        },
    },
    methods: {
        init() {
            const searchTerms = new Set();
            this.tree = this.data_tree
            this.book_data.forEach((o) => {
                const taxa = this.taxonomy_data[o.taxa_id]
                searchTerms.add(`Superfamily ${taxa.superfamily}`)
                searchTerms.add(`Family ${taxa.family}`)
                searchTerms.add(`Subfamily ${taxa.subfamily}`)
                searchTerms.add(`Tribe ${taxa.tribe}`)
                searchTerms.add(`Genus ${taxa.genus}`)
                searchTerms.add(taxa.species)
            })
            Object.keys(this.tree).forEach((sf) => {
                Object.keys(this.tree[sf]).forEach((f) => {
                    Object.keys(this.tree[sf][f]).forEach((subf) => {
                        Object.keys(this.tree[sf][f][subf]).forEach((t) => {
                            Object.keys(this.tree[sf][f][subf][t]).forEach((g) => {
                                Object.keys(this.tree[sf][f][subf][t][g]).forEach((s) => {
                                    this.allSpeciesList.push(s)
                                })
                            })
                        })
                    })
                })
            })
            this.allSearchAutoCompleteValues = Array.from(searchTerms)
        },
        speciesId(s) {
            return this.allSpeciesList.indexOf(s) + 1
        },
        searchTree(level, search) {
            const op = {}
            switch (level) {
            case 'Superfamily': op[search] = this.tree[search]
                break
            case 'Family': Object.keys(this.tree).forEach((sf) => {
                if (Object.keys(this.tree[sf]).includes(search)) {
                    op[sf] = {}
                    op[sf][search] = this.tree[sf][search]
                }
            })
                break
            case 'Subfamily': Object.keys(this.tree).forEach((sf) => {
                Object.keys(this.tree[sf]).forEach((f) => {
                    if (Object.keys(this.tree[sf][f]).includes(search)) {
                        op[sf] = {}
                        op[sf][f] = {}
                        op[sf][f][search] = this.tree[sf][f][search]
                    }
                })
            })
                break
            case 'Tribe': Object.keys(this.tree).forEach((sf) => {
                Object.keys(this.tree[sf]).forEach((f) => {
                    Object.keys(this.tree[sf][f]).forEach((subf) => {
                        if (Object.keys(this.tree[sf][f][subf]).includes(search)) {
                            op[sf] = {}
                            op[sf][f] = {}
                            op[sf][f][subf] = {}
                            op[sf][f][subf][search] = this.tree[sf][f][subf][search]
                        }
                    })
                })
            })
                break
            case 'Genus': Object.keys(this.tree).forEach((sf) => {
                Object.keys(this.tree[sf]).forEach((f) => {
                    Object.keys(this.tree[sf][f]).forEach((subf) => {
                        Object.keys(this.tree[sf][f][subf]).forEach((t) => {
                            if (Object.keys(this.tree[sf][f][subf][t]).includes(search)) {
                                op[sf] = {}
                                op[sf][f] = {}
                                op[sf][f][subf] = {}
                                op[sf][f][subf][t] = {}
                                op[sf][f][subf][t][search] = this.tree[sf][f][subf][t][search]
                            }
                        })
                    })
                })
            })
                break
            case 'Species': Object.keys(this.tree).forEach((sf) => {
                Object.keys(this.tree[sf]).forEach((f) => {
                    Object.keys(this.tree[sf][f]).forEach((subf) => {
                        Object.keys(this.tree[sf][f][subf]).forEach((t) => {
                            Object.keys(this.tree[sf][f][subf][t]).forEach((g) => {
                                this.tree[sf][f][subf][t][g].forEach((s) => {
                                    if (s[0] === search) {
                                        op[sf] = {}
                                        op[sf][f] = {}
                                        op[sf][f][subf] = {}
                                        op[sf][f][subf][t] = {}
                                        op[sf][f][subf][t][g] = []
                                        op[sf][f][subf][t][g].push(s)
                                    }
                                })
                            })
                        })
                    })
                })
            })
                break
            // no default
            }

            return op
        },
        cardFamilyTitle(superFamily, family) {
            return `Superfamily ${superFamily} > Family ${family}`
        },
        descriptionParagraphs(family) {
            let op = ['']
            if (this.descriptions[family] !== undefined) {
                op = this.descriptions[family].split('<br>')
            }
            return op
        },
        gotoSpeciesPage(o) {
            const url = `https://www.inaturalist.org/taxa/${o.taxa_id}`
            window.open(url, '_blank').focus()
        },
        showFamilyModal(f) {
            this.modalIsOpen = true
            this.selectedFamily = f
            this.familyModalDisplay = true
        },
        showSpeciesModal(species) {
            this.modalIsOpen = true
            this.selectedSpecies = species
            this.speciesModalDisplay = true
        },
        closeModal() {
            if (this.familyModalDisplay === false && this.speciesModalDisplay === false) {
                this.modalIsOpen = false
            }
        },
        previousSpecies(){
            this.selectedSpecies = this.allSpeciesList[this.speciesId(this.selectedSpecies) - 2]
        },
        nextSpecies() {
            this.selectedSpecies = this.allSpeciesList[this.speciesId(this.selectedSpecies)]
        },
    },
}
</script>
