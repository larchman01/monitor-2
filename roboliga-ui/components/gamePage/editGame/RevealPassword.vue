<template>
    <v-form ref="inputForm" v-model="valid">
        <v-row justify="center" align="center">
            <v-col cols="12" md="8">
                <v-text-field
                        :type="computedType"
                        v-model="password"
                        label="Game Password"
                        variant="underlined"
                        color="primary"
                        :readonly="true"
                        @mouseover="hover = true"
                        @mouseleave="hover = false"
                        @click="copyToClipboard"
                ></v-text-field>
            </v-col>
            <v-col cols="12" md="4">
                <v-btn color="primary" variant="text" @click="togglePasswordVisibility">
                    {{ isPasswordRevealed ? 'Hide Password' : 'Reveal Password' }}
                </v-btn>
            </v-col>
        </v-row>
    </v-form>
</template>

<script setup>
import { useAuthStore } from "~/stores/auth";
import { ref, computed } from 'vue';

const auth = useAuthStore();
const { gameId } = useRoute().params;
const emit = defineEmits(['snackBarEmit']);

const valid = ref(false);
const inputForm = ref(null);
const password = ref("");
const isPasswordRevealed = ref(false);
const hover = ref(false);

const computedType = computed(() => {
    if (hover.value) {
        return 'text';
    }
    return isPasswordRevealed.value ? 'password' : 'text';
});

async function togglePasswordVisibility() {
    if (!isPasswordRevealed.value) {
        const pass = auth.getPass(gameId);
        if (pass === 'no-pass') {
            emit('snackBarEmit', "You aren't authorized to view the password", "");
            return;
        }
        password.value = pass;
    } else {
        password.value = "";
    }
    isPasswordRevealed.value = !isPasswordRevealed.value;
}

function copyToClipboard() {
    if (password.value === "") {
        emit('snackBarEmit', "Password is empty, nothing to copy", "");
        return;
    }
    navigator.clipboard.writeText(password.value).then(() => {
        emit('snackBarEmit', "Password copied to clipboard", "");
    }).catch(err => {
        emit('snackBarEmit', "Failed to copy password", "");
    });
}
</script>

<style scoped>
</style>