<template>
  <div>
    <h2>Payment</h2>

    <table class="cart-table">
      <thead>
        <tr>
          <th>Item</th>
          <th>Quantity</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in cartItems" :key="item.id">
          <td>
            <div class="item-info">
              <img class="rounded" :src="`https://polskoydm.pythonanywhere.com/static/uploads/${item.image}`" width="40">
              <div class="item-details">
                <h4 class="item-name">{{ item.name }}</h4>
              </div>
            </div>
          </td>
          <td>{{ item.quantity }}</td>
          <td>${{ item.price }}</td>
        </tr>
      </tbody>
      <tfoot>
        <tr>
          <td colspan="2" class="total-label">Total:</td>
          <td class="total-price">${{ cartTotal }}</td>
        </tr>
      </tfoot>
    </table>

   
    <!-- Email Input -->
    <div class="form-group">
      <label for="email">Email:</label>
      <input type="email" id="email" v-model="email" required>
    </div>

    <!-- Name Input -->
    <div class="form-group">
      <label for="name">Name:</label>
      <input type="text" id="name" v-model="name" required>
    </div>

    <!-- Address Input -->
    <div class="form-group">
      <label for="address">Address:</label>
      <input type="text" id="address" v-model="address" required>
    </div>

    <div>
      <button @click="checkout">Proceed to Payment</button>
    </div>
  
   
  </div>

</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      orderID: null,
      cartItems: [],
      cartTotal: 0,
      email: '',
      name: '',
      address: '',
      paymentReceived: false,
    };
  },
  methods: {
    fetchCartItems(orderID) {
      axios.get(`https://polskoydm.pythonanywhere.com/payment/${orderID}`)
        .then(response => {
          this.cartItems = response.data.cart_items;
          this.cartTotal = response.data.total;
        })
        .catch(error => {
          console.log(error);
        });
    },
    checkout() {
      if (this.cartItems.length === 0) {
        alert('Your cart is empty!');
        return;
      }

      // Check if email, name, and address are provided
      if (!this.email || !this.name || !this.address) {
        alert('Please provide your email, name, and address.');
        return;
      }

      const data = {
        email: this.email,
        name: this.name,
        address: this.address,
      };

      // Call the server to create a new payment intent
      axios.post(`https://polskoydm.pythonanywhere.com/payment/${this.orderID}`, {
        email: this.email,
        name: this.name,
        address: this.address,
      })
        .then(response => {
          window.location.href = `https://polskoydm.pythonanywhere.com/create-checkout-session/${this.orderID}`;
        })
        .catch(error => {
          // Handle the error
        });
    },
  },
  created() {
    this.orderID = this.$route.params.orderID;
    this.fetchCartItems(this.orderID);
  },
};
</script>





<style>
.cart-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 1rem;
}

.cart-table th,
.cart-table td {
  padding: 0.5rem;
  border: 1px solid #ccc;
}

.cart-table thead th {
  background-color: #f0f0f0;
}

.product-image {
  width: 50px;
  height: 50px;
  object-fit: cover;
  margin-right: 1rem;
}

.total-label {
  font-weight: bold;
  text-align: right;
}

.total-price {
  font-weight: bold;
}

.form-group {
  margin-bottom: 1rem;
}

label {
  display: block;
  font-weight: bold;
  margin-bottom: 0.5rem;
}

input[type="email"],
input[type="text"] {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  padding: 0.5rem 1rem;
  background-color: #4caf50;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}
</style>
