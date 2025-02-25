<script>
import Supplier from "@/components/Supplier.vue";
import axios from 'axios';


export default {
  components: {Supplier},
  props: {
    name: "SuppliersList"
  },
  data() {
    return {
      suppliers: [],
      filteredSuppliers: [],
      loading: false,
      error: null,
      showOnlyInStock: false,  // Toggle to show only suppliers in stock
    }
  },
  methods: {
    sortByStock() {
      if (this.showOnlyInStock) {
        // If already filtered, reset to show all suppliers
        this.filteredSuppliers = this.suppliers;
      } else {
        // Filter only suppliers with a non-empty status (indicating they're in stock)
        this.filteredSuppliers = this.suppliers.filter(supplier => supplier.status !== '');
      }
      this.showOnlyInStock = !this.showOnlyInStock;  // Toggle the filter status
    }
  },
  created() {
    this.loading = true;
    setTimeout(() => {
      axios
          .get('https://suppliers.depembroke.fr/api/suppliers')
          .then(response => {
            this.suppliers = response.data.data.map(supplier => ({
              id: supplier.id,
              name: supplier.name,
              status: supplier.status,
              checkedAt: supplier.checkedAt,
            }));
          })
          .catch(error => {
            this.error = error.message;
            alert(error.message);
          })
          .finally(() => {
            this.loading = false;
          });
    }, 1000);
  }
};
</script>

<template>
  <div class="suppliers-container">
    <h1>Suppliers List</h1>
    <p v-if="loading"> Wait please... </p>

    <button @click="sortByStock()" style="margin-bottom: 10px;" type="button">Only show products in stock</button>

    <div class="suppliers-list">

      <RouterLink v-for="supplier in suppliers"
                  :key="supplier.id"
                  :to="{ name:'SupplierInfo', params: { id: supplier.id } }">
        <Supplier
            :name="supplier.name"
            :status="supplier.status"
            :checkedAt="supplier.checkedAt"
        />
      </RouterLink>

      <p v-if="error" :class="{errorColor: error}"> {{ error }} </p>

    </div>
  </div>
</template>

<style scoped>
h1 {
  font-size: 2.5rem;
  text-align: center;
  color: #00bd7e;
  margin-bottom: 2rem;
}

/* Container for the entire supplier list */
.suppliers-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 1rem;
}

/* Layout for supplier cards */
.suppliers-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); /* Responsive grid */
  gap: 1.5rem; /* Space between supplier cards */
}

.errorColor {
  color: red;
  font-weight: bold;
  font-size: 1.5rem;
}


</style>