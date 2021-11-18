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
@media only screen and (max-width: 480px){
    .genus-row{
        grid-template-columns: 1fr 1fr;
        grid-gap: 2px;
    }
    .species-card img{
        max-height:calc(40vw / 3 * 2);
        max-width: 40vw;
    }
}

</style>

<template>
    <div id="species-page">
        <n-auto-complete :options="searchAutoCompleteValues" v-model:value="searchValue" placeholder="Search" :clearable="true" />
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
                                <div v-for="species in genusData" class="species-card" :key="species[0]" @click='showSpeciesModal(species[1])'>
                                    <img :src="require('@/assets/book_data/medium/'+species[1].img)" alt="">
                                    <div class="species-name-div">
                                        <span class="species-name">
                                            {{species[0]}}
                                        </span>
                                    </div>
                                    <div>
                                        <!-- {{species[1]}} -->
                                    </div>
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
            <template #header-extra> <span class="close" @click="hideFamilyModal">&times;</span> </template>
            <p v-for="(p, k) in familyModalText" v-text="p" :key="k"></p>
        </n-card>
    </n-modal>
    <n-modal v-model:show="speciesModalDisplay" transform-origin="center">
        <n-card style="width: 80%;" :bordered="false" size="huge">
            <template #header>{{taxonomy_data[selectedSpecies.taxa_id].species}}</template>
            <template #header-extra> <span class="close" @click="hideSpeciesModal">&times;</span> </template>
            <img :src="require('@/assets/book_data/medium/'+selectedSpecies.img)" alt="">
            <h3 class="title2">Observed by: {{user_data[selectedSpecies.user_id][0]}}</h3>
            <h3 class="title2">Photo Date: {{selectedSpecies.observed_on}}</h3>
            <!-- <w-button class="ma1" bg-color="success" color="white" lg @click="gotoSpeciesPage(selectedSpecies)">Go to Species Page</w-button> -->
        </n-card>
    </n-modal>

</template>

<script>
    import { NCard, NModal, NAutoComplete } from 'naive-ui'

    import book_data from '../assets/book_data/book_data.json'
    import taxonomy_data from '../assets/book_data/taxonomy_data.json'
    import user_data from '../assets/book_data/user_data.json'
    import species_descriptions from '../assets/book_data/descriptions.json'

    export default {
        name: 'AllSpecies',
        components: { NCard, NModal, NAutoComplete },
        data() {
            return {
                book_data: book_data,
                user_data: user_data,
                taxonomy_data: taxonomy_data,
                descriptions: species_descriptions,
                tree:{},
                searchValue:"",
                allSearchAutoCompleteValues:[],
                familyModalDisplay: false,
                speciesModalDisplay: false,
                selectedFamily:"",
                selectedSpecies: "",
            }
        },
        created() {
            this.init()
        },
        computed:{
            familyModalText(){
                let op = ""
                if(this.selectedFamily != ""){
                    op = this.descriptionParagraphs(this.selectedFamily)
                }
                return op
            },
            searchAutoCompleteValues(){
                return this.allSearchAutoCompleteValues.filter(str => str.toLowerCase().includes(this.searchValue.toLowerCase()))
            },
            filteredTree(){
                let op = this.tree
                if(this.allSearchAutoCompleteValues.includes(this.searchValue)){
                    let words = this.searchValue.split(" ")
                    let name = ""
                    if(["Superfamily", "Family", "Subfamily", "Tribe", "Genus"].includes(words[0])){
                        op = this.searchTree(words[0], words[1])
                    } else {
                        op = this.searchTree("Species", this.searchValue)
                    }
                }
                return op
            }
        },
        methods: {
            init() {
                let search_terms = new Set();
                this.book_data.forEach(o => {
                    let taxa = this.taxonomy_data[o.taxa_id]
                    if(this.tree[taxa.superfamily] == undefined){
                        this.tree[taxa.superfamily] = {}
                    }
                    if(this.tree[taxa.superfamily][taxa.family] == undefined){
                        this.tree[taxa.superfamily][taxa.family] = {}
                    }
                    if(this.tree[taxa.superfamily][taxa.family][taxa.subfamily] == undefined){
                        this.tree[taxa.superfamily][taxa.family][taxa.subfamily] = {}
                    }
                    if(this.tree[taxa.superfamily][taxa.family][taxa.subfamily][taxa.tribe] == undefined){
                        this.tree[taxa.superfamily][taxa.family][taxa.subfamily][taxa.tribe] = {}
                    }
                    if(this.tree[taxa.superfamily][taxa.family][taxa.subfamily][taxa.tribe][taxa.genus] == undefined){
                        this.tree[taxa.superfamily][taxa.family][taxa.subfamily][taxa.tribe][taxa.genus] = []
                    }
                    this.tree[taxa.superfamily][taxa.family][taxa.subfamily][taxa.tribe][taxa.genus].push([taxa.species, o])

                    search_terms.add("Superfamily " + taxa.superfamily)
                    search_terms.add("Family " + taxa.family)
                    search_terms.add("Subfamily " + taxa.subfamily)
                    search_terms.add("Tribe " + taxa.tribe)
                    search_terms.add("Genus " + taxa.genus)
                    search_terms.add(taxa.species)
                })
                this.allSearchAutoCompleteValues = Array.from(search_terms)
            },
            searchTree(level, search){
                let op = {}
                switch(level){
                    case "Superfamily": op[search] = this.tree[search]
                                        break
                    case "Family": Object.keys(this.tree).forEach(sf => {
                                        if(Object.keys(this.tree[sf]).includes(search)){
                                            op[sf] = {}
                                            op[sf][search] = this.tree[sf][search]
                                        }
                                    })
                                    break
                    case "Subfamily": Object.keys(this.tree).forEach(sf => {
                                        Object.keys(this.tree[sf]).forEach(f => {
                                            if(Object.keys(this.tree[sf][f]).includes(search)) {
                                                op[sf] = {}
                                                op[sf][f] = {}
                                                op[sf][f][search] = this.tree[sf][f][search]
                                            }
                                        })
                                    })
                                    break;
                    case "Tribe": Object.keys(this.tree).forEach(sf => {
                                        Object.keys(this.tree[sf]).forEach(f => {
                                            Object.keys(this.tree[sf][f]).forEach(subf => {
                                                if(Object.keys(this.tree[sf][f][subf]).includes(search)){
                                                    op[sf] = {}
                                                    op[sf][f] = {}
                                                    op[sf][f][subf] = {}
                                                    op[sf][f][subf][search] = this.tree[sf][f][subf][search]
                                                }
                                            })
                                        })
                                    })
                                    break;
                    case "Genus": Object.keys(this.tree).forEach(sf => {
                                        Object.keys(this.tree[sf]).forEach(f => {
                                            Object.keys(this.tree[sf][f]).forEach(subf => {
                                                Object.keys(this.tree[sf][f][subf]).forEach(t => {
                                                    if(Object.keys(this.tree[sf][f][subf][t]).includes(search)){
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
                                    break;
                    case "Species": Object.keys(this.tree).forEach(sf => {
                                        Object.keys(this.tree[sf]).forEach(f => {
                                            Object.keys(this.tree[sf][f]).forEach(subf => {
                                                Object.keys(this.tree[sf][f][subf]).forEach(t => {
                                                    Object.keys(this.tree[sf][f][subf][t]).forEach(g => {
                                                        this.tree[sf][f][subf][t][g].forEach(s => {
                                                            if(s[0] == search){
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
                                    break;
                }

                return op
            },
            cardFamilyTitle (superFamily, family){
                return `Superfamily ${superFamily} > Family ${family}`
            },
            descriptionParagraphs(family) {
                let op = [""]
                if(this.descriptions[family] != undefined)
                    op = this.descriptions[family].split('<br>')
                return op
            },
            gotoSpeciesPage(o){
                let url = "https://www.inaturalist.org/taxa/" + o.taxa_id
                window.open(url, '_blank').focus();
            },
            showFamilyModal(f){
                document.body.style.overflow = 'hidden'
                this.selectedFamily = f
                this.familyModalDisplay = true
            },
            hideFamilyModal(){
                document.body.style.overflow = 'auto'
                this.familyModalDisplay = false
            },
            showSpeciesModal(s){
                document.body.style.overflow = 'hidden'
                this.selectedSpecies = s
                this.speciesModalDisplay = true
            },
            hideSpeciesModal(){
                document.body.style.overflow = 'auto'
                this.speciesModalDisplay = false
            }
        }
    }
</script>
