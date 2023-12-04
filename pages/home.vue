<template>
    <section class="section">
        <div class="container">
            <h1 class="title">Dashboard</h1>

            <p>Hi {{ user.name }}</p>

            <v-btn @click.prevent="logout">Logout</v-btn>
            <v-btn @click.prevent="handle">Handle</v-btn>
        </div>
    </section>
</template>
  
<script>
export default { 
    data() {
        return {
            user: this.$auth.user.data,
        }
    },
    methods: {

        handle(){
            console.log(this.$auth.user.data)
        },

        async logout() {
            try {
                await this.$auth.logout()
                this.$router.push('/login')
            }catch (error){
                console.log(error)
            }
        },
    },
}
</script>