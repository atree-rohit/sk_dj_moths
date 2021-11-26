<style>
    #about-page{
        margin:0 10px;
        padding: 10px;
        background: rgba(245,245,247,1);
        min-height: 85vh;
    }
    #about-page .n-collapse-item{
        /*background: -webkit-radial-gradient(center, rgba(131, 131, 131, .15) 0%, rgba(170,170,170,0) 50%);
        background: -moz-radial-gradient(center, rgba(131, 131, 131, .15) 0%, rgba(170,170,170,0) 50%);
        background: radial-gradient(ellipse at center, rgba(131, 131, 131, .15) 0%, rgba(170,170,170,0) 50%);*/
        padding: 2px;
    }
    #about-page .n-collapse-item .n-collapse-item__header{
        background: rgba(131, 131, 131, .2);
        border-radius: 20px;
    }
    #about-page .n-collapse-item__header:hover .n-collapse-item__header-main,
    #about-page .n-collapse-item__header:hover .n-collapse-item-arrow{
        color: #222 !important;
    }
    #about-page .n-collapse-item__header .n-collapse-item__header-main,
    #about-page .n-collapse-item__header .n-collapse-item-arrow{
        font-size: 2rem;
        color: #777 !important;

    }
    #about-page .n-collapse-item__content-inner{
        /*margin: 0 25px;*/
        padding: 16px;
        padding-top: 8px;
    }
</style>

<template>
    <div id="about-page">
        <n-collapse accordion>
            <n-collapse-item title="Moths" name="1">
                <div>Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
                tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
                quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo
                consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse
                cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non
                proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</div>
            </n-collapse-item>
            <n-collapse-item title="Sikkim - Darjeeling Himalaya" name="2">
                <div>Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
                tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
                quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo
                consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse
                cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non
                proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</div>
            </n-collapse-item>
            <n-collapse-item title="This App" name="3">
                <div>Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
                tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
                quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo
                consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse
                cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non
                proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</div>
            </n-collapse-item>
        </n-collapse>
    </div>
</template>

<script>
    import { NCollapse, NCollapseItem } from 'naive-ui'

    export default {
        name: 'AboutPage',
        components: { NCollapse, NCollapseItem },
        data() {
            return {
            }
        },
        methods: {
        }
    }
</script>

