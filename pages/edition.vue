<template>
  <section>
    <div class="header d-flex">
      <div class="pageTitle"><div class="stepNumber">1</div></div>
      <hr class="pageActive" />
      <div class="pageTitleActive"><div class="stepNumber">2</div></div>
      <hr class="pageActive" />
      <div class="pageTitleNext"><div class="stepNumber">3</div></div>
      <hr class="new1" />
      <div class="pageTitleNext"><div class="stepNumber">4</div></div>
    </div>

    <div class="container">
      <div class="box col-sm-5">
        <p class="boxTitle ">Prix total</p>
        <div>
          <input
            v-model="listPriceMin"
            placeholder="Min"
            type="number"
            name="minPrice"
            min="0"
          />
          <input
            v-model="listPriceMax"
            placeholder="Max"
            type="number"
            name="maxPrice"
            min="0"
          />
        </div>
      </div>
      <div class="box col-sm-5">
        <p class="boxTitle">Attributs</p>
        <div class="attributes">
          <label>
            <input
              type="checkbox"
              v-model="firstPrice"
              @change="checkAttributes('firstPrice', firstPrice)"
            />
            Premiers Prix
          </label>
          <label>
            <input
              type="checkbox"
              v-model="madeInFrance"
              @change="checkAttributes('madeInFrance', madeInFrance)"
            />
            Fabriqué en France
          </label>
          <label>
            <input
              type="checkbox"
              v-model="eco"
              @change="checkAttributes('eco', eco)"
            />
            Eco-responsable
          </label>
        </div>
      </div>
    </div>
    <div class="container">
      <div id="tableProduct" class="col-sm-12">
        <ul class="list-group" id="productsList">
          <li
            class="list-group-item"
            v-for="(product, index) of referenceList"
            v-bind:key="index"
          >
            <div class="referenceItem col-sm-2">
              <p v-if="product.quantity !== 'Null'">
                {{ product.quantity }} {{ product.label }}
              </p>
              <p v-if="product.quantity == 'Null'">{{ product.label }}</p>
            </div>
            <div>
              <label
                >Marques
                <input
                  @keyup.enter="addBrands($event, index)"
                  placeholder="appuyer sur entrer"
                />
              </label>
              <div
                v-for="(brand, brandIndex) of product.brands"
                v-bind:key="brandIndex"
              >
                {{ brand }}
                <button
                  class="trashButton"
                  @click="referenceList[index].brands.splice(brandIndex, 1)"
                >
                  <TrashIcon />
                </button>
              </div>
            </div>
            <div>
              <label
                >Prix Max
                <input
                  placeholder="Max"
                  type="number"
                  name="maxPrice"
                  v-model="product.maxPrice"
                  min="0"
                />
              </label>
            </div>
            <div class="attributes">
              <label>
                <input
                  type="checkbox"
                  v-model="product.attributes.firstPrice"
                />
                Premiers Prix
              </label>
              <label>
                <input
                  type="checkbox"
                  v-model="product.attributes.madeInFrance"
                />
                Fabriqué en France
              </label>
              <label>
                <input type="checkbox" v-model="product.attributes.eco" />
                Eco-responsable
              </label>
            </div>
          </li>
        </ul>
      </div>
    </div>
    <div class="validationFooter">
      <a class="backBtn " href="/"> RETOUR </a>
      <a class="validationBtn" @click="getLink()"> APPLIQUER </a>
    </div>
  </section>
</template>

<script>
// debug values
const productsList = [];

let totalQuantity = productsList.length;
let totalPrice = 0;
let itemsTotal = 0;

for (const product of productsList) {
  totalPrice += product.price * product.quantity;
  itemsTotal += product.quantity;
}
// totalquantity += product.quantity;

export default {
  data() {
    return {
      listPriceMin: null,
      listPriceMax: null,
      productsList,
      referenceList: [],
      totalQuantity,
      totalPrice,
      itemsTotal,
      madeInFrance: false,
      eco: false,
      firstPrice: false
    };
  },
  mounted() {
    const data = localStorage.getItem("csv");
    let csvData = data.split("\n");
    csvData.forEach(product => {
      let productData = product.split(";");
      this.referenceList.push({
        quantity: productData[0],
        label: productData[1],
        brands: [],
        maxPrice: null,
        attributes: {
          eco: false,
          firstPrice: false,
          madeInFrance: false
        }
      });
    });
  },

  methods: {
    addBrands(e, index) {
      this.referenceList[index].brands.push(e.target.value.toUpperCase());
      e.target.value = null;
    },

    checkAttributes(attribute, value) {
      this.referenceList.forEach(item => {
        item.attributes[attribute] = value;
      });
    },

    getLink() {
      if (process.client) {
        localStorage.setItem(
          "referenceList",
          JSON.stringify(this.referenceList)
        );
        localStorage.setItem("listPriceMin", this.listPriceMin);
        localStorage.setItem("listPriceMax", this.listPriceMax);
        this.$router.push("/selection");
      }
    }
  }
};
</script>

<style scoped>
@import "~/assets/css/main.css";
@import "~/assets/css/edition.css";
</style>
