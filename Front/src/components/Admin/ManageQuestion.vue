<script setup lang="ts">
import { ref, onMounted } from "vue";
import { useRouter } from 'vue-router';

const router = useRouter();
const questions = ref([]);
const newQuestionText = ref("");
const errorMessage = ref("");
const wait = ref(true);

async function fetchQuestions() {
    const token = localStorage.getItem('auth_token');
    if (!token) {
        errorMessage.value = "Vous n'êtes pas connecté";
        wait.value = false;
        return;
    }

    try {
        const response = await fetch("http://83.195.188.17:3000/question", {
            method: "GET",
            mode: "cors",
            credentials: "include",
            headers: {
                "Content-Type": "application/json",
                "Authorization": `Bearer ${token}`,
            },
        });
        if (response.ok) {
            questions.value = await response.json();
            wait.value = false;
        } else {
            errorMessage.value = "Erreur lors de la récupération des questions";
            wait.value = false;
        }
    } catch (error) {
        console.error("Erreur lors de la récupération des questions:", error);
        errorMessage.value = "Erreur lors de la récupération des questions";
        wait.value = false;
    }
}

</script>

<template>
    <div class="manage-question">
        <h1>Gestion des questions</h1>
        <div v-if="errorMessage" class="error">{{ errorMessage }}</div>
        <div v-if="wait" class="loading">Chargement...</div>
        <div v-else>
            <h2>Liste des questions</h2>
            <ul>
                <li v-for="question in questions" :key="question.id">
                    {{ question.text }}
                    <button @click="deleteQuestion(question.id)">Supprimer</button>
                </li>
            </ul>
            <h2>Ajouter une question</h2>
            <input type="text" v-model="newQuestionText" placeholder="Texte de la question" />
            <button @click="addQuestion">Ajouter</button>
        </div>
    </div>
</template>
