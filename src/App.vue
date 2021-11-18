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
</template>

<script>
    import { NMenu } from 'naive-ui'
    import HelloWorld from './components/HelloWorld.vue'
    import AllSpecies from './components/AllSpecies.vue'

    export default {
        name: 'App',
        components: { NMenu, HelloWorld, AllSpecies },
        data() {
            return {
                activeKey: 2,
                menuOptions: [{
                    label: "About",
                    key: 1
                }, {
                    label: "Species",
                    key: 2
                }]
            }
        },
        methods: {
            menuSelect(p){
                this.selectedPage = p
            }
        }
    }
</script>

