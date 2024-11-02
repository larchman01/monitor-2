<template>
    <v-row justify="center" align="center">
        <v-col cols="12" md="4">
            <v-btn color="primary" variant="text" @click="promptPassword">
                Add to "Your games"
            </v-btn>
        </v-col>
    </v-row>
</template>

<script setup>
import { useNuxtApp } from '#app';
import { useRoute } from 'vue-router';

const { $promptPassword } = useNuxtApp();
const { gameId } = useRoute().params;

function promptPassword() {
    $promptPassword(gameId);
}
</script>

<style scoped>
</style>