<template>
  <section>
    <div class="header d-flex">
      <div class="pageTitleActive"><div class="stepNumber">1</div></div>
      <hr class="pageActive" />
      <div class="pageTitleNext"><div class="stepNumber">2</div></div>
      <hr class="new1" />
      <div class="pageTitleNext"><div class="stepNumber">3</div></div>
      <hr class="new1" />
      <div class="pageTitleNext"><div class="stepNumber">4</div></div>
    </div>

    <h1>Edition Liste scolaire</h1>
    <div class="container">
      <div class="formGroup">
        <div class="formInput">
          <label class="formInputTitle">
            <div class="col-sm-2">Intitulé de la Liste</div>
            <input type="text" required v-model="listInfos.listName" />*
          </label>
        </div>
        <div class="formInput">
          <label class="formInputTitle">
            <div class="col-sm-2">Code Postal</div>
            <input type="text" v-model="listInfos.codePostal" />
          </label>
        </div>
        <div class="formInput">
          <label class="formInputTitle">
            <div class="col-sm-2">Etablissement</div>
            <input type="text" required v-model="listInfos.schoolName" /> *
          </label>
        </div>
        <div class="formInput">
          <label class="formInputTitle">
            <div class="col-sm-2">Commentaires</div>
            <input type="textarea" v-model="listInfos.comment" />
          </label>
        </div>
        <div class="formInput">
          <label class="formInputTitle">
            <div class="col-sm-2">Fichier CSV</div>
            <input type="file" accept=".csv" id="file" ref="file" />
          </label>
        </div>
        <div>
          * champ requis
        </div>
      </div>
    </div>
    <div class="sendCsv">
      <a class="validationBtn" @click="submitFile()">ENVOYER</a>
    </div>
  </section>
</template>

<script>
export default {
  /*
      Defines the data used by the component
    */
  data() {
    return {
      file: "",
      listInfos: {
        listName: null,
        codePostal: null,
        schoolName: null,
        comment: null
      }
    };
  },

  methods: {
    /*
        Submits the file to the server
      */
    submitFile() {
      const file = this.$refs.file.files[0];
      let fileContent = "";

      const readFileFunction = new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = e => {
          resolve((fileContent = reader.result));
        };
        reader.readAsText(file);
      });

      if (process.client) {
        if (file && this.listInfos.listName && this.listInfos.schoolName) {
          readFileFunction.then(() => {
            localStorage.setItem("csv", fileContent);
            localStorage.setItem("listInfo", JSON.stringify(this.listInfos));
            this.$router.push("/edition");
          });
        }
      }
    }
    // handleFileUpload() {}
  }
};
</script>
<style scoped>
@import "~/assets/css/index.css";
@import "~/assets/css/main.css";
</style>

//
