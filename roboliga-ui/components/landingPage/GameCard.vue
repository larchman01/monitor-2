<template>
    <v-card :to="`/game/${gameId}`" color="secondary" class="myFont">
        <v-card-title class="pa-0">
            <v-toolbar color="secondary" density="compact" class="ma-0">
                <v-toolbar-title class="text-h6 ma-0">
                    {{ gameId }}
                </v-toolbar-title>
                <v-btn style="position: absolute; right: 0;" icon="mdi-delete-forever-outline" variant="text"
                       density="compact" color="grey-darken-2" @click.prevent="deleteCard"></v-btn>
            </v-toolbar>
        </v-card-title>
        <v-card-text v-if="gameState && gameState.teams">
            <div class="text-blue-darken-1">{{ gameState.teams[teamBlueId]?.name }}</div>
            <div class="text-red-darken-1">{{ gameState.teams[teamRedId]?.name }}</div>
        </v-card-text>
    </v-card>

    <v-snackbar
            v-model="snackbar"
            timeout="2500"
            :color="snackbarColor">
        {{ snackbarText }}
    </v-snackbar>
</template>

<script setup>
import {useFetch, useRuntimeConfig} from "#app";
import {useAuthStore} from "~/stores/auth";

const {$promptPassword} = useNuxtApp()

const baseApiUrl = useRuntimeConfig().public.baseApiUrl
const auth = useAuthStore()
const props = defineProps(['gameId'])
const emit = defineEmits(['gameDeleted'])

const snackbar = ref(false)
const snackbarText = ref("")
const snackbarColor = ref("")

let gameState = ref(null)
let teamBlueId = ref(null)
let teamRedId = ref(null)

const fetchGameState = async () => {
    const {data, error} = await useFetch(baseApiUrl + `/game/${props.gameId}`, {
        method: 'GET',
        pick: ["teams"]
    })

    if (error.value) {
        snackbar.value = true
        snackbarText.value = "Failed to fetch game state"
        snackbarColor.value = "error"
        return
    }

    gameState.value = data.value

    if (gameState.value && gameState.value.teams) {
        const teams = gameState.value.teams
        teamBlueId.value = Object.values(teams).find(t => t.color === 'blue')?.id
        teamRedId.value = Object.values(teams).find(t => t.color === 'red')?.id
    }
}

const deleteCard = async () => {
    const {data, error} = await useFetch(baseApiUrl + `/game/`, {
        method: 'delete',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': 'Basic ' + btoa(`${props.gameId}:${auth.getPass(props.gameId)}`)
        },
    })

    if (error.value) {
        const errorMessage = error.value.message || error.value.toString()
        const statusMatch = errorMessage.match(/: (\d{3}) /)
        const status = statusMatch ? parseInt(statusMatch[1], 10) : null
        if (status === 401) {
            console.log("Calling $promptPassword")
            $promptPassword(props.gameId)
        } else {
            snackbar.value = true
            snackbarText.value = error.value
            snackbarColor.value = "error"
        }
        return
    }

    if (data.value) {
        snackbar.value = true
        snackbarText.value = "Game deleted successfully"
        snackbarColor.value = ""
        emit('gameDeleted')
    }
}

onMounted(() => {
    fetchGameState()
})
</script>

<style scoped>
</style>