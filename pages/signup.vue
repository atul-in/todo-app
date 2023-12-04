<template>
    <section class="section">
        <div class="container">
            <div class="col-3 mx-auto">
                <h2 class="text-center">Register</h2>
                <form method="post" @submit.prevent="register">
                    <v-text-field v-model="name" label="Name" />
                    <v-text-field v-model="email" label="Email" />
                    <v-text-field v-model="password" label="Password" />
                    <v-text-field v-model="password_confirmation" label="Confirm Password" />
                    <v-btn type="submit">Sign Up</v-btn>
                </form>
                <div style="margin-top: 20px">
                    Already got an account ? <nuxt-link to="/login">Login</nuxt-link>
                </div>
            </div>
        </div>
    </section>
</template>

<script>
export default {
    auth: false,
    data() {
        return {
            name: '',
            email: '',
            password: '',
            password_confirmation: '',
            error: null
        }
    },
    methods: {
        async register() {
            try {
                await this.$axios.get('/sanctum/csrf-cookie')
                const registerReponse = await this.$axios.post('/api/signup', {
                    name: this.name,
                    email: this.email,
                    password: this.password,
                    password_confirmation: this.password_confirmation
                })


                const loginResponse = await this.$auth.loginWith('laravelSanctum', {
                    data: {
                        email: this.email,
                        password: this.password
                    },
                })

                console.log(registerReponse, loginResponse)

                this.$router.push('/')
            } catch (e) {
                this.error = e.response.data.message
            }
        },
    },
}
</script>