<script setup>
import { computed, onMounted, ref, watch } from 'vue'
import CartItem from './components/CartItem.vue'

const products = ref([])
const search = ref('')
const category = ref('Усі')
const sort = ref('default')
const cart = ref([])
const cartOpen = ref(false)
const checkoutOpen = ref(false)
const notice = ref('')

onMounted(async () => {
  const response = await fetch(
    `${import.meta.env.BASE_URL}products.json?v=${Date.now()}`
  )
  products.value = await response.json()
  const savedCart = localStorage.getItem('peak-cart')
  if (savedCart) cart.value = JSON.parse(savedCart)
})

watch(cart, (value) => {
  localStorage.setItem('peak-cart', JSON.stringify(value))
}, { deep: true })

const categories = computed(() =>
  ['Усі', ...new Set(products.value.map((product) => product.category))]
)

const filteredProducts = computed(() => {
  const term = search.value.trim().toLowerCase()
  let result = products.value.filter((product) =>
    (category.value === 'Усі' || product.category === category.value) &&
    product.name.toLowerCase().includes(term)
  )
  if (sort.value === 'low') result = [...result].sort((a, b) => a.price - b.price)
  if (sort.value === 'high') result = [...result].sort((a, b) => b.price - a.price)
  return result
})

const cartCount = computed(() =>
  cart.value.reduce((sum, item) => sum + item.quantity, 0)
)

const cartTotal = computed(() =>
  cart.value.reduce((sum, item) => sum + item.unitPrice * item.quantity, 0)
)

function addToCart(product) {
  const existing = cart.value.find((item) =>
    item.product.id === product.id && item.mode === 'buy'
  )
  if (existing) {
    existing.quantity++
  } else {
    cart.value.push({
      product,
      mode: 'buy',
      quantity: 1,
      unitPrice: product.price
    })
  }
  notice.value = `${product.name} додано до кошика`
  setTimeout(() => { notice.value = '' }, 2200)
}

function addRental(product) {
  const existing = cart.value.find((item) =>
    item.product.id === product.id && item.mode === 'rent'
  )
  if (existing) existing.quantity++
  else cart.value.push({
    product,
    mode: 'rent',
    quantity: 1,
    unitPrice: product.rentalPrice
  })
  notice.value = `${product.name} додано до оренди`
  setTimeout(() => { notice.value = '' }, 2200)
}

function changeQuantity(item, delta) {
  item.quantity += delta
  if (item.quantity < 1) removeItem(item)
}

function removeItem(item) {
  cart.value = cart.value.filter((cartItem) => cartItem !== item)
}

function submitOrder(event) {
  const form = event.target
  if (!form.checkValidity()) return
  notice.value = 'Замовлення прийнято! Ми зв’яжемося з вами для підтвердження.'
  cart.value = []
  checkoutOpen.value = false
  form.reset()
  setTimeout(() => { notice.value = '' }, 4000)
}
</script>

<template>
  <header class="topbar">
    <a class="brand" href="#" aria-label="Peak Store головна">
      <span class="brand-mark">▲</span>
      <span>PEAK <b>STORE</b></span>
    </a>
    <button class="cart-button" type="button" @click="cartOpen = true">
      <span>Кошик</span>
      <b>{{ cartCount }}</b>
    </button>
  </header>

  <main>
    <section class="hero">
      <div>
        <p class="eyebrow">Екіпіруйся для нових вершин</p>
        <h1>Спорядження для<br><em>будь-якої висоти</em></h1>
        <p class="hero-copy">Купуйте перевірене спорядження або беріть його в оренду для наступної гірської мандрівки.</p>
        <a class="hero-button" href="#catalog">Перейти до каталогу</a>
      </div>
    </section>

    <section class="catalog-section" id="catalog">
      <div class="catalog-header">
        <div>
          <p class="eyebrow">Магазин і оренда</p>
          <h2>Обери спорядження</h2>
        </div>
        <div class="controls">
          <label class="search">
            <span>Пошук</span>
            <input v-model="search" type="search" placeholder="Назва спорядження">
          </label>
          <label>
            <span>Категорія</span>
            <select v-model="category">
              <option v-for="item in categories" :key="item">{{ item }}</option>
            </select>
          </label>
          <label>
            <span>Сортування</span>
            <select v-model="sort">
              <option value="default">За замовчуванням</option>
              <option value="low">Спочатку дешевші</option>
              <option value="high">Спочатку дорожчі</option>
            </select>
          </label>
        </div>
      </div>

      <div class="shop-layout">
        <div class="product-grid">
          <article v-for="product in filteredProducts" :key="product.id" class="product-card">
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
              <div class="card-actions">
                <button class="add-button" type="button" @click="addToCart(product)">Купити</button>
                <button class="rent-button" type="button" @click="addRental(product)">Орендувати</button>
              </div>
            </div>
          </article>
          <p v-if="!filteredProducts.length" class="empty-state">За таким запитом нічого не знайдено.</p>
        </div>

        <aside class="cart-panel">
          <div class="cart-heading">
            <h2>Кошик</h2>
            <span>{{ cartCount }} товарів</span>
          </div>
          <div v-if="cart.length" class="cart-list">
            <CartItem
              v-for="item in cart"
              :key="`${item.product.id}-${item.mode}`"
              :item="item"
              @quantity="changeQuantity"
              @remove="removeItem"
            />
          </div>
          <p v-else class="cart-empty">Кошик порожній</p>
          <div class="cart-footer">
            <div><span>Разом</span><strong>{{ cartTotal.toLocaleString('uk-UA') }} ₴</strong></div>
            <button type="button" :disabled="!cart.length" @click="checkoutOpen = true">
              Оформити замовлення
            </button>
          </div>
        </aside>
      </div>
    </section>

    <section class="about" id="about">
      <p class="eyebrow">Peak Store</p>
      <h2>Підготовлені до вершини</h2>
      <p>Допоможемо підібрати спорядження для походу, скелелазіння та зимових трекінгів.</p>
    </section>
  </main>

  <footer id="footer">
    <span>PEAK STORE</span>
    <p>+380 (44) 123-45-67 · hello@peak.store</p>
  </footer>

  <div v-if="checkoutOpen" class="modal-backdrop" @click.self="checkoutOpen = false">
    <section class="checkout-modal" role="dialog" aria-modal="true" aria-labelledby="checkout-title">
      <button class="modal-close" type="button" aria-label="Закрити" @click="checkoutOpen = false">×</button>
      <p class="eyebrow">Оформлення</p>
      <h2 id="checkout-title">Оплата замовлення</h2>
      <form @submit.prevent="submitOrder">
        <fieldset class="payment-cards">
          <legend>Спосіб оплати</legend>
          <label>
            <input type="radio" name="payment" value="visa" checked>
            <span class="card-option visa">VISA</span>
          </label>
          <label>
            <input type="radio" name="payment" value="mastercard">
            <span class="card-option mastercard"><i></i><i></i><b>mastercard</b></span>
          </label>
        </fieldset>
        <label>Номер картки
          <input
            name="card"
            required
            inputmode="numeric"
            autocomplete="cc-number"
            pattern="[0-9 ]{16,19}"
            maxlength="19"
            placeholder="0000 0000 0000 0000"
          >
        </label>
        <label>CVV
          <input
            name="cvv"
            required
            inputmode="numeric"
            autocomplete="cc-csc"
            pattern="[0-9]{3}"
            maxlength="3"
            placeholder="000"
          >
        </label>
        <label>Отримати чек на
          <select name="receiptType" required>
            <option value="phone">Номер телефону</option>
            <option value="email">Email</option>
          </select>
        </label>
        <label>Телефон або пошта
          <input name="receipt" required placeholder="+380 00 000 00 00 або name@example.com">
        </label>
        <div class="checkout-total">
          <span>До сплати</span>
          <strong>{{ cartTotal.toLocaleString('uk-UA') }} ₴</strong>
        </div>
        <button class="submit-button" type="submit">Підтвердити замовлення</button>
      </form>
    </section>
  </div>
  <div v-if="cartOpen" class="modal-backdrop" @click.self="cartOpen = false">
    <section class="cart-modal" role="dialog" aria-modal="true" aria-labelledby="cart-title">
      <button class="modal-close" type="button" aria-label="Закрити" @click="cartOpen = false">×</button>
      <p class="eyebrow">Ваше спорядження</p>
      <h2 id="cart-title">Кошик · {{ cartCount }} товарів</h2>
      <div v-if="cart.length" class="cart-modal-list">
        <CartItem
          v-for="item in cart"
          :key="`${item.product.id}-${item.mode}`"
          :item="item"
          @quantity="changeQuantity"
          @remove="removeItem"
        />
      </div>
      <p v-else class="cart-empty">Кошик порожній</p>
      <div class="cart-modal-footer">
        <div>
          <span>Разом</span>
          <strong>{{ cartTotal.toLocaleString('uk-UA') }} ₴</strong>
        </div>
        <button type="button" :disabled="!cart.length" @click="cartOpen = false; checkoutOpen = true">
          Перейти до оплати
        </button>
      </div>
    </section>
  </div>
  <div v-if="notice" class="toast">{{ notice }}</div>
  <button
    v-if="cart.length"
    class="mobile-cart-bar"
    type="button"
    @click="checkoutOpen = true"
  >
    <span>До оплати · {{ cartCount }} товарів</span>
    <strong>{{ cartTotal.toLocaleString('uk-UA') }} ₴</strong>
    <b>Перейти до оплати</b>
  </button>
</template>
