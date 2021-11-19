<style>
    body{
        background: rgb(24,24,28);
        color: #ccc;
    }

    #header-row{
        display: flex;
        justify-content: space-between;
        border-bottom: 1px solid rgba(200,200,200,.2);
        box-shadow: 0px 2px 10px rgba(200,200,200,.2);
    }
    #logo{
        max-width: 300px;
        padding: 10px;
    }
    #logo-text{
        font-size: 1.2vw !important;
        font-weight: 700 !important;
    }
    #menu{
        vertical-align: middle;
        margin: auto 0;
        color: red;
    }
    .n-menu .n-menu-item-content .n-menu-item-content-header{
        color: #ccc;
    }
    @media only screen and (max-width: 480px){
        #logo-text{
            font-size: 3.5vw !important;
        }
    }
</style>

<template>
    <div id="header-row">
        <div id="logo">
            <span id="logo-text">Sikkim - Darjeeling Moths</span>
        </div>
        <div id="menu">
            <n-menu v-model:value="activeKey" mode="horizontal" :options="menuOptions" />
        </div>
    </div>
    <AllSpecies v-if="activeKey == 2"/>
    <SpeciesMap :observations="speciesData" v-if="activeKey == 3"/>
</template>

<script>
    import SpeciesMap from './components/SpeciesMap.vue'


    import { NMenu } from 'naive-ui'
    import HelloWorld from './components/HelloWorld.vue'
    import AllSpecies from './components/AllSpecies.vue'

    export default {
        name: 'App',
        components: { NMenu, HelloWorld, AllSpecies, SpeciesMap },
        data() {
            return {
                activeKey: 2,
                menuOptions: [{
                    label: "About",
                    key: 1
                }, {
                    label: "Species",
                    key: 2
                }],
                speciesData:[ { "id": 4461268, "observed_on": "2016-10-10", "location": "27.089,88.5034", "place_guess": "Delo Park", "taxa_id": 124651, "user_id": "subhajit_roy", "img": null }, { "id": 4965930, "observed_on": "2016-10-11", "location": "27.2179,88.7423", "place_guess": "Subanedara", "taxa_id": 124651, "user_id": "subhajit_roy", "img": null }, { "id": 31226751, "observed_on": "2015-12-05", "location": "27.3382,88.614", "place_guess": "Sungava, Gangtok, Sikkim", "taxa_id": 124651, "user_id": "rohitmg", "img": null }, { "id": 31269837, "observed_on": "2015-12-09", "location": "27.3369,88.6131", "place_guess": "Upper Sichey, Gangtok, Sikkim 737101", "taxa_id": 124651, "user_id": "rohitmg", "img": "Drepana_pallida.jpg" }, { "id": 31270291, "observed_on": "2015-12-13", "location": "27.3372,88.614", "place_guess": "Upper Sichey, Gangtok, Sikkim 737101", "taxa_id": 124651, "user_id": "rohitmg", "img": null }, { "id": 38826848, "observed_on": "2019-10-18", "location": "27.4716,88.5093", "place_guess": "Hee-Gyathang, Sikkim 737116", "taxa_id": 124651, "user_id": "rohitmg", "img": null }, { "id": 38826885, "observed_on": "2019-10-18", "location": "27.4716,88.5093", "place_guess": "Hee-Gyathang, Sikkim 737116", "taxa_id": 124651, "user_id": "rohitmg", "img": null }, { "id": 38830579, "observed_on": "2019-10-18", "location": "27.4716,88.5093", "place_guess": "Hee-Gyathang, Sikkim 737116", "taxa_id": 124651, "user_id": "rohitmg", "img": null }, { "id": 87624105, "observed_on": "2021-07-19", "location": "26.8945,88.2816", "place_guess": "Kurseong, West Bengal", "taxa_id": 124651, "user_id": "prayash29", "img": null }, { "id": 87729905, "observed_on": "2021-07-19", "location": "26.933,88.2315", "place_guess": "Moonda Kotee Tea Garden", "taxa_id": 124651, "user_id": "meghna25", "img": null }, { "id": 88481918, "observed_on": "2021-07-25", "location": "27.0398,88.2659", "place_guess": "Darjeeling town, Darjeeling, West Bengal", "taxa_id": 124651, "user_id": "bonpradhan", "img": null }, { "id": 88824000, "observed_on": "2021-07-27", "location": "27.0398,88.2659", "place_guess": "Darjeeling town, Darjeeling, West Bengal", "taxa_id": 124651, "user_id": "bonpradhan", "img": null }, { "id": 88946410, "observed_on": "2021-07-28", "location": "27.0398,88.2659", "place_guess": "Darjeeling town, Darjeeling, West Bengal", "taxa_id": 124651, "user_id": "bonpradhan", "img": null }, { "id": 91072051, "observed_on": "2014-01-30", "location": "27.3354,88.6115", "place_guess": "Sungava, Gangtok, Sikkim", "taxa_id": 124651, "user_id": "drnamgyal04", "img": null } ]
            }
        },
        methods: {
            menuSelect(p){
                this.selectedPage = p
            }
        }
    }
</script>

