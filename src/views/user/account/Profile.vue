<template>
  <el-card class="box-card gd-min-height-77vh">
    <el-tabs v-model="localActiveName">
      <el-tab-pane name="details">
        <span slot="label"><i class="el-icon-info"></i>My details</span>
        <edit-profile></edit-profile>
      </el-tab-pane>
      <el-tab-pane name="changePassword">
        <span slot="label"><icon name="lock"></icon>Password</span>
        <change-password></change-password>
      </el-tab-pane>
      <el-tab-pane name="orderHistory">
        <span slot="label"><i class="el-icon-tickets"></i>My orders</span>
        <order-history class="gd-min-height-70vh"></order-history>
      </el-tab-pane>
    </el-tabs>
  </el-card>
</template>

<script>
  import EditProfile from '@/components/Profile/EditProfile.vue'
  import ChangePassword from '@/components/Profile/ChangePassword.vue'
  import OrderHistory from '@/components/Profile/OrderHistory.vue'
  export default {
    components: {
      'edit-profile': EditProfile,
      'change-password': ChangePassword,
      'order-history': OrderHistory
    },
    props: {
      activeName: { type: String }
    },
    data () {
      return {
        localActiveName: ''
      }
    },
    mounted () {
      this.localActiveName = this.activeName
    },
    watch: {
      activeName: function (value) {
        this.localActiveName = this.activeName
      }
    }
  }
</script>

<style>
  i {
    margin-right: 5px;
  }
  .fa-icon {
    margin-right: 5px;
    vertical-align: text-bottom;
  }
</style>

