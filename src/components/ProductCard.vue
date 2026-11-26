<script setup>
defineProps({
  product: { type: Object, required: true }
})

defineEmits(['add'])
</script>

<template>
  <article class="product-card">
    <div class="product-image-wrap">
      <img :src="product.image" :alt="product.name" class="product-image">
      <span class="category-badge">{{ product.category }}</span>
    </div>
    <div class="product-info">
      <h3>{{ product.name }}</h3>
      <div class="price-row">
        <div>
          <span class="price-label">Купити</span>
          <strong>{{ product.price.toLocaleString('uk-UA') }} ₴</strong>
        </div>
        <div>
          <span class="price-label">Оренда / день</span>
          <strong>{{ product.rentalPrice }} ₴</strong>
        </div>
      </div>
      <button class="add-button" type="button" @click="$emit('add', product)">
        Додати до кошика
      </button>
    </div>
  </article>
</template>
