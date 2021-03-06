<template>
  <el-card class="gd_wrapper" v-loading="loading">
    <h2>Order history</h2>
    <el-card v-if="!orders" style="margin-bottom:40vh;">
      <h3>You do not have any orders yet</h3>
    </el-card>
    <el-card
      v-else
      class="gd_order"
      v-for="order in orders"
      :key="order.id"
      shadow="hover">
        <div slot="header" class="gd_order_header" align="left">
          <el-row>
            <el-col :xs="24" :sm="8" :md="8" :lg="7">
              <el-col :xs="12" :sm="12" :md="12" :lg="12">
                <span class="gd_label">Order no.:</span>
              </el-col>
              <el-col :xs="12" :sm="10" :md="10" :lg="10">
                {{order.orderNumber}}
              </el-col>
            </el-col>
            <el-col :xs="24" :sm="8" :md="8" :lg="8">
              <el-col :xs="12" :sm="12" :md="12" :lg="12">
                <span class="gd_label">Order date:</span>
              </el-col>
              <el-col :xs="12" :sm="10" :md="10" :lg="10">
                {{order.createDate}}
              </el-col>
            </el-col>
            <el-col :xs="24" :sm="9" :md="9" :lg="9">
              <el-col :xs="12" :sm="13" :md="13" :lg="13">
              <span class="gd_label">Order status:</span>
              </el-col>
              <el-col :xs="12" :sm="11" :md="11" :lg="11">
                {{order.status}}
              </el-col>
            </el-col>
          </el-row>
        </div>
        <el-row class="gd_line_bottom_margin">
          <el-col :span="18">
          <span class="gd_label">Total: </span>
          {{order.totalPrice.toFixed(2)}} €
          </el-col>
          <el-col :span="6">
            <el-tooltip placement="bottom" content="Add all items to cart">
              <el-button size="small" @click="addItemsToCart(order.items)" align="right">
                <icon name="cart-arrow-down"/>
              </el-button>
            </el-tooltip>
          </el-col>
        </el-row>
        <el-collapse>
          <el-collapse-item>
            <template slot="title">
              <span class="gd_label">{{countItems(order.items)}}</span>
            </template>
            <div v-for="item in order.items" :key="item.itemID" :item="item">
            <el-row class="gd_item_name_row">
              <el-col :xs="24" :sm="19" :md="19" :lg="19">
              {{item.name}}
              </el-col>
              <el-col :xs="24" :sm="5" :md="5" :lg="5" align="right">
                <el-tooltip
                  placement="top"
                  content="View item"
                  effect="dark">
                  <el-button
                    icon="el-icon-search"
                    @click="viewItemDetails(item.itemID)"
                    size ="small"
                    circle>
                  </el-button>
                </el-tooltip>
                <el-tooltip
                  placement="top"
                  content="Add item to cart"
                  effect="dark">
                    <el-button
                      @click="addItemToCart(item.itemID)"
                      size ="small"
                      circle>
                      <icon name="cart-plus"/>
                    </el-button>
                </el-tooltip>
              </el-col>
            </el-row>
            <el-row>
              <el-col :xs="9" :sm="4" :md="4" :lg="4">
                <span class="gd_label gd_gray_text">Unit price:</span>
              </el-col>
              <el-col :xs="7" :sm="5" :md="5" :lg="5">
                <span class="gd_gray_text">{{item.price.toFixed(2)}} €</span>
              </el-col>
            </el-row>
            <el-row>
              <el-col :xs="9" :sm="4" :md="4" :lg="4">
                <span class="gd_label gd_gray_text">Quantity:</span>
              </el-col>
              <el-col :xs="7" :sm="5" :md="5" :lg="5">
                <span class="gd_gray_text">{{item.count}}</span>
              </el-col>
            </el-row>
            </div>
          </el-collapse-item>
          <el-collapse-item>
            <template slot="title">
              <span class="gd_label">Delivery address</span>
            </template>
            <div class="gd_gray_text">
              <el-row>{{order.deliveryAddress.Name}} {{order.deliveryAddress.Surname}}</el-row>
              <el-row>{{order.deliveryAddress.Street}}</el-row>
              <el-row>{{order.deliveryAddress.City}}</el-row>
              <el-row>{{order.deliveryAddress.Country}} {{order.deliveryAddress.Postcode}}</el-row>
            </div>
          </el-collapse-item>
        </el-collapse>
      </el-card>
      <el-row v-if="orders">
        <el-col class="hidden-xs-only">
          <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page.sync="currentPage"
            :page-sizes="pageOptions"
            :page-size="perPage"
            background
            layout="total, sizes, prev, pager, next, jumper"
            :total="totalOrders">
          </el-pagination>
        </el-col>
        <el-col class="hidden-sm-and-up">
          <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :page-sizes="pageOptions"
            :page-size="perPage"
            small
            layout="total, sizes"
            :total="totalOrders">
          </el-pagination>
          <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page.sync="currentPage"
            :page-size="perPage"
            :pager-count="5"
            background
            small
            layout="prev, pager, next"
            :total="totalOrders">
          </el-pagination>
        </el-col>
      </el-row>
      <el-row v-else>
        <h1 style="padding-top: 10%">Your order history is empty</h1>
      </el-row>
  </el-card>
</template>

<script>
import EventBus from '@/eventBus'
export default {
  data () {
    return {
      orders: [],
      loading: '',
      itemsText: 'items',
      perPage: 10,
      totalOrders: 0,
      pageOptions: [5, 10, 15, 20],
      currentPage: 1
    }
  },
  mounted () {
    this.fetchData()
  },
  methods: {
    countItems (items) {
      var length = items.length
      if (length === 1) return length + ' item'
      return length + ' items'
    },
    viewItemDetails (id) {
      this.$router.push({name: 'itemdetails', params: {id: id}})
    },
    handleSizeChange (pageSize) {
      this.perPage = pageSize
      this.fetchData()
    },
    handleCurrentChange (currentPage) {
      this.currentPage = currentPage
      this.fetchData()
    },
    fetchData () {
      this.loading = true

      var ordersCountPromise = this.axios.get(`odata/Orders?$count=true&$top=0`)
      ordersCountPromise.then(response => {
        this.totalOrders = response.data['@odata.count']
      })

      var ordersPromise = this.axios.get(`odata/Orders?$expand=items&$skip=${this.perPage * (this.currentPage - 1)}&$top=${this.perPage}&$orderby=createDate desc`)
      ordersPromise.then(response => {
        this.orders = response.data.value
        this.parseAddressesToObjects()
      })

      Promise.all([ordersCountPromise, ordersPromise]).then(() => {
        this.loading = false
      }).catch(this.handleError)
    },
    parseAddressesToObjects () {
      var length = this.orders.length
      for (var i = 0; i < length; i++) {
        this.orders[i].deliveryAddress = JSON.parse(this.orders[i].deliveryAddress)
      }
    },
    addItemsToCart (items) {
      if (items === null) return
      var itemsList = []
      for (var i = 0; i < items.length; i++) {
        itemsList.push({ItemID: items[i].itemID, Count: 1})
      }
      this.loading = true
      this.axios.post('cart', {Items: itemsList})
        .then(() => {
          this.loading = false
          this.$notify.success({
            title: 'Success',
            message: 'Items were added to cart.',
            offset: 50
          })
          EventBus.$emit('updateCartCount')
        })
        .catch(this.handleError)
    },
    addItemToCart (id) {
      this.loading = true
      var newItem = {
        ItemID: id,
        Count: 1
      }
      this.axios.put(`cart`, newItem).then(() => {
        this.loading = false
        this.$notify.success({
          title: 'Success',
          message: 'Item was added to cart.',
          offset: 50
        })
        EventBus.$emit('updateCartCount')
      })
      .catch(this.handleError)
    },
    handleError (err) {
      this.loading = false
      if (err.cookieExpired) {
        EventBus.$emit('cookieExpired')
        return
      }
      this.$notify.error({
        title: 'Error',
        message: err.response.data.message,
        offset: 50
      })
    }
  }
}
</script>

<style scoped>
  .gd_order {
    max-width: 600px;;
    margin: 20px auto;
    padding: 5px;
    text-align: left;
  }
  .gd_wrapper {
    margin: auto;
    max-width: 700px;
    margin-top: 40px;
  }
  h2 {
    margin: 20px 0;
  }
  .gd_label {
    font-weight: bold;
  }
  .gd_gray_text {
    color: gray;
  }
  .gd_line_bottom_margin {
    margin-bottom: 10px;
  }
  .gd_item_name_row {
    border-top: 1px solid lightgray;
    padding-top: 5px;
    color: #DF3A01;
  }
  .el-button {
    margin-left: 1px;
  }
</style>
