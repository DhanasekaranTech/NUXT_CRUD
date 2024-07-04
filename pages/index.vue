<template>
  <div class="container">
    <h1 class="font mb-5 mt-10px text-dark">Products</h1>

    <div class="d-flex justify-content-evenly mb-3">
      <div class="mb-3">
        <!-- Filter by category of a product-->
        <select class="form-control" v-model="selectedCategory">
          <option value="">All category</option>
          <option value="smartphones">smartphones</option>
          <option value="laptops">laptops</option>
          <option value="fragrances">fragrances</option>
          <option value="skin-care">skincare</option>
          <option value="furniture">furniture</option>
          <option value="motorcycle">motorcycle</option>
        </select>
      </div>

      <div class="mb-3">
        <!-- search by brand of a product -->
        <input
          type="text"
          class="form-control"
          placeholder="search by brand eg.apple"
          v-model="searchTerm"
        />
      </div>

      <!-- Add product to the table -->
      <button class="btn btn-primary mb-3" @click="toggleAddForm">
        Add Product
      </button>
    </div>

    <!-- product table -->
    <table class="table table-success table-striped">
      <thead>
        <tr>
          <th @click="handleSortBy('title')">Name</th>
          <th @click="handleSortBy('price')">Price</th>
          <th>Brand</th>
          <th>Category</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="product in currentProducts" :key="product.id">
          <td>{{ product.title }}</td>
          <td>{{ product.price }}</td>
          <td>{{ product.brand }}</td>
          <td>{{ product.category }}</td>
          <td>
            <button
              class="btn btn-success btn-sm"
              @click="handleViewProduct(product)"
            >
              view
            </button>
            <button
              class="btn btn-warning btn-sm ms-2 me-2"
              @click="handleEditProduct(product)"
            >
              edit
            </button>
            <button
              class="btn btn-danger btn-sm"
              @click="handleDeleteProduct(product.id, product.title)"
            >
              delete
            </button>
          </td>
        </tr>
      </tbody>
    </table>

    <div class="d-flex justify-content-between">
      <button
        class="btn btn-secondary"
        @click="handlePrevPage"
        :disabled="currentPage === 1"
      >
        Previous
      </button>
      <button
        class="btn btn-secondary"
        @click="handleNextPage"
        :disabled="currentPage === totalPages"
      >
        Next
      </button>
    </div>

    <!-- To add product -->
    <AddProductForm
      :isVisible="isAddFormVisible"
      @close="toggleAddForm"
      @productAdded="handleProductAdded"
    />

    <!-- To edit and update product -->
    <EditProductForm
      :isVisible="isEditFormVisible"
      :product="selectedProduct"
      @close="toggleEditForm"
      @productUpdated="handleProductUpdated"
    />
  </div>
</template>
  
  <script setup lang="ts">
import axios from "axios";
import { reactive, ref, computed, onMounted } from "vue";

interface Product {
  id: string;
  title: string;
  price: number;
  brand: string;
  category: string;
  description: string;
}

const state = reactive({
  products: [] as Product[],
});

const searchTerm = ref("");
const selectedCategory = ref("");
const currentPage = ref(1);
const productsPerPage = 10;
const sortBy = ref("title");
const sortOrder = ref("asc");
const isAddFormVisible = ref(false);
const isEditFormVisible = ref(false);
const selectedProduct = ref<Product | null>(null);

const fetchProducts = async () => {
  try {
    const response = await axios.get<Product[]>(
      "http://localhost:2003/products"
    );
    return (state.products = response.data);
  } catch (error) {
    console.log("Error in fetching products" + error);
  }
};

onMounted(fetchProducts);

const filteredProducts = computed(() => {
  return state.products.filter(
    (product) =>
      (selectedCategory.value
        ? product.category === selectedCategory.value
        : true) &&
      (searchTerm.value
        ? product.brand.toLowerCase().includes(searchTerm.value.toLowerCase())
        : true)
  );
});

const sortedProducts = computed(() => {
  return [...filteredProducts.value].sort((a, b) => {
    if (sortBy.value === "title") {
      return sortOrder.value === "asc"
        ? a.title.localeCompare(b.title)
        : b.title.localeCompare(a.title);
    } else if (sortBy.value === "price") {
      return sortOrder.value === "asc" ? a.price - b.price : b.price - a.price;
    }
    return 0;
  });
});

const totalPages = computed(() =>
  Math.ceil(sortedProducts.value.length / productsPerPage)
);

// current products after the functionalities added like Filter, search, sorted
const currentProducts = computed(() => {
  const indexOfLastProduct = currentPage.value * productsPerPage;
  const indexOfFirstProduct = indexOfLastProduct - productsPerPage;
  return sortedProducts.value.slice(indexOfFirstProduct, indexOfLastProduct);
});

const handleSortBy = (field: string) => {
  const order =
    sortBy.value === field && sortOrder.value === "asc" ? "desc" : "asc";
  sortBy.value = field;
  sortOrder.value = order;
};

const handleNextPage = () => {
  if (currentPage.value < totalPages.value) {
    currentPage.value++;
  }
};

const handlePrevPage = () => {
  if (currentPage.value > 1) {
    currentPage.value--;
  }
};

// to view product details via alert
const handleViewProduct = (product: Product) => {
  alert(`Details: 
  -------------------
   name:   ${product.title}
   price:  ${product.price}
   brand:  ${product.brand}
   category:   ${product.category}
  
    ${product.description}`);
};

//to delete the product
const handleDeleteProduct = async (id: string, title: string) => {
  try {
    await axios.delete(`http://localhost:2003/products/${id}`);
    const productList = state.products.filter((product) => product.id !== id);
    state.products = productList;
    return alert(`Product deleted successfully: ${title}`);
  } catch (error) {
    console.error("Error deleting product: ", error);
  }
};

//to add the product to the table via modal
const toggleAddForm = () => {
  isAddFormVisible.value = !isAddFormVisible.value;
};

const handleProductAdded = (newProduct: Product) => {
  state.products.push(newProduct);
  currentPage.value = totalPages.value;
};

const handleProductUpdated = (updatedProduct: Product) => {
  const index = state.products.findIndex((p) => p.id === updatedProduct.id);
  if (index !== -1) {
    state.products[index] = updatedProduct;
  }
};
console.log("index rendered");

//to update the product via modal
const handleEditProduct = (product: Product) => {
  console.log("*********product id", product.id);

  selectedProduct.value = { ...product };
  console.log("******selected product value", selectedProduct.value);

  isEditFormVisible.value = true;
};

const toggleEditForm = () => {
  isEditFormVisible.value = !isEditFormVisible.value;
  selectedProduct.value = null;
};
</script>
  
  