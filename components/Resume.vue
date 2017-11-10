<template>
    <div class="card">
        <div class="card-content">
            <table class="table is-striped is-fullwidth" v-if="!emptyCart">
                <thead>
                <tr>
                    <td>Name</td>
                    <td>Price</td>
                    <td>Quantity</td>
                </tr>
                </thead>
                <tbody>
                <tr v-for="p in products">
                    <td>{{ p.name }}</td>
                    <td>{{ p.price | currency }}</td>
                    <td>{{ p.quantity }}</td>
                </tr>
                <tr>
                    <td>
                        <b>Total:</b>
                    </td>
                    <td></td>
                    <td>
                        <b>{{ total | currency }}</b>
                    </td>
                </tr>
                </tbody>
            </table>

            <section class="section" v-if="emptyCart">
                <div class="content has-text-grey has-text-centered">
                    <p>
                        <span class="icon is-large">
                          <i class="mdi">sentiment_very_dissatisfied</i>
                        </span>
                    </p>
                    <p>Nothing here.</p>
                </div>
            </section>

            <button
                    v-if="!emptyCart"
                    class="button is-primary"
                    @click="initPayment"
            >
                Pay now
            </button>
        </div>
    </div>
</template>

<script>
  import axios from 'axios'

  export default {
    name: 'resume',
    props: {
      products: {
        type: Array,
        default: () => [],
        required: true
      }
    },
    data () {
      return {
        stripeHandler: null
      }
    },
    created () {
      const script = document.createElement('script')
      script.src = 'https://checkout.stripe.com/checkout.js'
      script.onload = () => this.initStripe()
      document.body.appendChild(script)
    },
    computed: {
      emptyCart () {
        return this.products.length < 1
      },
      total () {
        return this.products.reduce((total, product) => total + product.price * product.quantity, 0)
      }
    },
    methods: {
      initStripe () {
        this.stripeHandler = StripeCheckout.configure({
          key: 'YOUR API KEY',
          image: 'https://upload.wikimedia.org/wikipedia/commons/thumb/9/99/Unofficial_JavaScript_logo_2.svg/1200px-Unofficial_JavaScript_logo_2.svg.png',
          locale: 'es',
          token: (token) => {
            const payload = {
              token: token.id,
              products: this.products,
              email: token.email
            }
            const loadingComponent = this.$loading.open()
            axios.post('your api url', payload)
              .then(response => {
                loadingComponent.close()
              })
              .catch(error => {
                console.log(error.response)
              })
          }
        })
      },
      initPayment () {
        this.stripeHandler.open({
          name: 'Your store name',
          description: '',
          currency: 'eur',
          zipCode: true,
          billingAddress: true,
          amount: parseFloat(this.total) * 100
        })
      }
    },
    filters: {
      currency (val) {
        return `${val.toFixed(2)} €`
      }
    }
  }
</script>
