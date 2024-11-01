<template>
    <v-form ref="inputForm" v-model="valid">
        <v-row justify="center" align="center">
            <v-col cols="6" md="4">
                <v-text-field
                        v-model="blueTeamFuel"
                        label="Blue team fuel"
                        variant="underlined"
                        color="blue-lighten-1"
                        :rules="numberRules"
                ></v-text-field>
            </v-col>

            <v-col cols="6" md="4">
                <v-text-field
                        v-model="redTeamFuel"
                        label="Red team fuel"
                        variant="underlined"
                        color="red-lighten-1"
                        :rules="numberRules"
                ></v-text-field>
            </v-col>
            <v-col cols="12" md="4">
                <v-btn color="primary" variant="text" @click="validate">
                    update fuel
                </v-btn>
            </v-col>
        </v-row>
    </v-form>
</template>

<script setup>
import {useAuthStore} from "~/stores/auth";

const {$promptPassword} = useNuxtApp()
const baseApiUrl = useRuntimeConfig().public.baseApiUrl
const auth = useAuthStore()
const {gameId} = useRoute().params
const props = defineProps(['teamsId'])
const emit = defineEmits(['snackBarEmit'])

const valid = ref(false)
const inputForm = ref(null)

const blueTeamFuel = ref("25")
const redTeamFuel = ref("25")

const numberRules = [
    v => v.length > 0 || 'This field may not be empty',
    v => Number.isInteger(Number(v)) || 'The value must be an integer number',
    //v => v >= 0 || 'The value must be greater or equal to zero'
]

async function validate() {
    const {valid} = await inputForm.value.validate()
    if (valid) {
        const {data, error} = await useFetch(baseApiUrl + `/game/fuel`, {
            method: 'put',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': 'Basic ' + btoa(`${gameId}:${auth.getPass(gameId)}`)
            },
            body: {
                [props.teamsId[0]]: Number(blueTeamFuel.value),
                [props.teamsId[1]]: Number(redTeamFuel.value)
            },
        });
        if (error.value) {
            const errorMessage = error.value.message || error.value.toString()
            const statusMatch = errorMessage.match(/: (\d{3}) /)
            const status = statusMatch ? parseInt(statusMatch[1], 10) : null
            if (status === 401) {
                $promptPassword(gameId)
            } else
                emit('snackBarEmit', error.value, "error")
        } else if (data.value) {
            emit('snackBarEmit', "Fuel updated successfully", "")
        }
    }
}
</script>

<style scoped>
</style>