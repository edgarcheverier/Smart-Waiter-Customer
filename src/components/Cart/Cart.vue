<template>
  <div>
    <CartNav />
    <div 
      v-if="!cartList[0]" 
      class="NoOrdersContainer">
      <h2 class="NoOrdersTitle">You don't have any products yet</h2>
      <button 
        class="NoOrdersButton" 
        @click="goBack">Go Back </button>
    </div>
    <div 
      v-if="cartList[0]" 
      class="cartContainer">
      <div 
        v-for="(item, index) in cartList" 
        v-if="item.count !== 0"
        :key="index" 
        class="listContainer">
        <div class="itemsContainers">
          <p class="itemName">{{ item.name }}</p>
        </div>
        <div class="priceCount">
          <button 
            class="removeButton" 
            @click="() => removeElements(item)" >
            <ion-icon 
              id="checkout" 
              name="remove-circle"/>
          </button>
          <p class="itemName">{{ item.count }}</p>
          <button 
            class="removeButton" 
            @click="() => addElements(item)" > 
            <ion-icon 
              id="checkout" 
              name="add-circle"/>
          </button>
        </div>
        <div class="priceContainer">
          <p class="itemName">{{ item.price }} €</p>
        </div>
        <hr>
      </div>
      <div 
        v-if="userKey !== 'Code'" 
        class="buttonContainer">
        <button 
          class="orderButton" 
          @click="SubmitOrder">Order Now</button>
      </div>
      <div 
        v-if="userKey == 'Code'" 
        class="buttonContainer">
        <button 
          class="orderButton" 
          @click="goToConnect">Please Connect to Order</button>
      </div>
    </div>
  </div>
</template>

<script>
import ItemCart from './ItemCart.vue';
import CartNav from './CartNav.vue';
export default {
  name: 'Cart',
  components: {
    CartNav,
    ItemCart,
  },
  data: function() {
    return {
      total: this.$store.state.amount.total,
      userKey: this.$store.state.restaurantKey,
      connectionId: this.$store.state.connectionId,
      //cartList: this.$store.state.cartList,
    };
  },
  computed: {
    cartList() {
      return this.$store.state.cartList;
    },
  },
  beforeCreate() {
    if (!this.$store.state.customer.id) {
      this.$router.push('/');
    }
  },
  mounted() {
    this.total = this.$store.state.amount.total;
    console.log('carlist:', this.$store.state.cartList);
    let map = this.$store.state.cartList.reduce(
      (acc, cur) => {
        const repeated = acc.find(el => el.id === cur.id);
        if (repeated !== undefined) {
          repeated.count++;
        } else {
          acc.push(cur);
        }
        return acc;
      },
      []
    );

    console.log('map reducer:', map);
    this.$store.commit('cartList', map);
    //this.cartList = this.$store.state.cartList;
  },
  methods: {
    SubmitOrder() {
      let arrayOfId = [];
      this.$store.state.cartList.map(ele => {
        for (let i = 0; i < ele.count; i++) {
          arrayOfId.push(ele.id);
        }
      });
      this.$store.dispatch('submitOrder', {
        array: arrayOfId.join(','),
        id: this.connectionId,
        amount: this.$store.state.amount.total.toPrecision(
          4
        ),
      });
      this.$store.commit('Orders', {
        amount: this.$store.state.amount.total.toPrecision(
          4
        ),
        products: this.$store.state.cartList,
      });
      this.$store.commit('cartList', []);
      this.$store.commit('restartAmount');
      this.$swal('Your order is in process thank you!');
      this.$forceUpdate();
      this.$router.push('/Welcome');
    },
    goBack() {
      this.$router.push('/welcome');
    },
    removeElements(item) {
      if (item.count > 0) {
        this.$store.state.cartList.map(ele => {
          if (ele.id === item.id) {
            return {
              id: item.id,
              name: item.name,
              price: item.price,
              count: item.count--,
            };
          } else {
            return item;
          }
        });
        this.$store.commit('subtracAmount', item.price);
      }
      if (item.count == 0) {
        this.$store.state.shoppingList.map((ele, index) => {
          if (ele.id === item.id) {
            arr.splice(index, 1);
          }
        });
      }
      this.$forceUpdate();
    },

    addElements(item) {
      console.log(
        'from Cart Cartlist:',
        this.$store.state.cartList
      );
      this.$store.state.cartList.map(ele => {
        if (ele.id === item.id) {
          return {
            id: item.id,
            name: item.name,
            price: item.price,
            count: item.count++,
          };
        } else {
          return ele;
        }
      });
      // console.log('arr', arr);
      // this.$store.commit('cartList', arr);
      console.log(
        'from Cart Cartlist:',
        this.$store.state.cartList
      );
      this.$store.commit('updateAmount', item.price);
      this.$forceUpdate();
    },
    goToConnect() {
      this.$router.push('/Connection');
    },
  },
};
</script>

<style>
.removeButton {
  color: #28abfa;
}

.orderButton {
  border: 0.5px solid #28abfa;
  background-color: #28abfa;
  font-size: 15px;
  margin-top: 15px;
  margin-bottom: 15px;
  padding: 5px 30px;
  color: white;
}
.itemsContainers {
  display: flex;
  justify-content: center;
  margin-top: 35px;
  margin-left: 10px;
  margin-right: 10px;
}
.priceCount {
  display: flex;
  justify-content: space-around;
  margin-top: 25px;
  margin-left: 10px;
  margin-right: 10px;
}
.itemName {
  color: #064d78;
}
.priceContainer {
  display: flex;
  justify-content: center;
  margin-top: 15px;
  margin-bottom: 15px;
}
.buttonContainer {
  margin-top: 15px;
  margin-bottom: 15px;
  display: flex;
  justify-content: center;
}
.cartContainer {
  overflow: scroll;
  margin-top: 12px;
  height: 1000px;
}
</style>
