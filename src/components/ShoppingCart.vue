<template>
   <div>
      <router-view />
    </div>
<div class="product-grid">

  <div style="padding: 2rem;" class="container">
  <h2>Products</h2>
  <br>
  <div class="product-grid__cards">
    <div v-for="product in products" :key="product.id" class="product-grid__card" @click="addToCart(product)">
      <div class="product-grid__info">
        <h4 class="product-grid__name">{{product.name}}</h4>
    
      </div>
      <div class="product-grid__image-wrapper">
        <img :src="`${baseUrl}/${product.image}`" alt="product image" class="product-grid__image">
        <div class="product-grid__add-to-cart">
        <button class="btn btn-success">Add ${{product.price}}.0</button> 
        </div>
      </div>
    
    </div>
  </div>

</div>



<link href="https://polskoydm.pythonanywhere.com/static/styles/checkout.css" rel="stylesheet">
<div class="container mt-5 p-3 rounded cart">
  <div class="row no-gutters">
    <div class="col-md-8">
      <div class="product-details mr-2">
        <div class="d-flex flex-row align-items-center"><i style="margin-right: 1rem;" class="fa fa-shopping-cart"></i><span class="ml-2"> Shopping Cart</span></div>
        <hr>
      
        <div class="d-flex justify-content-between">
          <div class="d-flex flex-row align-items-center"><span class="text-black-50">Total: </span>
            <div class="price ml-2"><span class="mr-1"> ${{cartTotal}}</span></div>
          </div>
        </div>

        <div class="cart-items mt-3">
          <div v-for="(item, index) in cartItems" :key="index" class="item d-flex justify-content-between align-items-center p-2 rounded">
            <div class="d-flex flex-row">
              <img class="rounded" :src="'https://polskoydm.pythonanywhere.com/static/uploads/' + item.product.image" width="40">
              <div class="ml-2">
                <span class="font-weight-bold d-block">{{ item.product.name }}</span>
                <span class="spec">{{ item.quantity }}x</span>
              </div>
            </div>
            <div class="d-flex flex-row align-items-center">
              <span class="d-block ml-3 font-weight-bold">${{ item.product.price }}.0</span>
              <button class="btn btn-danger ml-3" style="margin-left: 2rem;" @click="removeFromCart(index)">
  <i class="fa fa-trash-o text-white"></i>
</button>

            </div>
          </div>
        </div>

      </div>
    </div>
  </div>
</div>

           
     
      <br>
      <button style="margin: 1rem;" class="btn btn-primary" @click="checkout">Checkout</button>
    </div>
  </template>
  
  <script>
  import axios from 'axios';

export default {
 
 
  data() {
    
    return {
      storeName: '',
      products: [],
      baseUrl: 'https://polskoydm.pythonanywhere.com/static/uploads',
      cartItems: [],
    }
  },

  created() {
    const storeId = window.location.href.split('/')[3];
    axios.get(`https://polskoydm.pythonanywhere.com/${storeId}/shop`)
      .then(response => {
        this.storeName = response.data[0].store
        this.products = response.data
      })
      .catch(error => {
        console.log(error)
      })
  },
  computed: {
    cartTotal() {
      return this.cartItems.reduce((total, item) => total + item.product.price * item.quantity, 0);
    },
  },
  methods: {
    addToCart(product) {
      const cartItem = this.cartItems.find(item => item.product.id === product.id);
      if (cartItem) {
        cartItem.quantity++;
      } else {
        this.cartItems.push({
          product,
          quantity: 1,
        });
      }
    },
    removeFromCart(index) {
      this.cartItems.splice(index, 1);
    },

    checkout() {
      if (this.cartItems.length === 0) {
        alert('Your cart is empty!');
        return;
      }
  
      const data = {
        items: this.cartItems.map(item => ({
          product_id: item.product.id,
          quantity: item.quantity,
        })),
        total: this.cartTotal,
      };
  
      axios.post('https://polskoydm.pythonanywhere.com/checkout', data)
        .then(response => {
          const orderID = response.data.order_id;
          this.cartItems = [];
  
          // Redirect to the payment page with the order ID
          this.$router.push({ name: 'Payment', params: { orderID } });
        })
        .catch(error => {
          console.log(error);
        });
    },
  },
};


  </script>


<style>
.product-grid__card {
  position: relative;
}

.product-grid__add-to-cart {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: rgba(0, 0, 0, 0.5);
  
}




.product-grid__add-to-cart button {
  margin: 10px;
}


.product-grid {
  max-width: 1200px;
  margin: 0 auto;
}

.product-grid__cards {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));

  grid-gap: 20px;

}

.product-grid__card {
  border: 1px solid #ddd;
  border-radius: 5px;
  overflow: hidden;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
}

.product-grid__image {
  width: 16rem;
  height: 16rem;
  
  object-fit: cover;
  display: block;
  margin: 0 auto;
}

.product-grid__info {
  flex: 1;
  padding: 10px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.product-grid__name {
  margin: 0;
}

.product-grid__description {
  margin: 10px 0;
}

.product-grid__price {
  margin: 0;
}

.product-grid__actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 10px;
}


</style>