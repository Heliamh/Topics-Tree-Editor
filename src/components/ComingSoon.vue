<template>
    <v-overlay :absolute="true"
            :opacity="0"
               v-if="false"
    >
        <v-btn color="orange lighten-2"
                @click="overlay = false"
        >
            به زودی
        </v-btn>
    </v-overlay>
</template>

<script>
    export default {
        name: 'ComingSoon'
    }
</script>

<style scoped>

</style>
