<template>
  <section>
    <div class="header d-flex">
      <div class="pageTitle"><div class="stepNumber">1</div></div>
      <hr class="pageActive" />
      <div class="pageTitle"><div class="stepNumber">2</div></div>
      <hr class="pageActive" />
      <div class="pageTitle"><div class="stepNumber">3</div></div>
      <hr class="pageActive" />
      <div class="pageTitleActive"><div class="stepNumber">4</div></div>
    </div>
    <div class="container">
      <div id="tableProduct" class="col-sm-12">
        <ul class="list-group" id="productsList">
          <li
            class="list-group-item"
            v-for="product of choicesList"
            v-bind:key="product.reference.reference_label"
          >
            <div class="referenceItem col-sm-2">
              <!-- <p v-if="product.reference_quantity !== 'Null'">{{ product.reference_quantity }} {{ product.reference_label }}</p>
              <p v-if="product.reference_quantity == 'Null'">{{ product.reference_label }}</p> -->
              <p v-if="product.reference.product_quantity == 'Null'">
                {{ product.reference.reference_label }}
              </p>
              <p v-if="product.reference.product_quantity !== 'Null'">
                {{ product.reference.product_quantity }}
                {{ product.reference.reference_label }}
              </p>
            </div>

            <div scope="row" class="productPicture">
              <img :src="product.choice.picture_url" height="80" />
            </div>
            <div class="col-sm-3">{{ product.choice.label }}</div>
            <div class="col-sm-2">{{ product.choice.sku }}</div>
            <div class="col-sm-1">{{ product.quantity }}</div>
            <div class="productPrice col-sm-1">{{ product.choice.price }}€</div>
          </li>
        </ul>
      </div>
    </div>
    <div class="validationFooter">
      <a class="backBtn " href="/selection"> RETOUR </a>
      <div id="listTotal">
        <div class="listTotalContent">
          <div hidden>Nombre de produits: {{ totalQuantity }}</div>
          <div hidden>total produits : {{ itemsTotal }}</div>
          <div class="totalPriceValue">
            <span class="text-dark">Prix total:</span>
            {{ totalPrice.toFixed(2) }} euros
          </div>
        </div>
      </div>
      <button class="validationBtn" @click="dowloadCsv()">
        EXPORTER CSV
      </button>
    </div>
  </section>
</template>

<script>
export default {
  data() {
    return {
      listInfos: {},
      choicesList: [],
      totalQuantity: 0,
      totalPrice: 0,
      itemsTotal: 0
    };
  },
  mounted() {
    const data = localStorage.getItem("choicesList");
    this.choicesList = JSON.parse(data);
    this.listInfos = JSON.parse(localStorage.getItem("listInfo"));

    this.totalQuantity = this.choicesList.length;

    for (const product of this.choicesList) {
      this.totalPrice += product.choice.price * product.quantity;
      this.itemsTotal += product.quantity;
    }
  },

  methods: {
    dowloadCsv() {
      var csv = "Sku;quantity\n";
      //merge the data with CSV
      this.choicesList.forEach(function(product) {
        csv += `${product.choice.sku};${product.quantity}\n `;
      });

      var hiddenElement = document.createElement("a");
      hiddenElement.href = "data:text/csv;charset=utf-8," + encodeURI(csv);
      // hiddenElement.target = "_blank";

      // provide the name for the CSV file to be downloaded
      const { schoolName, listName } = this.listInfos;
      hiddenElement.download = `${schoolName} ${listName}.csv`;

      hiddenElement.click();
    }
  }
};
</script>

<style scoped>
@import "~/assets/css/main.css";
</style>
