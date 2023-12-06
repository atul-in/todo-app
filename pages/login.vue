<template>
    <section class="section">
        <div class="container">
            <div class="col-3 mx-auto">
                <h2 class="title text-center">Login</h2>
                <form method="post" @submit.prevent="login">
                    <v-text-field v-model="email" label="Email" type="email" required></v-text-field>
                    <v-text-field v-model="password" label="Password" type="password" required></v-text-field>
                    <v-btn type="submit" color="success" block :loading="loading">Login</v-btn>
                </form>
                <div style="margin-top: 20px" class="text-center">
                    New user? <nuxt-link to="/signup">Sign up</nuxt-link>
                </div>
            </div>
        </div>
    </section>
</template>

<script>

export default {
    data() {
        return {
            email: '',
            password: '',
            loading: false
        };
    },
    methods: {
        async login() {
            try {
                this.loading = true;
                const res = await this.$auth.loginWith('laravelSanctum', {
                    data: {
                        email: this.email,
                        password: this.password,
                    },
                });
                console.log(res);
                if (res.status === 201) {
                    this.$router.push('/');
                }
            }
            catch (error) {
                console.log(error);
                console.log(error.message);
            }
        },
    },
}
</script>

<style scoped>
.container{
    margin-top: 12rem;
}
</style>