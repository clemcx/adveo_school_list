<template>
  <section>
    <div class="header d-flex">
      <div class="pageTitle"><div class="stepNumber">1</div></div>
      <hr class="pageActive" />
      <div class="pageTitle"><div class="stepNumber">2</div></div>
      <hr class="new1" />
      <div class="pageTitleActive"><div class="stepNumber">3</div></div>
      <hr class="new1" />
      <div class="pageTitleNext"><div class="stepNumber">4</div></div>
    </div>
    <div class="container">
      <div id="tableProduct" class="col-sm-12">
        <ul class="list-group" id="productsList">
          <li
            class="list-group-item"
            v-for="(product, index) of selectionList"
            v-bind:key="index"
          >
            <div class="referenceItem col-sm-2">
              <p v-if="product.reference_quantity !== 'Null'">
                {{ product.reference_quantity }} {{ product.reference_label }}
              </p>
              <p v-else>
                {{ product.reference_label }}
              </p>
            </div>
            <div
              class="itemsWrapper col-sm-9"
              v-if="product.suggestions.length"
            >
              <div
                class="itembox"
                :class="{ selectedSuggestion: suggestion.selected }"
                v-for="suggestion of product.suggestions"
                v-bind:key="suggestion.sku"
              >
                <div class="d-flex">
                  <a
                    :href="suggestion.url"
                    target="_blank"
                    title="voir le produit"
                  >
                    <img :src="suggestion.picture_url" height="90" alt="" />
                  </a>
                  <div class="d-flex flex-column">
                    <p class="choiceLabel">{{ suggestion.label }}</p>
                    <p>{{ suggestion.sku }}</p>
                    <p>{{ suggestion.brand }}</p>
                    <p>{{ suggestion.price }} €</p>
                  </div>
                </div>

                <button
                  class="trashButton"
                  v-if="suggestion.selected"
                  @click="selectItem(suggestion.sku, index)"
                >
                  <TrashIcon />
                </button>
                <button
                  class="chooseItemBtn"
                  v-else
                  @click="selectItem(suggestion.sku, index)"
                >
                  Choisir
                </button>
              </div>
            </div>
            <div class="itemsWrapper col-sm-9" v-else>
              <div class="noMatchItembox">Aucun produit correspondant</div>
            </div>

            <div>
              <input
                class="form-control"
                type="number"
                min="0"
                v-model="product.quantity"
                @change="updateTotalPrice(), updateItemsTotal()"
              />
            </div>
          </li>
        </ul>
      </div>
    </div>
    <div class="validationFooter col-sm-12">
      <a class="backBtn" href="/edition"> RETOUR </a>
      <div id="listTotal">
        <div class="listTotalContent">
          <div class="totalPrice">
            <div class="text-dark">Prix total :</div>
            <div
              class="totalPriceValue"
              :class="{ isSuperior: totalPrice > listpriceMax }"
              v-if="totalPrice >= 0"
            >
              {{ totalPrice.toFixed(2) }} euros
            </div>
            <div class="totalPriceFail" v-else>
              Vérifiez que tous les produits sélectionnés ont une quantité
            </div>
          </div>
        </div>
      </div>
      <a class="validationBtn" @click="getLink()"> APPLIQUER </a>
    </div>
  </section>
</template>

<script>
// debug values
// const selectionList = []

export default {
  data() {
    return {
      listpriceMax: 0,
      listpriceMin: 0,
      selectionList: [],
      totalQuantity: 0,
      totalPrice: 0,
      totalPriceSuperior: null,
      itemsTotal: 0
    };
  },
  mounted() {
    const data = localStorage.getItem("referenceList");
    let referenceList = JSON.parse(data);
    this.listpriceMax = localStorage.getItem("listPriceMax");
    this.listpriceMin = localStorage.getItem("listPriceMin");

    // let labels = []
    // // let quantities = this.$route.query.quantity.split(',')
    let that = this;

    referenceList.forEach(function(item, index) {
      const label = item.label.normalize("NFD").replace(/[\u0300-\u036f]/g, "");
      let url = `http://doors.retailshake.com:32178/products/_like_this?q=${label}&company_name=Calipage&limit=3`;
      if (item.brands[0]) url += `&brand_name=${item.brands[0]}`;
      if (item.maxPrice) url += `&price_range=0,${item.maxPrice}`;
      const attributes = [];
      if (item.attributes.madeInFrance) attributes.push("Made-in-France-Web");
      if (item.attributes.eco) attributes.push("Eco-responsable");
      if (item.attributes.firstPrice) attributes.push("1erPrix_Scolaire");

      if (attributes.length > 0) {
        url += "&attributes=";
        attributes.forEach(attribute => (url += attribute + ","));
      }

      that.$axios
        .get(
          url

          // `http://doors.retailshake.com:32178/products/_like_this?q=${item.label}&brand_name=BIC&company_name=Calipage&category_label='Coloriage'&limit=3`
          // `http://doors.retailshake.com:32178/products/_like_this?q=${item.label}&company_name=Calipage&size=3`
        )
        .then(response => {
          let products = response.data.products.slice(0, 3);

          let entry = {
            reference_label: item.label,
            suggestions: [],
            quantity: item.quantity,
            reference_quantity: item.quantity
          };
          products.forEach(element => {
            entry.suggestions.push({
              label: element.product_label,
              price: element.last_recommended_price?.toFixed(2),
              brand: element.brand_name,
              sku: element.product_internal_ref,
              picture_url: element.product_picture_url,
              url: element.product_url
            });
          });
          that.selectionList.push(entry);
        })
        .catch(err => {
          console.error(err);
        });
    });
    for (const productIndex in this.selectionList) {
      for (const suggestionIndex in this.selectionList[productIndex]
        .suggestions) {
        this.selectionList[productIndex].suggestions[
          suggestionIndex
        ].selected = false;
      }
    }

    this.totalQuantity = this.selectionList.length;

    for (const product of this.selectionList) {
      this.itemsTotal += product.quantity;
    }
    // totalquantity += product.quantity;
  },
  methods: {
    updateTotalPrice() {
      this.totalPrice = 0;
      for (const product of this.selectionList) {
        for (const suggestion of product.suggestions) {
          if (suggestion.selected) {
            this.totalPrice += suggestion.price * product.quantity;
          }
        }
      }
    },
    updateItemsTotal() {
      this.itemsTotal = 0;
      for (const product of this.selectionList) {
        this.itemsTotal += parseInt(product.quantity);
      }
    },
    selectItem(sku, index) {
      this.selectionList[index].suggestions.map(suggestion => {
        if (suggestion.sku === sku) {
          suggestion.selected = !suggestion.selected;
        } else {
          suggestion.selected = false;
        }
        return suggestion;
      });
      this.updateTotalPrice();
    },
    getLink() {
      const choicesList = [];
      this.selectionList.forEach(product => {
        product.suggestions.forEach(suggestion => {
          if (suggestion.selected) {
            choicesList.push({
              choice: suggestion,
              quantity: product.quantity,
              reference: {
                reference_label: product.reference_label,
                product_quantity: product.reference_quantity
              }
            });
          }
        });
      });
      if (process.client) {
        localStorage.setItem("choicesList", JSON.stringify(choicesList));
        this.$router.push("/validation");
      }
    }
  }
};
</script>

<style scoped>
@import "~/assets/css/main.css";
@import "~/assets/css/selection.css";
</style>
