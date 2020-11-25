<template>
    <div class="home">

        <b-container>

            <b-row>

                <b-col v-for="(product, productIndex) in products" :key="productIndex">

                    <h3>{{ product.title }}</h3>

                    <b-btn @click="onAddToCartClick(product)">Add To Cart</b-btn>

                </b-col>

            </b-row>

            <b-row>

                <b-col>

                    <b-btn @click="onCheckoutClick">Checkout</b-btn>

                </b-col>

            </b-row>

        </b-container>

    </div>
</template>

<script>
import axios from 'axios'

let stripe;

export default {

    name: 'Home',

    mounted() {
        this.loadStripe().then(() => {
            stripe = new Stripe(process.env.VUE_APP_STRIPE_PUBLIC_KEY);
        });
    },

    beforeDestroy() {
        this.unloadStripe();
    },

    data: function () {
        return {
            products: [
                {
                    title: 'Orange Shirt'
                },
                {
                    title: 'Blue Shirt'
                }
            ],
            lineItems: []
        }
    },

    methods: {

        loadStripe () {
            return this.$loadScript('https://js.stripe.com/v3');
        },

        unloadStripe () {
            return this.$unloadScript('https://js.stripe.com/v3');
        },

        onAddToCartClick (product) {
            this.lineItems.push({
                title: product.title,
                quantity: 1
            })
        },

        onCheckoutClick () {
            axios.post('http://stripe-test-project.megumidev/api/v1/stripe/create-checkout-session', {
                line_items: this.lineItems
            })
                .then((response) => {
                    return stripe.redirectToCheckout({ sessionId: response.data.data.id });
                })
                .then((result) => {
                    if (result.error) {
                        alert(result.error.message);
                    }
                })
                .catch((error) => {
                    console.error('Error:', error);
                });
        }

    }

}
</script>
