<template>
  <el-card class="gd_step_body" v-loading="loading">
    <el-row>
      <h3>Order summary</h3>
      <cart-item
      class="gd_cart_item"
      v-for="item in cart.items"
      v-bind:key="item.sku"
      v-bind:item="item"
      v-bind:editable="false"/>
    </el-row>
    <el-row>
      <el-card class="gd_cart_item gd_font">
        <el-row>
          <el-col class="gd_padding_bottom" :md="12" :lg="12">
            <el-row>
              <h3>Payment details</h3>
            </el-row>
            <el-col :span="6" class="hidden-xs-only">
              <img src="http://prints.ultracoloringpages.com/26394191ee3b0e1409d127324c3a5d56.png" class="gd_image">
            </el-col>
            <el-col :lg="18" :sm="18" :md="18" :xs="24" align="left">
              <el-row>
                <el-col :span="13" class="gd_label">Card number: </el-col>
                <el-col :span="11">**** **** **** {{cardDetails.number.substr(15)}}</el-col>
              </el-row>
              <el-row>
                <el-col :span="13"  class="gd_label">Card holder: </el-col>
                <el-col :span="11">{{cardDetails.holder}}</el-col>
              </el-row>
              <el-row>
                <el-col :span="13" class="gd_label">Expiration date: </el-col>
                <el-col :span="11">{{cardDetails.exp_year}}/{{cardDetails.exp_month}}</el-col>
              </el-row>
              <el-row>
                <el-col :span="13" class="gd_label">CVV: </el-col>
                <el-col :span="11">***</el-col>
              </el-row>
            </el-col>
          </el-col>
          <el-col class="gd_padding_bottom gd_border_color" :md="12" :lg="12">
            <el-row>
              <h3>Delivery Address</h3>
            </el-row>
            <el-col :span="6" class="hidden-xs-only">
              <img src="https://seeklogo.com/images/M/man-silhouette-delivery-logo-0DBA9FBE43-seeklogo.com.png" class="gd_image">
            </el-col>
            <el-col :lg="18" :sm="18" :md="18" :xs="24" align="left">
                <el-row>
                  <el-col :span="12" class="gd_label">Name: </el-col>
                  <el-col :span="12">{{address.name}} {{address.surname}}</el-col>
                </el-row>
                <el-row>
                  <el-col :span="12" class="gd_label">Street: </el-col>
                  <el-col :span="12">{{address.street}}</el-col>
                </el-row>
                <el-row>
                  <el-col :span="12" class="gd_label">Postcode: </el-col>
                  <el-col :span="12">{{address.postcode}}</el-col>
                </el-row>
                <el-row>
                  <el-col :span="12" class="gd_label">City: </el-col>
                  <el-col :span="12">{{address.city}}, {{address.country}}</el-col>
                </el-row>
            </el-col>
          </el-col>
        </el-row>
        <div class="gd_summary">
          <el-row class="gd_padding_bottom">
            <el-row>
              <el-col class="gd-text-align-right" :xs="12" :sm="14" :md="16" :lg="17">Subtotal:</el-col>
              <el-col :xs="12" :sm="10" :md="8" :lg="7">{{(subtotal.toFixed(2))}} €</el-col>
            </el-row>
            <el-row>
              <el-col class="gd-text-align-right" :xs="12" :sm="14" :md="16" :lg="17">Discount:</el-col>
              <el-col :xs="12" :sm="10" :md="8" :lg="7">- {{(discount.toFixed(2))}} €</el-col>
            </el-row>
          </el-row>
          <el-row>
            <el-col class="gd-text-align-right" :xs="12" :sm="14" :md="16" :lg="17">Total:</el-col>
            <el-col :xs="12" :sm="10" :md="8" :lg="7">{{(total.toFixed(2))}} €</el-col>
          </el-row>
        </div>
      </el-card>
    </el-row>
    <el-row>
      <el-button @click="$emit('previousStep')">Previous</el-button>
      <el-button type="primary" @click="$emit('performCheckout')">Place order</el-button>
    </el-row>
  </el-card>
</template>

<script>
import CartItem from '@/components/Cart/CartItem.vue'
export default {
  props: {
    cardDetails: { type: Object },
    address: { type: Object },
    loading: { type: Boolean },
    total: { type: Number }
  },
  components: {
    'cart-item': CartItem
  },
  data () {
    return {
      discount: 0,
      subtotal: 0,
      formName: 'Summary',
      cart: {}
    }
  },
  mounted () {
    var cartPromise = this.loadCart()
    Promise.all([cartPromise]).then(() => {
        this.$emit('calculateSubtotal')
        this.$emit('updateSubtotal')
        this.$emit('setLoading', false)
      }).catch((err) => {
        console.log(err)
        this.$emit('setLoading', false)
      })
  },
  methods: {
    loadCart () {
      this.$emit('setLoading', true)
      return this.axios.get(`Cart`)
      .then(response => {
        this.cart = response.data
        this.calculateSubtotal()
        this.calculateDiscount()
      })
      .catch(err => {
        this.$notify.error({
          title: 'Error!',
          message: err.response.data.message,
          offset: 50
        })
      })
    },
    calculateSubtotal () {
      var items = this.cart.items
      this.$nextTick(() => {
        if (items) {
          var arrayLength = items.length
          this.subtotal = 0
          for (var i = 0; i < arrayLength; i++) {
            this.subtotal += items[i].price * items[i].count
          }
        }
      })
    },
    calculateDiscount () {
      var items = this.cart.items
      this.$nextTick(() => {
        if (items) {
          var arrayLength = items.length
          this.discount = 0
          for (var i = 0; i < arrayLength; i++) {
            if (items[i].discount) {
              this.discount += items[i].discount * items[i].count
            } else {
              this.discount += items[i].price * items[i].count
            }
          }
          if (this.discount !== 0) {
            this.discount = this.subtotal - this.discount
          }
        }
      })
    }
  }
}
</script>

<style scoped>
  .gd_font{
    background: #F5F5F5;
  }
  .gd_label {
    font-weight: bold;
  }
  .gd_step_title {
    margin-top: 20px;
    margin-bottom: 30px;
  }
  .gd_image {
    max-width: 60px;
    max-height: 70px;
  }
  .gd_cart_item {
    margin-bottom: 20px;
  }
  .gd_summary{
    padding-top: 20px;
    font-size: 20pt;
  }
  .gd_padding_bottom{
    padding-bottom: 20px;
  }
  .gd_border_color{
    border-bottom: 0.5px outset #8a170617;
  }
  @media (max-width: 768px) {
    .gd_summary{
      font-size: 16pt;
    }
  }
</style>
