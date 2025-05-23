<script setup lang="ts">
import { ref, onMounted } from "vue";
import { useRouter } from 'vue-router';

import { fetchThemes } from '../Game/themes.ts';

const router = useRouter();
const questions = ref([]);
const themes = ref<string[]>([]); // Liste des thèmes
const selectedTheme = ref<string>("");

// Variables pour l'ajout de question
const newQuestionText = ref("");
const newQuestionAnswer = ref("");
const newQuestionType = ref("text"); // Défaut sur 'text'
const newQuestionSubtheme = ref("");
const errorMessage = ref("");
const wait = ref(true);

// Variables pour la modification
const editingQuestion = ref(null);
const editQuestionText = ref("");
const editQuestionAnswer = ref("");
const editQuestionTheme = ref("");
const editQuestionSubtheme = ref("");
const editQuestionType = ref("");

async function fetchQuestions(page = 1) {
    const token = localStorage.getItem('auth_token');
    if (!token) {
        errorMessage.value = "Vous n'êtes pas connecté";
        wait.value = false;
        return;
    }
    console.log(page)
    if (page < 1) {
        page = 1;
    }

    const url = new URL("http://83.195.188.17:3000/question");
    url.searchParams.append("page", page.toString());
    // url.searchParams.append("theme", selectedTheme.value || "Général");

    fetch(url.toString(), {
        method: "GET",
        mode: "cors",
        credentials: "include",
        headers: {
            "Content-Type": "application/json",
            "Authorization": `Bearer ${token}`,
        },
    })
    .then(async (response) => {
        questions.value = await response.json();
        console.log(questions.value);
        
        wait.value = false;
    })
    .catch((error) => {
        console.error("Erreur lors de la récupération des questions:", error);
        errorMessage.value = "Erreur lors de la récupération des questions";
        wait.value = false;
    });
}

async function addQuestion() {
    const token = localStorage.getItem('auth_token');
    if (!token || !newQuestionText.value || !newQuestionAnswer.value) {
        errorMessage.value = "Veuillez remplir tous les champs requis";
        return;
    }

    try {
        const response = await fetch("http://83.195.188.17:3000/question", {
            method: "POST",
            mode: "cors",
            credentials: "include",
            headers: {
                "Content-Type": "application/json",
                "Authorization": `Bearer ${token}`,
            },
            body: JSON.stringify({
                question: newQuestionText.value,
                answer: newQuestionAnswer.value,
                theme: selectedTheme.value || "Général",
                subtheme: newQuestionSubtheme.value || " ",
                type: newQuestionType.value
            }),
        });
        if (response.ok) {
            const newQuestion = await response.json();
            fetchQuestions(); // Rafraîchir la liste complète
            
            // Réinitialiser les champs
            newQuestionText.value = "";
            newQuestionAnswer.value = "";
            newQuestionSubtheme.value = "";
            selectedTheme.value = "";
            
            errorMessage.value = "Question ajoutée avec succès";
            setTimeout(() => {
                errorMessage.value = "";
            }, 2000);
        } else {
            errorMessage.value = "Erreur lors de l'ajout de la question";
        }
    } catch (error) {
        console.error("Erreur lors de l'ajout de la question:", error);
        errorMessage.value = "Erreur lors de l'ajout de la question";
    }
}

async function deleteQuestion(id) {
    const token = localStorage.getItem('auth_token');
    if (!token) {
        errorMessage.value = "Vous n'êtes pas connecté";
        return;
    }

    if (!confirm("Êtes-vous sûr de vouloir supprimer cette question ?")) {
        return;
    }

    try {
        const response = await fetch(`http://83.195.188.17:3000/question/${id}`, {
            method: "DELETE",
            mode: "cors",
            credentials: "include",
            headers: {
                "Content-Type": "application/json",
                "Authorization": `Bearer ${token}`,
            },
        });
        if (response.ok) {
            fetchQuestions(); // Rafraîchir la liste complète
            errorMessage.value = "Question supprimée avec succès";
            setTimeout(() => {
                errorMessage.value = "";
            }, 2000);
        } else {
            errorMessage.value = "Erreur lors de la suppression de la question";
        }
    } catch (error) {
        console.error("Erreur lors de la suppression de la question:", error);
        errorMessage.value = "Erreur lors de la suppression de la question";
    }
}

function startEdit(question) {
    editingQuestion.value = question.id;
    editQuestionText.value = question.question;
    editQuestionAnswer.value = question.answer;
    editQuestionTheme.value = question.theme;
    editQuestionSubtheme.value = question.subtheme;
    editQuestionType.value = question.type || "text";
}

async function saveEdit(id) {
    const token = localStorage.getItem('auth_token');
    if (!token) {
        errorMessage.value = "Vous n'êtes pas connecté";
        return;
    }

    try {
        const response = await fetch(`http://83.195.188.17:3000/question/${id}`, {
            method: "PUT",
            mode: "cors",
            credentials: "include",
            headers: {
                "Content-Type": "application/json",
                "Authorization": `Bearer ${token}`,
            },
            body: JSON.stringify({
                question: editQuestionText.value,
                answer: editQuestionAnswer.value,
                theme: editQuestionTheme.value,
                subtheme: editQuestionSubtheme.value,
                type: editQuestionType.value
            }),
        });
        
        if (response.ok) {
            editingQuestion.value = null;
            fetchQuestions(); // Rafraîchir la liste
            errorMessage.value = "Question modifiée avec succès";
            setTimeout(() => {
                errorMessage.value = "";
            }, 2000);
        } else {
            errorMessage.value = "Erreur lors de la modification de la question";
        }
    } catch (error) {
        console.error("Erreur lors de la modification de la question:", error);
        errorMessage.value = "Erreur lors de la modification de la question";
    }
}

function cancelEdit() {
    editingQuestion.value = null;
}

onMounted(() => {
    wait.value = true;
    fetchQuestions();

    fetchThemes()
    .then((data) => {
        themes.value = data; // Store the themes in the ref
    })
    .catch((error) => {
        console.error("Error fetching themes:", error);
    });
});
</script>

<template>
    <div class="manage-question">
        <h1>Gestion des questions</h1>
        <div v-if="errorMessage" class="error-message">{{ errorMessage }}</div>
        <div v-if="wait" class="loading">Chargement...</div>
        <div v-else>
            <h2>Liste des questions</h2>
            <div class="table-container">
                <table class="questions-table">
                    <thead>
                        <tr>
                            <th>Question</th>
                            <th>Réponse</th>
                            <th>Thème</th>
                            <th>Sous-thème</th>
                            <th>Type</th>
                            <th>Actions</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="question in questions.questions" :key="question.id">
                            <!-- Mode affichage normal -->
                            <template v-if="editingQuestion !== question.id">
                                <td>{{ question.question }}</td>
                                <td>{{ question.answer }}</td>
                                <td>{{ question.theme || 'Non défini' }}</td>
                                <td>{{ question.subtheme || 'Non défini' }}</td>
                                <td>{{ question.type || 'text' }}</td>
                                <td class="actions">
                                    <button class="edit-btn" @click="startEdit(question)">Modifier</button>
                                    <button class="delete-btn" @click="deleteQuestion(question.id)">Supprimer</button>
                                </td>
                            </template>
                            
                            <!-- Mode édition -->
                            <template v-else>
                                <td><input v-model="editQuestionText" placeholder="Question" /></td>
                                <td><input v-model="editQuestionAnswer" placeholder="Réponse" /></td>
                                <td>
                                    <select v-model="editQuestionTheme">
                                        <option v-for="theme in themes" :key="theme" :value="theme">
                                            {{ theme }}
                                        </option>
                                    </select>
                                </td>
                                <td><input v-model="editQuestionSubtheme" placeholder="Sous-thème" /></td>
                                <td>
                                    <select v-model="editQuestionType">
                                        <option value="text">Texte</option>
                                        <option value="choice">Choix multiple</option>
                                    </select>
                                </td>
                                <td class="actions">
                                    <button class="save-btn" @click="saveEdit(question.id)">Enregistrer</button>
                                    <button class="cancel-btn" @click="cancelEdit">Annuler</button>
                                </td>
                            </template>
                        </tr>
                    </tbody>
                </table>
                <div class="pagination">
                    <button 
                        :disabled="questions.currentPage <= 1" 
                        @click="fetchQuestions(questions.currentPage - 1)">
                        Précédent
                    </button>
                    <span>Page {{ questions.currentPage }} sur {{ questions.totalPages }}</span>
                    <button 
                        :disabled="questions.currentPage >= questions.totalPages" 
                        @click="fetchQuestions(questions.currentPage + 1)">
                        Suivant
                    </button>
                </div>
            </div>
            
            <h2>Ajouter une question</h2>
            <div class="add-question-form">
                <div class="form-group">
                    <label for="new-question">Question:</label>
                    <input id="new-question" type="text" v-model="newQuestionText" placeholder="Texte de la question" />
                </div>
                
                <div class="form-group">
                    <label for="new-answer">Réponse:</label>
                    <input id="new-answer" type="text" v-model="newQuestionAnswer" placeholder="Réponse correcte" />
                </div>
                
                <div class="form-group">
                    <label for="theme-select">Thème:</label>
                    <select id="theme-select" v-model="selectedTheme">
                        <option value="" disabled>-- Sélectionnez un thème --</option>
                        <option v-for="theme in themes" :key="theme" :value="theme">
                            {{ theme }}
                        </option>
                    </select>
                </div>
                
                <div class="form-group">
                    <label for="subtheme">Sous-thème:</label>
                    <input id="subtheme" type="text" v-model="newQuestionSubtheme" placeholder="Sous-thème (optionnel)" />
                </div>
                
                <div class="form-group">
                    <label for="question-type">Type:</label>
                    <select id="question-type" v-model="newQuestionType">
                        <option value="text">Texte</option>
                        <option value="choice">Choix multiple</option>
                    </select>
                </div>
                
                <button class="add-btn" @click="addQuestion">Ajouter cette question</button>
            </div>
        </div>
    </div>
</template>

<style scoped>
.manage-question {
    max-width: 1200px;
    margin: 0 auto;
    padding: 1rem;
}

.error-message {
    background-color: #ffdddd;
    color: #ff0000;
    padding: 10px;
    margin-bottom: 15px;
    border-radius: 5px;
}

.table-container {
    overflow-x: auto;
    margin-bottom: 20px;
}

.questions-table {
    width: 100%;
    border-collapse: collapse;
}

.questions-table th, .questions-table td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
}

.questions-table th {
    background-color: #0d0d0d;
    position: sticky;
    top: 0;
}

.questions-table tr:nth-child(even) {
    background-color: #070707;
}

.questions-table tr:hover {
    background-color: #0f0f0f;
}

.actions {
    display: flex;
    gap: 5px;
}

button {
    padding: 5px 10px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}

.edit-btn {
    background-color: #4caf50;
    color: white;
}

.delete-btn {
    background-color: #f44336;
    color: white;
}

.save-btn {
    background-color: #2196F3;
    color: white;
}

.cancel-btn {
    background-color: #607D8B;
    color: white;
}

.add-btn {
    background-color: #4caf50;
    color: white;
    padding: 8px 16px;
    font-size: 16px;
}

.add-question-form {
    background-color: #070707;
    padding: 15px;
    border-radius: 5px;
    margin-top: 10px;
}

.form-group {
    margin-bottom: 15px;
}

.form-group label {
    display: block;
    margin-bottom: 5px;
    font-weight: bold;
}

.form-group input, .form-group select {
    width: 100%;
    padding: 8px;
    border: 1px solid #333;
    border-radius: 4px;
}
</style>