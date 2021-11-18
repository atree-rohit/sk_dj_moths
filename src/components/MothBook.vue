<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.app{
    margin-top:60px;
}
.genus-row{
    display: flex;
    justify-content: center;
    flex-wrap: wrap;

}
imga{
    height:auto;
    width:auto;
    max-height:200px;
    max-width:300px;
}
.w-card__content{
    padding:0 !important;

}
</style>

<template>
    <div class="app">
        <w-card v-for="(superfamilyData, superfamily) in tree" :key="superfamily">
            <div class="title1"
                :class="selected_class('superfamily', superfamily)"
                v-text="superfamily"
                @click="select_taxa('superfamily', superfamily)"
                ></div>
            <div v-if="selected.superfamily == superfamily">
                <w-card v-for="(familyData, family) in superfamilyData" :key="family">
                    <div class="title2"
                        :class="selected_class('family', family)"
                        v-text="family"
                        @click="select_taxa('family', family)"
                        ></div>
                    <div v-if="selected.family == family">
                        <w-card v-for="(subfamilyData, subfamily) in familyData" :key="subfamily">
                            <div class="title2"
                                :class="selected_class('subfamily', subfamily)"
                                v-text="subfamily"
                                @click="select_taxa('subfamily', subfamily)"
                                ></div>
                            <div v-if="selected.subfamily == subfamily">
                                <w-card v-for="(tribeData, tribe) in subfamilyData" :key="tribe">
                                    <div class="title3"
                                        :class="selected_class('tribe', tribe)"
                                        v-text="tribe"
                                        @click="select_taxa('tribe', tribe)"
                                        ></div>
                                    <div v-if="selected.tribe == tribe">
                                        <w-card v-for="(genusData, genus) in tribeData" :key="genus">
                                            <div class="title4"
                                                :class="selected_class('genus', genus)"
                                                v-text="genus"
                                                @click="select_taxa('genus', genus)"
                                                ></div>
                                            <div v-if="selected.genus == genus" class="genus-row">
                                                <w-card v-for="species in genusData" :key="species[0]">
                                                    <div>
                                                        <img :src="require('@/assets/book_data/medium/'+species[1].img)" alt="">
                                                    </div>
                                                    <div class="title5"
                                                        v-text="species[0]"
                                                        ></div>
                                                    <div>
                                                        {{species[1]}}
                                                    </div>

                                                </w-card>
                                            </div>
                                        </w-card>
                                    </div>
                                </w-card>
                            </div>
                        </w-card>
                    </div>
                </w-card>
            </div>

            <!-- <img :src="require('@/assets/book_data/medium/'+o.img)" alt=""> -->
        </w-card>
    </div>
</template>

<script>
import book_data from '../assets/book_data/book_data.json'
import taxonomy_data from '../assets/book_data/taxonomy_data.json'
import user_data from '../assets/book_data/user_data.json'

export default {
  name: 'MothBook',
  data() {
      return {
          book_data: book_data,
          user_data: user_data,
          taxonomy_data: taxonomy_data,
          data:{},
          tree:{},
          levels: ["superfamily", "family", "subfamily", "tribe", "genus", "species"],
          selected:{}
      }
  },
  created() {
      this.init()
  },
  methods: {
      init() {
          this.levels.forEach(l => {
              this.selected[l] = ''
          })
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
          })
          console.log(this.tree)
      },
      selected_class(l, t){
          let op = ""
          if(this.selected[l] == t){
              op = 'green'
              switch (l) {
                  case 'superfamily': op += '-dark3'
                  break
                  case 'family': op += '-dark2'
                  break
                  case 'subfamily': op += '-dark1'
                  break
                  case 'tribe': op += '-light1'
                  break
                  case 'genus': op += '-light2'
                  break
                  case 'species': op += '-light3'
                  break
              }
          }
          op += '--bg'
          return op
      },
      select_taxa(l, t){
        if(this.selected[l] == t){
            this.selected[l] = ''
        } else {
            this.selected[l] = t
        }

      },
      img_url(img, size){
          let op = "@/assets/book_data/"
          if(size == 'm'){
              op += "medium/"
          } else if(size == 'l'){
              op += "large/"
          }
          op += img
          return op
      }
  }
}
</script>

