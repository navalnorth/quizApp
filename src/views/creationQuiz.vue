<template>
    <div class="w-full">
        <h2 class="text-4xl m-2 flex justify-center">Création d'un nouveau Quiz</h2>

        <form @submit.prevent="clique" class="flex flex-col justify-center items-center">
            <div>
                <input class="px-5 py-2.5 rounded-3xl ring-1 ring-cyan-700 m-2 w-96" v-model="monQuiz.nom_quiz"
                    placeholder="Nom du nouveau Quiz" />
            </div>
            <div>
                <input class="px-5 py-2.5 rounded-3xl ring-1 ring-cyan-700 m-2 w-96" placeholder="Nom de la question"
                    v-model="currentQuestion.nom_question" />
            </div>

            <div class="flex flex-col justify-center items-center my-4">

                <button @click.prevent="reponsePlus"
                    class="px-5 py-2.5 bg-cyan-950 text-white border-none rounded-3xl hover:bg-cyan-700">
                    Ajouter réponse
                </button>

                <div v-for="(reponse, index) in currentQuestion.reponses" :key="index"
                    class="flex items-center mt-4 flex-col">
                    <input class="px-5 py-2.5 rounded-3xl ring-1 ring-cyan-700 mb-2 w-96"
                        placeholder="Ecrivez votre réponse" v-model="currentQuestion.reponses[index]" />
                    <input type="radio" v-model="currentQuestion.reponse_correcte" :value="index"
                        :name="'correct' + QuestionIndex" />
                </div>
            </div>
            <div class="w-full flex justify-between px-10">
                <button class="px-5 py-2.5 bg-cyan-700 text-white border-none rounded-3xl hover:bg-cyan-950 w-32"
                    @click.prevent="precedent" :disabled="QuestionIndex === 0">
                    Précedent
                </button>

                <div class="text-red-600 text-center">
                    <p v-for="erreur in erreurs" :key="erreur">{{ erreur }}</p>
                </div>

                <button class="px-5 py-2.5 bg-cyan-700 text-white border-none rounded-3xl hover:bg-cyan-950 w-32"
                    @click.prevent="suivant">
                    Suivant
                </button>
            </div>



            <button class="px-5 py-2.5 bg-cyan-700 text-white border-none rounded-3xl hover:bg-cyan-950 w-32"
                type="submit">
                Créer
            </button>
        </form>


    </div>
</template>



<script setup lang="ts">
import { computed, ref } from 'vue'
import type { Quiz } from "../types/config";
import { useRouter } from 'vue-router';

const router = useRouter()
const erreurs = ref<string[]>([]);
const VITE_API_URL = process.env.VUE_APP_API_URL;

const monQuiz = ref<Quiz>({
    nom_quiz: '',
    isActive: false,
    questions: [
        {
            nom_question: '',
            reponses: ['', ''],
            reponse_correcte: 0,
        }
    ]
});



const QuestionIndex = ref(0);

const currentQuestion = computed(() => monQuiz.value.questions[QuestionIndex.value]);
const isLastQuestion = computed(() => QuestionIndex.value === monQuiz.value.questions.length - 1);



const suivant = () => {
    erreurs.value = []
    if (!validerQuestion()) return;

    if (isLastQuestion.value) {
        addQuestion();
    }
    QuestionIndex.value++;
}

const precedent = () => {
    if (QuestionIndex.value > 0) {
        QuestionIndex.value--;
    }
}



const addQuestion = () => {
    monQuiz.value.questions.push({
        nom_question: '',
        reponses: ['', ''],
        reponse_correcte: 0
    });
}



const reponsePlus = () => {
    currentQuestion.value.reponses.push('');
}


const validerQuestion = () => {
    const { nom_question, reponses, reponse_correcte } = currentQuestion.value

    if (!monQuiz.value.nom_quiz) {
        erreurs.value.push('Le nom du quiz est obligatoire.');
        return false;
    }

    if (!nom_question) {
        erreurs.value.push('Le nom de la question est obligatoire.');
        return false;
    }

    if (reponses.length < 2 || reponses.some(rep => !rep)) {
        erreurs.value.push('Chaque question doit avoir au moins deux réponses valides.');
        return false;
    }

    if (reponse_correcte === null || undefined) {
        erreurs.value.push('Une réponse correcte doit être sélectionnée.');
        return false;
    }

    return true;
}



const clique = async () => {
    erreurs.value = []

    if (!validerQuestion()) return

    await CreationQuiz();

    monQuiz.value.nom_quiz = '';
    monQuiz.value.questions = [{ nom_question: '', reponses: ['', ''], reponse_correcte: 0 }];
    QuestionIndex.value = 0;
    router.push('/profileAdmin')
}




const CreationQuiz = async () => {
    try {
        const formatedQuestion = monQuiz.value.questions.map((q) => ({
            nom_question: q.nom_question,
            reponses: q.reponses,
            reponse_correcte: q.reponse_correcte
        }))
        const response = await fetch(`${VITE_API_URL}/api/quiz/quiz/creationQuiz`, {
            method: 'POST',
            body: JSON.stringify({
                nom_quiz: monQuiz.value.nom_quiz,
                questions: formatedQuestion
            }),
            headers: {
                'Accept': 'application/json',
                'Content-Type': 'application/json',
            },
        });
        if (!response.ok) {
            console.error('Erreur lors de la création du quiz');
            return;
        }

        monQuiz.value.nom_quiz = '';
        monQuiz.value.questions = [{ nom_question: '', reponses: ['', ''], reponse_correcte: 0 }];
        QuestionIndex.value = 0;
    } catch (err) {
        console.error('Erreur durant la création du quiz:', err);
    }
}

</script>