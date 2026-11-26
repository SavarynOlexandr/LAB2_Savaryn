<script setup>
defineProps({
  item: { type: Object, required: true }
})

defineEmits(['quantity', 'remove'])
</script>

<template>
  <div class="cart-item">
    <img :src="item.product.image" :alt="item.product.name">
    <div class="cart-item-info">
      <h4>{{ item.product.name }}</h4>
      <span class="mode-label">{{ item.mode === 'rent' ? 'Оренда / день' : 'Купівля' }}</span>
      <div class="quantity-row">
        <button type="button" aria-label="Зменшити кількість" @click="$emit('quantity', item, -1)">−</button>
        <span>{{ item.quantity }}</span>
        <button type="button" aria-label="Збільшити кількість" @click="$emit('quantity', item, 1)">+</button>
      </div>
    </div>
    <div class="cart-item-end">
      <strong>{{ (item.unitPrice * item.quantity).toLocaleString('uk-UA') }} ₴</strong>
      <button class="remove-button" type="button" aria-label="Видалити" @click="$emit('remove', item)">×</button>
    </div>
  </div>
</template>
