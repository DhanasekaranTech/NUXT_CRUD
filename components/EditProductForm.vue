<template>
  <div class="modal" v-if="isVisible">
    <div class="modal-content">
      <span class="close" @click="closeModal">&times;</span>
      <h2>Edit Product</h2>
      <form @submit.prevent="handleSubmit">
        <div class="form-group">
          <label for="title">Title</label>
          <input
            type="text"
            id="title"
            v-model="form.title"
            class="form-control"
            required
          />
        </div>
        <div class="form-group">
          <label for="price">Price</label>
          <input
            type="number"
            id="price"
            v-model="form.price"
            class="form-control"
            required
          />
        </div>
        <div class="form-group">
          <label for="brand">Brand</label>
          <input
            type="text"
            id="brand"
            v-model="form.brand"
            class="form-control"
            required
          />
        </div>
        <div class="form-group">
          <label for="category">Category</label>
          <select id="category" v-model="form.category" class="form-control">
            <option value="smartphones">smartphones</option>
            <option value="laptops">laptops</option>
            <option value="fragrances">fragrances</option>
            <option value="skin-care">skincare</option>
            <option value="furniture">furniture</option>
            <option value="motorcycle">motorcycle</option>
          </select>
        </div>
        <div class="form-group">
          <label for="description">Description</label>
          <textarea
            id="description"
            v-model="form.description"
            class="form-control"
          ></textarea>
        </div>
        <button type="submit" class="btn btn-primary mt-3">
          Update Product
        </button>
      </form>
    </div>
  </div>
</template>
  
  <script setup lang="ts">
import { reactive, watch, defineProps, defineEmits } from "vue";
import axios from "axios";
console.log("edit rendered");

interface Product {
  id: string;
  title: string;
  price: number;
  brand: string;
  category: string;
  description: string;
}

const props = defineProps<{
  isVisible: boolean;
  product: Product | null;
}>();

const emit = defineEmits(["close", "productUpdated"]);

const defaultProduct: Product = {
  id: "",
  title: "",
  price: 0,
  brand: "",
  category: "",
  description: "",
};

const form = reactive<Product>({ ...defaultProduct });

watch(
  () => props.product,
  (newProduct) => {
    Object.assign(form, newProduct ? { ...newProduct } : { ...defaultProduct });
  },
  { immediate: true }
);

//To update product
const handleSubmit = async () => {
  try {
    const response = await axios.put(
      `http://localhost:2003/products/${form.id}`,
      form
    );
    emit("productUpdated", response.data);
    closeModal();
  } catch (error) {
    console.error("Error updating product:", error);
  }
};

const closeModal = () => {
  emit("close");
};
</script>
  
  <style scoped>
.modal {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 1000;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 5px;
  width: 500px;
  max-width: 90%;
}

.close {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 20px;
  cursor: pointer;
}
</style>
  