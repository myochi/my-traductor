<script>
  import DxTextArea from "devextreme-vue/text-area";
  import DxSelectBox from "devextreme-vue/select-box";
  import DxButton from "devextreme-vue/button";
  import axios from "axios";
  import { v4 as uuidv4 } from 'uuid';

  let key = "14aab6bd748840beb501bf071317bb42";
  let location = "francecentral";
  let endpoint = "https://api.cognitive.microsofttranslator.com";

  export default {
    data(){
      return {
        text2translate: "Bonjour, bienvenue sur le traducteur, écrivez votre texte et appuyez sur le bouton traduire!",
        textTranslated:"Hello, welcome to the translator, write your text and press the translate button!",
        langages:[],
        langageFrom: {name: "Français", nativeName: "Français", subName: "fr"},
        langageTo: {name: "Anglais", nativeName: "English", subName: "en"},
        charged: undefined
      }
    },
    components:{
      DxSelectBox,
      DxTextArea,
      DxButton
    },
    methods:{
      async translate(){
        var textRes = await axios({
          baseURL: endpoint,
          url: '/translate',
          method: 'post',
          headers: {
              'Ocp-Apim-Subscription-Key': key,
              // location required if you're using a multi-service or regional (not global) resource.
              'Ocp-Apim-Subscription-Region': location,
              'Content-type': 'application/json',
              'X-ClientTraceId': uuidv4().toString()
          },
          params: {
              'api-version': '3.0',
              'from': this.langageFrom.subName,
              'to': [this.langageTo.subName]
          },
          data: [{
              'text': this.text2translate
          }],
          responseType: 'json'
      }).then(function(response){
          return JSON.stringify(response.data[0].translations[0].text, null, 4);
      })
      this.textTranslated = textRes;
      },
      async getLanguages(){
        var allLangagesAvailable = await axios({
            baseURL: endpoint,
            url: '/languages',
            method: 'get',
            headers: {
                'Ocp-Apim-Subscription-Key': key,
                // location required if you're using a multi-service or regional (not global) resource.
                'Ocp-Apim-Subscription-Region': location
            },
            params: {
                'api-version': '3.0'
            }
        }).then(function(response){
            return response.data.translation
        })
        for (const [key, value] of Object.entries(allLangagesAvailable))
        {
            var objLanguage = {
              name: value.name,
              nativeName: value.nativeName,
              subName: key
            };
            this.langages.push(objLanguage);
        }
        console.log(this.langages)
        console.log(this.langageFrom)
        console.log(this.langageTo)
        this.charged = true;
      }
    },
    mounted(){
      this.getLanguages()
    }
  }
</script>
<template>
  <div v-if="charged" class="translator">
    <div class="translator-header">
      <h1>Bienvenue dans votre traducteur!</h1>
    </div>
    <div class="transaltor-body">
      <div class="translator-original">
        <div class="select-list">
          <DxSelectBox 
            :items="langages"
            v-model:value="langageFrom"
            display-expr="nativeName"
          />
        </div>
        <DxTextArea
          height="200px"
          v-model:value="text2translate"
        ></DxTextArea>
      </div>
      <div class="translator-translated">
        <div class="select-list">
          <DxSelectBox
            :items="langages"
            v-model:value="langageTo"
            display-expr="nativeName"
          />
        </div>
        <DxTextArea
          height="200px"
          v-model:value="textTranslated"
        ></DxTextArea>
      </div>
    </div>
    <div class="translator-bottom">
      <DxButton @click="translate()" text="Traduire"></DxButton>
    </div>
  </div>
</template>
<style>
h1{
  text-align: center;
}

.translator{
  width: 511px;
  height: fit-content;
  display: inline-block;
  box-shadow: rgba(0, 0, 0, 0.35) 0px 5px 15px;
  border-radius: 10%;
  padding: 1em;
  margin: auto;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
}
.translator-bottom
{
  display: flex;
  justify-content: right;
  padding: 1em;
}
</style>