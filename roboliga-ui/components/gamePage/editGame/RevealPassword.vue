<template>
    <v-form ref="inputForm" v-model="valid">
        <v-row justify="center" align="center">
            <v-col cols="12" md="8">
                <v-text-field
                        :type="hover ? 'text' : 'password'"
                        v-model="password"
                        label="Game Password"
                        variant="underlined"
                        color="primary"
                        :readonly="true"
                        @mouseover="hover = true"
                        @mouseleave="hover = false"
                ></v-text-field>
            </v-col>
            <v-col cols="12" md="4">
                <v-btn color="primary" variant="text" @click="revealPassword">
                    Reveal Password
                </v-btn>
            </v-col>
        </v-row>
    </v-form>
</template>

<script setup>
import { useAuthStore } from "~/stores/auth";

const auth = useAuthStore();
const { gameId } = useRoute().params;
const emit = defineEmits(['snackBarEmit']);

const valid = ref(false);
const inputForm = ref(null);
const password = ref("");
const hover = ref(false);

async function revealPassword() {
    password.value = auth.getPass(gameId);
    emit('snackBarEmit', "Password revealed successfully", "");
}
</script>

<style scoped>
</style>