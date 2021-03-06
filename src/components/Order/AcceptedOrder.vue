<template>
  <div>
    <v-hover
      v-slot:default="{ hover }"
      open-delay="200"
    >
      <v-card
        :elevation="hover ? 16 : 2"
        class="mx-auto transition-swing"
        max-width="800"
      >
        <v-toolbar
          flat
          class="body-1 pt-2"
        >
          <v-toolbar-title>{{ $t('order.acceptedOrder.title') }}</v-toolbar-title>
          <v-divider class="mx-4" />
        </v-toolbar>

        <v-container
          v-if="loading"
          style="text-align: center;"
        >
          <div class="loadingio-spinner-ripple-s9nf2nweyv">
            <div class="ldio-62v9m71m4dc">
              <div></div>
              <div></div>
            </div>
          </div>
        </v-container>

        <v-container v-else>
          <div v-if="orders.length == 0">
            <v-card-title>{{ $t('order.acceptedOrder.noOrderTitle') }}</v-card-title>
            <v-card-subtitle>{{ $t('order.acceptedOrder.noOrderSubTitle') }}</v-card-subtitle>
          </div>
          <div v-if="orders.length != 0">
            <template v-for="order in orders">
              <v-card
                v-bind:key="order.order_id"
                class="mx-auto mb-6"
                max-width="calc(100% - 36px)"
                elevation="2"
              >

                <v-card-title>
                  {{ order.order_user_dormitory }}
                </v-card-title>

                <v-card-subtitle>
                  {{ order.order_user_name }} | {{ order.order_user_gender }} | {{ order.order_user_campus }} | {{ order.order_user_telephone }}
                </v-card-subtitle>

                <v-card-text class="pb-2">
                  <v-row>
                    <v-col cols="auto">
                      <v-btn
                        v-if="order.order_user_telephone"
                        color="blue-grey"
                        dark
                        @click="telephoneCall(order.order_user_telephone)"
                        class="mr-2"
                      >
                        <v-icon left>mdi-phone-in-talk-outline</v-icon> {{ $t('order.acceptedOrder.telephoneCall') }}
                      </v-btn>

                      <v-btn
                        color="success"
                        @click="finishOrder(order)"
                        class="mr-2"
                      >
                        <v-icon left>mdi-checkbox-multiple-marked</v-icon> {{ $t('order.acceptedOrder.finishOrder') }}
                      </v-btn>
                    </v-col>

                    <v-spacer />

                    <v-col cols="auto">
                      <v-btn
                        icon
                        @click="expandPanel(order.order_id)"
                      >
                        <v-icon>{{ expandShow === order.order_id ? 'mdi-chevron-up' : 'mdi-chevron-down' }}</v-icon>
                      </v-btn>
                    </v-col>
                  </v-row>
                </v-card-text>

                <v-expand-transition>
                  <div v-show="expandShow === order.order_id">
                    <v-divider />

                    <v-card-subtitle>{{ $t('order.acceptedOrder.expanded.orderId') }}{{ order.order_id }}</v-card-subtitle>

                    <v-card-text class="pb-2">
                      <v-row>
                        <!-- 订单描述 -->
                        <v-col
                          cols="12"
                          v-if="order.order_user_description"
                        >
                          <v-chip
                            label
                            outlined
                            class="mr-2"
                          >
                            <v-icon
                              small
                              left
                            >mdi-calendar-blank-outline</v-icon>
                            {{ $t('order.acceptedOrder.expanded.description') }}
                          </v-chip>
                          <span
                            class="body-1"
                            style="display: inline-block;"
                          >{{ order.order_user_description }}</span>
                        </v-col>
                        <!-- 订单创建日期 -->
                        <v-col cols="12">
                          <v-chip
                            label
                            outlined
                            class="mr-2"
                          >
                            <v-icon
                              small
                              left
                            >mdi-calendar-blank-outline</v-icon>
                            {{ $t('order.acceptedOrder.expanded.createDate') }}
                          </v-chip>
                          <span
                            class="body-1"
                            style="display: inline-block;"
                          >{{ order.create_date }}</span>
                        </v-col>
                        <!-- 订单经过时间 -->
                        <v-col cols="12">
                          <v-chip
                            label
                            outlined
                            class="mr-2"
                          >
                            <v-icon
                              small
                              left
                            >mdi-progress-clock</v-icon>
                            {{ $t('order.acceptedOrder.expanded.openTime') }}
                          </v-chip>
                          <span
                            class="body-1"
                            style="display: inline-block;"
                          >{{ order.order_open_time }}</span>
                        </v-col>
                        <!-- 订单额外操作
                        <v-col cols="12">
                          <v-btn
                            small
                            depressed
                            @click="cancelOrder(order)"
                          >{{ $t('order.acceptedOrder.expanded.cancelOrder') }}</v-btn>
                          <v-btn
                            class="ml-2"
                            small
                            depressed
                            @click="closeOrder(order)"
                          >{{ $t('order.acceptedOrder.expanded.closeOrder') }}</v-btn>
                        </v-col>
                        -->
                      </v-row>
                    </v-card-text>
                  </div>
                </v-expand-transition>
              </v-card>
            </template>
          </div>
        </v-container>
      </v-card>
    </v-hover>
  </div>
</template>

<script>
export default {
  name: 'AcceptedOrder',
  data: () => ({
    loading: true,
    expandShow: null,
    orders: []
  }),
  created () {
    if (this.$DevMode) {
      const orders = this.$DevData.order.acceptedOrders
      this.orders = orders
      if (orders.length != 0) this.expandShow = orders[0].order_id
      this.loading = false
      return
    }

    this.axios.post('/api/order/queryAcceptedOrder').then((Response) => {
      if (Response.data) {
        if (Response.data.length != 0) {
          this.orders = Response.data
          this.expandShow = this.orders[0].order_id
        } else {
          this.orders = []
        }
        this.loading = false
      }
    })
  },
  methods: {
    expandPanel (orderId) {
      if (this.expandShow === orderId) {
        this.expandShow = null
      } else {
        this.expandShow = orderId
      }
    },
    finishOrder (item) {
      // 设置订单状态为已完成
      if (confirm(this.$i18n.t('order.orderList.actions.finishOrderConfirm'))) {
        if (this.$DevMode) {
          this.$Bus.$emit(
            'setSnackbar',
            this.$i18n.t('order.orderList.actions.finishOrderSucceed')
          )
          for (let i = 0; i < this.orders.length; i++) {
            if (this.orders[i] == item) {
              this.orders.splice(i, 1)
              break
            }
          }
          if (this.orders.length != 0) this.expandShow = this.orders[0].order_id
          return
        }

        this.axios.post('/api/order/finishOrder', {
          order_id: item.order_id
        }).then((Response) => {
          if (Response.data) {
            this.$Bus.$emit(
              'setSnackbar',
              this.$i18n.t('order.orderList.actions.finishOrderSucceed')
            )
          } else {
            this.$Bus.$emit(
              'setSnackbar',
              this.$i18n.t('order.orderList.actions.finishOrderFailed')
            )
          }
          this.refreshRouter()
        })
      }
    },
    /*
    cancelOrder (order) { // 取消订单
      if (confirm(this.$i18n.t('order.orderList.actions.cancelOrderConfirm'))) {
        this.axios.post('/api/order/cancelOrder', {
          order_id: order.order_id
        }).then((Response) => {
          if (Response.data) {
            this.$Bus.$emit('setSnackbar', this.$i18n.t('order.orderList.actions.cancelOrderSucceed'))
            this.refreshRouter()
          } else {
            this.$Bus.$emit('setSnackbar', this.$i18n.t('order.orderList.actions.cancelOrderFailed'))
            this.refreshRouter()
          }
        })
      }
    },
    closeOrder (order) { // 关闭订单
      if (confirm(this.$i18n.t('order.orderList.actions.closeOrderConfirm'))) {
        this.axios.post('/api/order/closeOrder', {
          order_id: order.order_id
        }).then((Response) => {
          if (Response.data) {
            this.$Bus.$emit('setSnackbar', this.$i18n.t('order.orderList.actions.closeOrderSucceed'))
            this.refreshRouter()
          } else {
            this.$Bus.$emit('setSnackbar', this.$i18n.t('order.orderList.actions.closeOrderFailed'))
            this.refreshRouter()
          }
        })
      }
    },
    */
    telephoneCall (telephone) {
      window.location.href = 'tel:' + telephone
    },
    refreshRouter () {
      this.$router.push({ path: '/_empty' })
      this.$router.back(-1)
    }
  }
}
</script>
