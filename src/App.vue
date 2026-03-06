<template>
  <div class="container">
    <!-- 顶部导航 -->
    <header class="header">
      <div class="header-left">
        <span class="logo">淘宝</span>
        <span class="title">购物车</span>
      </div>
      <div class="header-right">
        <span>淘宝网首页</span>
        <span>已买到的宝贝</span>
        <span>我的淘宝</span>
        <span>购物车{{ checkedGoods.length }}</span>
      </div>
    </header>

    <!-- 购物车主区 -->
    <main class="main">
      <div class="cart-left">
        <div class="cart-tabs">
          <span class="tab active">全部商品({{ cartList.length }})</span>
          <span class="tab">消费券</span>
          <span class="tab">降价</span>
        </div>

        <div class="cart-toolbar">
          <label class="checkbox-label">
            <input 
              type="checkbox" 
              v-model="isAllChecked"
              @change="handleAllCheck"
            > 全选
          </label>
          <button class="btn">移入收藏</button>
          <button class="btn" @click="deleteSelected">删除</button>
          <div class="coupon-notice">
            您有2张共计105元消费券，可尽快使用
            <span class="countdown">距结束 02:31:57</span>
          </div>
        </div>

        <!-- 商品列表 -->
        <div class="cart-list">
          <div 
            class="cart-item" 
            v-for="item in cartList" 
            :key="item.id"
          >
            <input 
              type="checkbox" 
              class="item-check"
              v-model="item.checked"
              @change="updateAllCheckStatus"
            >
            <div class="item-img">
              <img :src="item.image" alt="商品图片">
            </div>
            <div class="item-info">
              <div class="shop-info">
                <span class="shop-tag" :style="{ backgroundColor: item.shopType === '淘宝' ? '#ff4400' : '#ff8800' }">
                  {{ item.shopType }}
                </span>
                <span class="shop-name">{{ item.shopName }}</span>
              </div>
              <div class="item-title">{{ item.title }}</div>
              <div class="item-tags">
                <span class="tag" v-for="tag in item.tags" :key="tag">{{ tag }}</span>
              </div>
              <div class="item-spec">{{ item.spec }}</div>
            </div>
            <div class="item-price">
              <span class="price-sale">¥{{ item.price }}</span>
              <span class="price-original" v-if="item.originalPrice">¥{{ item.originalPrice }}</span>
            </div>
            <div class="item-quantity">
              <button 
                class="qty-btn minus"
                @click="changeQty(item, -1)"
                :disabled="item.quantity <= 1"
              >-</button>
              <input 
                type="number" 
                class="qty-input" 
                v-model.number="item.quantity"
                min="1"
                @change="updateCheckout"
              >
              <button 
                class="qty-btn plus"
                @click="changeQty(item, 1)"
              >+</button>
            </div>
            <div class="item-actions">
              <span class="action">移入收藏</span>
              <span class="action" @click="deleteItem(item.id)">删除</span>
            </div>
          </div>
        </div>
      </div>

      <!-- 结算区 -->
      <div class="cart-right">
        <div class="checkout-header">
          <input type="text" placeholder="床上四件套" class="search-input">
          <button class="search-btn">搜淘宝</button>
        </div>
        <div class="checkout-title">结算明细</div>
        <div class="checkout-desc">实际优惠金额以下单页为准</div>

        <div class="checkout-items">
          <div class="checkout-item">
            <img 
              :src="item.image" 
              alt="商品缩略图" 
              v-for="item in checkedGoods" 
              :key="item.id"
            >
          </div>
        </div>

        <div class="checkout-row">
          <span>商品总价</span>
          <span>¥{{ goodsTotal.toFixed(2) }}</span>
        </div>
        <div class="checkout-row discount">
          <span>店铺优惠</span>
          <span>- ¥{{ shopDiscount.toFixed(2) }}</span>
        </div>
        <div class="checkout-row">
          <span>单品直降</span>
          <span>- ¥{{ itemDiscount.toFixed(2) }}</span>
        </div>
        <div class="checkout-row">
          <span>平台优惠</span>
          <span>- ¥{{ platformDiscount.toFixed(2) }}</span>
        </div>
        <div class="checkout-row">
          <span>红包</span>
          <span>- ¥{{ redPacket.toFixed(2) }}</span>
        </div>

        <div class="checkout-total-row">
          <span>优惠共减 ¥{{ totalDiscount.toFixed(2) }}</span>
          <span>合计 ¥{{ finalTotal.toFixed(2) }}</span>
        </div>

        <div class="checkout-total">
          <span>合计:</span>
          <span class="total-price">¥{{ finalTotal.toFixed(2) }}</span>
        </div>
        <div class="checkout-total-desc">共减 ¥{{ totalDiscount.toFixed(2) }}</div>

        <button 
          class="checkout-btn"
          :disabled="checkedGoods.length === 0"
        >
          结算({{ checkedGoods.length }})
        </button>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

// 模拟购物车数据
const cartList = ref([
  {
    id: 1,
    checked: true,
    image: 'https://via.placeholder.com/80',
    shopType: '淘宝',
    shopName: '名颜美妆化妆品店',
    title: '曼秀雷敦男士保湿酷爽冰露145ml清爽爽肤水保湿不油腻',
    tags: ['全网低价', '退货宝', '7天无理由退货', '极速退款'],
    spec: '适合肤质：保湿酷爽冰露75ml 1瓶',
    price: 23.5,
    originalPrice: '',
    quantity: 1
  },
  {
    id: 2,
    checked: true,
    image: 'https://via.placeholder.com/80',
    shopType: '天猫',
    shopName: '优露士家居专营店',
    title: '威露士老款清香型洗衣液',
    tags: ['信用卡支付', '立减3.40元', '退货宝', '假一赔四', '极速退款', '7天无理由退换'],
    spec: '香味：【原味共2斤】有氧洗衣液1瓶 *1L',
    price: 24.9,
    originalPrice: 28.3,
    quantity: 1
  }
])

// 全选状态
const isAllChecked = ref(true)

// 计算属性：已选中的商品
const checkedGoods = computed(() => {
  return cartList.value.filter(item => item.checked)
})

// 计算属性：商品总价
const goodsTotal = computed(() => {
  return checkedGoods.value.reduce((sum, item) => {
    return sum + item.price * item.quantity
  }, 0)
})

// 优惠相关数据
const shopDiscount = ref(3.4)
const itemDiscount = ref(0)
const platformDiscount = ref(0)
const redPacket = ref(0)

// 总优惠
const totalDiscount = computed(() => {
  return shopDiscount.value + itemDiscount.value + platformDiscount.value + redPacket.value
})

// 最终总价
const finalTotal = computed(() => {
  return Math.max(0, goodsTotal.value - totalDiscount.value)
})

// 全选/取消全选
const handleAllCheck = () => {
  cartList.value.forEach(item => {
    item.checked = isAllChecked.value
  })
}

// 更新全选状态（当单个商品勾选状态变化时）
const updateAllCheckStatus = () => {
  isAllChecked.value = cartList.value.every(item => item.checked)
}

// 改变商品数量
const changeQty = (item, num) => {
  item.quantity = Math.max(1, item.quantity + num)
}

// 删除单个商品
const deleteItem = (id) => {
  cartList.value = cartList.value.filter(item => item.id !== id)
  updateAllCheckStatus()
}

// 删除选中的商品
const deleteSelected = () => {
  cartList.value = cartList.value.filter(item => !item.checked)
  updateAllCheckStatus()
}

// 监听选中商品变化，自动更新结算信息
watch(checkedGoods, () => {
  updateAllCheckStatus()
}, { deep: true })

// 更新结算信息（兼容手动修改数量的情况）
const updateCheckout = () => {
  // 确保数量不小于1
  cartList.value.forEach(item => {
    item.quantity = Math.max(1, item.quantity)
  })
}
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: "Microsoft Yahei", sans-serif;
}

body {
  background-color: #fff;
  color: #333;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

/* 顶部导航 */
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 0;
  border-bottom: 1px solid #eee;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 10px;
}

.logo {
  font-size: 28px;
  font-weight: bold;
  color: #ff4400;
}

.title {
  font-size: 22px;
  color: #333;
}

.header-right {
  display: flex;
  gap: 20px;
  font-size: 14px;
  color: #666;
}

/* 购物车主体 */
.main {
  display: flex;
  gap: 20px;
  margin-top: 20px;
}

.cart-left {
  flex: 1;
}

.cart-right {
  width: 320px;
  border: 1px solid #eee;
  border-radius: 4px;
  padding: 15px;
}

/* 标签栏 */
.cart-tabs {
  display: flex;
  gap: 30px;
  margin-bottom: 15px;
}

.tab {
  font-size: 16px;
  color: #666;
  cursor: pointer;
  padding-bottom: 8px;
}

.tab.active {
  color: #ff4400;
  border-bottom: 2px solid #ff4400;
  font-weight: bold;
}

/* 工具栏 */
.cart-toolbar {
  display: flex;
  align-items: center;
  gap: 15px;
  padding: 10px;
  background-color: #f9f9f9;
  border-radius: 4px;
  margin-bottom: 15px;
}

.checkbox-label {
  display: flex;
  align-items: center;
  gap: 5px;
  cursor: pointer;
}

.btn {
  padding: 5px 10px;
  border: 1px solid #ddd;
  border-radius: 3px;
  background: #fff;
  cursor: pointer;
  font-size: 13px;
}

.coupon-notice {
  flex: 1;
  color: #ff4400;
  font-size: 14px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.countdown {
  color: #333;
}

/* 商品列表 */
.cart-list {
  border: 1px solid #eee;
  border-radius: 4px;
}

.cart-item {
  display: flex;
  align-items: center;
  padding: 15px;
  border-bottom: 1px solid #eee;
  gap: 15px;
}

.cart-item:last-child {
  border-bottom: none;
}

.item-check {
  width: 16px;
  height: 16px;
}

.item-img img {
  width: 80px;
  height: 80px;
  object-fit: cover;
  border-radius: 4px;
}

.item-info {
  flex: 1;
}

.shop-info {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 8px;
}

.shop-tag {
  font-size: 12px;
  padding: 2px 4px;
  border-radius: 2px;
  color: #fff;
}

.shop-name {
  font-size: 14px;
  color: #666;
}

.item-title {
  font-size: 14px;
  margin-bottom: 8px;
  line-height: 1.4;
}

.item-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
  margin-bottom: 8px;
}

.tag {
  font-size: 12px;
  padding: 2px 4px;
  border-radius: 2px;
  color: #ff4400;
  background-color: #fff2f0;
}

.item-spec {
  font-size: 13px;
  color: #999;
}

.item-price {
  width: 80px;
  text-align: center;
  font-size: 14px;
  color: #ff4400;
  font-weight: bold;
}

.price-original {
  font-size: 12px;
  color: #999;
  text-decoration: line-through;
  display: block;
}

.item-quantity {
  display: flex;
  align-items: center;
  width: 100px;
  justify-content: center;
}

.qty-btn {
  width: 24px;
  height: 24px;
  border: 1px solid #ddd;
  background: #fff;
  cursor: pointer;
}

.qty-btn:disabled {
  cursor: not-allowed;
  opacity: 0.5;
}

.qty-input {
  width: 36px;
  height: 24px;
  text-align: center;
  border: 1px solid #ddd;
  border-left: none;
  border-right: none;
}

.item-actions {
  width: 80px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  font-size: 13px;
  color: #666;
}

.action {
  cursor: pointer;
}

.action:hover {
  color: #ff4400;
}

/* 结算区 */
.checkout-header {
  display: flex;
  margin-bottom: 15px;
}

.search-input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px 0 0 4px;
}

.search-btn {
  padding: 8px 15px;
  background-color: #ff4400;
  color: #fff;
  border: none;
  border-radius: 0 4px 4px 0;
  cursor: pointer;
}

.checkout-title {
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 5px;
}

.checkout-desc {
  font-size: 12px;
  color: #999;
  margin-bottom: 15px;
}

.checkout-items {
  margin-bottom: 15px;
}

.checkout-item img {
  width: 40px;
  height: 40px;
  object-fit: cover;
  border-radius: 4px;
  margin-right: 5px;
}

.checkout-row {
  display: flex;
  justify-content: space-between;
  font-size: 14px;
  margin-bottom: 8px;
}

.checkout-row.discount {
  color: #ff4400;
}

.checkout-total-row {
  display: flex;
  justify-content: space-between;
  font-size: 13px;
  color: #999;
  margin-bottom: 15px;
}

.checkout-total {
  display: flex;
  justify-content: space-between;
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 5px;
}

.total-price {
  color: #ff4400;
}

.checkout-total-desc {
  font-size: 12px;
  color: #999;
  text-align: right;
  margin-bottom: 15px;
}

.checkout-btn {
  width: 100%;
  padding: 12px;
  background-color: #ff4400;
  color: #fff;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;
}

.checkout-btn:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
</style>