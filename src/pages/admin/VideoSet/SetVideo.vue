<template>
    <v-container :fluid="true" dir="rtl">
        <v-form>
            <v-sheet color="#f4f4f4">
                <v-row>
                    <v-col :md="3">
                        <v-select label="آزمون" :items="quizList.list" item-text="title" item-value="id" v-model="selectedQuizzes" multiple dense :disabled="editMode" outlined />
                    </v-col>
                    <v-col :md="2">
                        <v-btn @click="selectAllExams" color="secondary">انتخاب همه آزمون ها</v-btn>
                    </v-col>
                    <v-col :md="2">
                        <v-select label="درس" :items="subCategoriesList.list" item-text="display_title" item-value="id" v-model="currentQuestion.sub_category_id" dense :disabled="editMode" outlined />
                    </v-col>
                    <v-col :md="5">
                        <v-text-field label="لینک ست" outlined dense v-model="setLink" />
                    </v-col>
                    <v-col :md="12" type="submit" @click="submit">
                        <v-btn color="primary" block height="750" outlined>ثبت</v-btn>
                    </v-col>
                </v-row>
            </v-sheet>
        </v-form>
    </v-container>
</template>

<script>
    import {Question} from '@/models/Question'
    import {ExamList} from '@/models/Exam'
    import {QuestSubcategoryList} from '@/models/QuestSubcategory'
    import axios from 'axios'
    import API_ADDRESS from "@/api/Addresses";

    export default {
        name: 'setVideo',
        data: () => {
            return {
                setLink: null,
                quizList: new ExamList(),
                subCategoriesList: new QuestSubcategoryList(),
                selectedQuizzes: [],
                exams: [],
            }
        },
        methods: {
            getExamById(quizId) {
                return this.exams.find((quiz) => quiz.id === quizId)
            },
            getQuizById(quizId) {
                return this.quizList.list.find((quiz) => quiz.id === quizId)
            },
            selectAllExams() {
                if (this.selectedQuizzes.length !== this.quizList.list.length) {
                    this.selectedQuizzes = []
                    this.quizList.list.forEach((item) => {
                        this.selectedQuizzes.push(item.id)
                    })
                }
            },
            submit () {
                const that = this
                const exams = []
                this.selectedQuizzes.forEach((item) => {
                    exams.push({exam_id: item})
                })
                axios.post(API_ADDRESS.question.attachSubCategoryToQuestion, {
                    sub_category_id: that.currentQuestion.sub_category_id,
                    video: that.setLink,
                    exams,
                }).then((response) => {
                    console.log(response)
                })
                // $.ajax({
                //         type: 'POST',
                //         url: 'http://localhost/3a/api/exam-question/attach/sub-category',
                //         data: {
                //             sub_category_id: that.currentQuestion.sub_category_id,
                //             video: that.setLink,
                //             exams
                //         },
                //         success: function (data) {
                //             console.log(data)
                //         },
                //     }
                // );
            }
        },
        mounted() {

        },
        created() {
            this.editMode = this.$route.name === 'quest.edit'
            new ExamList().fetch().then((response) => {
                this.quizList = new ExamList(response.data.data)
                if (!this.editMode) {
                    this.selectedQuizzes.push(this.quizList.list[0].id)
                }
            })
            this.subCategoriesList.fetch().then((response) => {
                this.subCategoriesList = new QuestSubcategoryList(response.data)
            })
            if (this.editMode) {
                this.currentQuestion.show(null, API_ADDRESS.question.getCurrentQuestion(this.$route.params.id))
                    .then((response) => {
                        this.currentQuestion = new Question(response.data.data)
                        this.trueChoiceIndex = this.currentQuestion.choices.list.findIndex((item) => item.answer )
                        this.updateRendered()
                    })
            } else {
                this.currentQuestion = new Question(this.questionData)
            }
        },
        watch: {
            'selectedQuizzes': function () {
                this.exams = []
                this.selectedQuizzes.forEach((item) => { this.exams.push({ id: item, order: null }) })
            }
        }
    }
</script>

<style>
    .renderedPanel {
        direction: rtl;
    }
    .renderedPanel .katex {
        direction: ltr;
    }
    .renderedPanel strong>strong>s {
        text-decoration: underline;
        text-underline-position: under;
    }

    .ML__virtual-keyboard-toggle {
        color: gray !important;
    }

    #mathfield, .latexData {
        font-size: 32px;
        margin: 3em;
        padding: 8px;
        border-radius: 8px;
        border: 1px solid rgba(0, 0, 0, .3);
        box-shadow: 0 0 8px rgba(0, 0, 0, .2)
    }
</style>

<style scoped>

</style>
