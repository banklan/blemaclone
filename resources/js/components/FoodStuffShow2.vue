<template>
    <v-app light>
        <v-content>
            <v-container grid-list-sm>
                <v-btn fixed dark elevation="12" fab top right class="mt-5 mr-4" href="/my_cart"><v-icon>shopping_cart</v-icon></v-btn>
                <v-layout my-3>
                    <v-btn color="#ff3c38" dark raised rounded ripple @click.prevent="$router.go(-1)"><v-icon left>arrow_left</v-icon>Back</v-btn>
                </v-layout>
                <v-layout row wrap class="mb-4">
                    <v-flex xs12 sm8 class="mt-4">
                        <v-card raised elevation="12" light ripple hover min-height="380">
                            <v-container grid-list-sm>
                                <v-layout row wrap v-if="product" class="my-3">
                                    <v-flex xs12 sm6 text-xs-center >
                                        <v-img contain max-height="320" :src="`/images/products/organic/${product.picture}`" transition="scale-transition"></v-img>
                                    </v-flex>
                                    <v-flex xs12 sm6>
                                    <v-card-title>
                                        <div class="body-5 primary--text">{{ product.name }}  - &#8358;{{ product.price | price }}</div>
                                    </v-card-title>
                                        <v-card-text>
                                            <div class="body-3 grey--text mb-3">{{ product.description }}</div>
                                            <!-- <div v-if="product.service" class="mt-5"> -->
                                                <!-- <div class="subtitle">Extra Service</div> -->
                                                <!-- <v-switch v-model="serviceStatus" :label="`${product.service.name} ${product.name}`"></v-switch> -->
                                            <!-- </div> -->
                                        </v-card-text>
                                        <v-card-actions>
                                            <v-container fluid>
                                                <v-row align="center">
                                                    <v-col cols="6">
                                                        <v-select dense small :items="units" :label="product.unit" v-model="picked.units"></v-select>
                                                    </v-col>
                                                    <v-col cols="6">
                                                        <v-btn :disabled="loading" text light class="primary--text" @click.prevent="addToCart(product)">Add To Cart</v-btn>
                                                    </v-col>
                                                </v-row>
                                            </v-container>
                                        </v-card-actions>
                                    </v-flex>
                                </v-layout>
                                <v-progress-circular indeterminate color="coral" :width="7" :size="70" v-if="!product"></v-progress-circular>
                                <v-snackbar v-model="addSuccess" :timeout="4000" top color="#44a80f">
                                    You have added an item to your cart
                                    <v-btn color="white green--text" text @click="addSuccess = false">Close</v-btn>
                                </v-snackbar>
                            </v-container>
                        </v-card>
                    </v-flex>
                    <v-flex xs12 sm4 class="mt-4">
                        <v-card v-if="product && product.service.length > 0" raised elevation="12" light ripple min-height="200" width="100%" class="mb-4">
                            <v-card-title class="justify-center">
                                <div class="subtitle">Extra Services on {{ product.name }}</div>
                            </v-card-title>
                             <v-card v-for="serv in product.service" :key="serv.id" class="serv_card" height="100" width="90%">
                                 <v-card-title>
                                     <div class="body-2 primary--text">{{ serv.name }}</div>
                                 </v-card-title>
                                 <v-card-text>
                                    {{ serv.description }}
                                     <v-btn v-if="added" :disabled="loading" text dark class="primary--text float-right" @click.prevent="addService(serv)">Add Service</v-btn>
                                </v-card-text>
                             </v-card>
                        </v-card>
                        <v-card raised elevation="12" light ripple hover min-height="380" width="100%">
                            <v-card-title class="justify-center">
                                <div class="subtitle">Similar Products</div>
                            </v-card-title>
                            <v-progress-circular indeterminate color="coral" :width="2" :size="30" v-if="similar.length == 0"></v-progress-circular>
                            <div v-if="similar.length > 0" class="similar">
                                <v-card v-for="prod in similar" :key="prod.id" class="mb-3" height="80" width="90%" :to="{path: '/raw_foodstuff/' + prod.id + '/' + prod.slug}" hover>
                                    <v-container grid-list-sm>
                                        <v-layout row wrap>
                                            <v-flex xs3>
                                                <v-img contain max-height="60" :src="`/images/products/organic/${prod.picture}`" transition="scale-transition"></v-img>
                                            </v-flex>
                                            <v-flex xs9>
                                                <div class="details">
                                                    <div class="body-5 primary--text">{{ prod.name }} - &#8358;{{ product.price | price }}</div>
                                                    <div class="body-2 grey--text">{{ prod.description }}</div>
                                                </div>
                                            </v-flex>
                                        </v-layout>
                                    </v-container>
                                </v-card>
                            </div>
                        </v-card>
                    </v-flex>
                </v-layout>
                <v-snackbar v-model="serviceAddedSuccess" :timeout="4000" top color="#44a80f">
                    You have added a service to your cart
                    <v-btn color="white green--text" text @click.prevent="serviceAddedSuccess = false">Close</v-btn>
                </v-snackbar>
            </v-container>
        </v-content>
    </v-app>
</template>

<script>
export default {
    data() {
        return {
            id: null,
            product: null,
            units: [1,2,3,4,5],
            picked: {
                product: null,
                units: 1,
                cost: null,
            },
            addedService: {
                id: null,
                type: null,
                price: null,
                units: null,
                cost: null
            },
            loading: false,
            addSuccess: false,
            similar: [],
            serviceAddedSuccess: false,
            added: false
        }
    },
    watch: {
        '$route.params':{
            handler(newVal){
                let id = newVal
                this.getProduct(id);
            },
            immediate: true
        }
    },
    methods: {
        getProduct(){
            axios.get(`/get_product/${this.$route.params.id}`).then((res) => {
                this.product = res.data
                // console.log(res.data.name)

                //get similar products
                axios.get(`/get_similar_products/${res.data.category_id}/${this.$route.params.id}`).then((res) => {
                    this.similar = res.data
                    // console.log(res.data)
                })
            })
        },
        addToCart(product){
            // console.log(product.name)
            this.loading = true
            this.added = true
            this.picked.product = product
            this.picked.cost = parseFloat(this.product.price) * this.picked.units
            // console.log(this.picked)
            this.$store.commit('addItemsToCart', this.picked)
            this.loading = false
            this.addSuccess = true
        },
        addService(serv){
            this.loading = true
            this.addedService.type = serv.name
            this.addedService.id = serv.id
            this.addedService.price = serv.price
            this.addedService.units = this.picked.units
            this.addedService.cost = parseFloat(this.addedService.price) * this.picked.units
            this.$store.commit('addServicesToCart', this.addedService)
            this.serviceAddedSuccess = true
            this.loading = false
        },
    },
    mounted() {
        this.getProduct();
    },
}
</script>

<style lang="scss" scoped>
    @media screen and (min-width: 600px){
        .v-card{
            margin-left: 2.5rem !important;

        }
        .v-card__title{
            font-size: 1.1rem;
            font-weight: 200 !important;
        }
    }
    .v-application .primary--text{
        color: #ff3c38 !important;
    }
    .v-application .sec--text{
        color: #15C5C5 !important;
    }

    *{
        text-transform: none !important;
    }
    .similar a{
        margin: 1rem auto !important;
        text-decoration: none !important;

        .details{
            padding: 8px;
        }
    }
    .similar a:first-child{
        margin-top:-1rem;
    }
    .v-card .serv_card{
        margin: 1rem auto !important;
        line-height: 1.8 !important;
    }
</style>
