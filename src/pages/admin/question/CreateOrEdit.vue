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
                    <v-col :md="2">
                        <div v-for="index in selectedQuizzes.length" :key="index">
                            <v-text-field
                                    v-if="typeof getQuizById(selectedQuizzes[index - 1]) !== 'undefined' && typeof getExamById(selectedQuizzes[index - 1]) !== 'undefined'"
                                    :label="'ترتیب در ' + getQuizById(selectedQuizzes[index - 1]).title"
                                    v-model="getExamById(selectedQuizzes[index - 1]).order"
                                    type="number"
                                    :disabled="editMode"
                                    outlined
                            />
                        </div>
                    </v-col>
                    <v-col :md="3" @click="submitQuestion" type="submit">
                        <v-btn color="primary" block>ثبت سوال</v-btn>
                    </v-col>
                    <v-col :md="12">
                        متن سوال:
                        <div class="renderedPanel" v-html="questRendered">
                        </div>
                    </v-col>
                </v-row>
            </v-sheet>
            <v-radio-group v-model="trueChoiceIndex">
                <v-row>
                    <v-col md="6" v-for="index in 4" :key="index" :style="{ 'display': 'flex' }">
    <!--                    <v-col class="pl-5" :md="5">-->
<!--                            <v-textarea v-if="typeof currentQuestion.choices.list[index - 1] !== 'undefined'"-->
<!--                                        dir="rtl"-->
<!--                                        clearable-->
<!--                                        clear-icon="mdi-close-circle"-->
<!--                                        auto-grow-->
<!--                                        :label="choiceNumber[index -1]"-->
<!--                                        v-model="currentQuestion.choices.list[index - 1].title"-->
<!--                                        @input="updateRendered"-->
<!--                            ></v-textarea>-->
    <!--                    </v-col>-->
    <!--                    <v-col :md="2">-->
    <!--                        <v-radio :value="index - 1" />-->
    <!--                        <markdown-btn :elem="currentQuestion.choices.list[index - 1]" :elem-key="'title'" :rendered-matrix-katex="renderedMatrixKatex" :url="url" @add="markdownBtnAddString" />-->

    <!--                    </v-col>-->
                        <v-radio :value="index - 1" />
                        {{ index }})
                        <div class="renderedPanel" v-html="choiceRendered[index - 1]">
                        </div>
                    </v-col>
                </v-row>
            </v-radio-group>
<!--            <v-row>-->
<!--                <v-col>-->
<!--                    <v-textarea v-model="currentQuestion.answer" outlined label="جواب تشریحی" />-->
<!--                </v-col>-->
<!--            </v-row>-->
        </v-form>
        <hr>
        <div class="d-flex justify-space-around mb-5 mt-5">
            <v-btn-toggle v-model="selectedField">
                <v-btn :value="0">
                    متن سوال
                </v-btn>

                <v-btn :value="1">
                    گزینه 1
                </v-btn>

                <v-btn :value="2">
                    گزینه 2
                </v-btn>

                <v-btn :value="3">
                    گزینه 3
                </v-btn>

                <v-btn :value="4">
                    گزینه 4
                </v-btn>
            </v-btn-toggle>
            <markdown-btn v-if="selectedField === 0" :elem="currentQuestion" :elem-key="'statement'" :rendered-matrix-katex="renderedMatrixKatex" :url="url" @add="markdownBtnAddString" />
            <markdown-btn v-else :elem="currentQuestion.choices.list[selectedField - 1]" :elem-key="'title'" :rendered-matrix-katex="renderedMatrixKatex" :url="url" @add="markdownBtnAddString" />
        </div>
        <v-textarea
            v-if="selectedField === 0"
            v-model="currentQuestion.statement"
            outlined
            dir="rtl"
            clearable
            clear-icon="mdi-close-circle"
            auto-grow
            :label="'متن'"
            @input="updateRendered"
        ></v-textarea>
        <v-textarea
                v-else
                v-model="currentQuestion.choices.list[selectedField - 1].title"
                outlined
                dir="rtl"
                clearable
                clear-icon="mdi-close-circle"
                auto-grow
                :label="'متن'"
                @input="updateRendered"
        ></v-textarea>
        <div id="mathfield" locale="fa">x=\frac{-b\pm \sqrt{b^2-4ac}}{2a}</div>
        <div class="latexData" v-html="latexData"></div>
        <v-text-field full-width label="url" v-model="url" dir="ltr"/>
        <v-sheet width="500px">
            <v-container :fluid="true">
                <v-row>
                    <v-col :md="2">
                        <v-text-field label="عرض" outlined dense v-model="matrixWidth" type="number" @input="initMatrix"/>
                    </v-col>
                    <v-col :md="2">
                        <v-text-field label="ارتفاع" outlined dense v-model="matrixHeight" type="number" @input="initMatrix"/>
                    </v-col>
                    <v-col :md="4">
                        <v-select outlined :items="matrixMenu" item-text="title" item-value="value" v-model="determinan" dense />
                    </v-col>
                </v-row>
                <div dir="ltr">
                    <v-row v-for="indexY in parseInt(matrixTempHeight)" :key="indexY">
                        <v-col v-for="indexX in parseInt(matrixTempWidth)" :key="indexX">
                            <v-text-field v-model="matrix[indexY - 1][indexX - 1]" outlined :label="'(' + indexX + ', ' + indexX + ')'" dense></v-text-field>
                        </v-col>
                    </v-row>
                </div>
            </v-container>
        </v-sheet>

        <!--        <div dir="rtl" v-katex:auto>-->
        <!--                        این یک فرمول ریاضی هست-->
        <!--            <span dir="ltr">\(x=\frac{-b\pm\sqrt[]{b^2-4aca}}{2a}\)</span>-->
        <!--                        ببینید جوابش چی میشه؟-->
        <!--        </div>-->
        <!--        <hr>-->
        <!--        <div v-katex:auto>-->
        <!--            \(x=\frac{-b\pm\sqrt[]{b^2-4aca}}{2a}\)-->
        <!--        </div>-->

    </v-container>
</template>

<script>
    import '@/assets/scss/markdownKatex.scss'
    import MathLive from 'mathlive'
    import 'mathlive/dist/mathlive-fonts.css'
    import 'mathlive/dist/mathlive-static.css'
    import { Question } from '@/models/Question'
    import {ExamList} from '@/models/Exam'
    import {QuestSubcategoryList} from '@/models/QuestSubcategory';
    import Vue from 'vue'
    import MarkdownBtn from '@/components/QuizEditor/MarkdownBtn';
    import API_ADDRESS from '@/api/Addresses'

    var md = require('markdown-it')()
    md.use(require('markdown-it-new-katex'))
    md.use(require('markdown-it-container'), 'mesra')

    // ::: spoiler click me
    // *content*
    // :::

    export default {
        name: 'CreateOrEdit',
        components: {MarkdownBtn},
        computed: {
            renderedMatrixKatex () {
                return this.renderMatrixKatex()
            }
        },
        data: () => {
            return {
                selectedField: 0,
                whiteSpace: '                   ',
                test: [true, false, false, false],
                questMarkdownText: '# Math Rulez! \n  $x=\\frac{-b\\pm\\sqrt[]{b^2-4ac}}{2a}$',
                choicesMarkdownText: ['', '', '', ''],
                questRendered: '',
                formula: '',
                choiceRendered: ['', '', '', ''],
                latexData: null,
                choiceNumber: {
                    0: '1 ',
                    1: '2 ',
                    2: '3 ',
                    3: '4 '
                },
                trueChoiceIndex: 0,
                fieldId: [0],
                questionData: {
                    statement: '',
                    category_id: '',
                    sub_category_id: 1,
                    order: [],
                    choices: [
                        {
                            title: '',
                            order: 1,
                            answer: false
                        },
                        {
                            title: '',
                            order: 2,
                            answer: false
                        },
                        {
                            title: '',
                            order: 3,
                            answer: false
                        },
                        {
                            title: '',
                            order: 4,
                            answer: false
                        }
                    ]
                },
                currentQuestion: new Question(),
                url: '',
                editMode: false,
                quizList: new ExamList(),
                subCategoriesList: new QuestSubcategoryList(),
                selectedQuizzes: [],
                exams: [],
                matrixWidth: 1,
                matrixHeight: 1,
                matrixTempWidth: 1,
                matrixTempHeight: 1,
                matrix: [[]],
                determinan: false,
                matrixMenu: [{ title: 'دترمینان', value: true}, { title: 'ماتریس', value: false}]
            }
        },
        methods: {
            initMatrix () {
                this.matrix = []
                let row = []
                for (let i = 0; i < this.matrixHeight; i++) {
                    row = []
                    for (let j = 0; j < this.matrixWidth; j++) {
                        row.push(0)
                    }
                    this.matrix.push(row)
                }
                this.matrixTempWidth = this.matrixWidth
                this.matrixTempHeight = this.matrixHeight
            },
            renderMatrixKatex () {
                let matrixType = this.determinan ? 'vmatrix' : 'bmatrix'
                let matrixKatex = ''
                matrixKatex += '$\\begin{' + matrixType + '}'
                for (let i = 0; i < this.matrixTempHeight; i++) {
                    for (let j = 0; j < this.matrixTempWidth; j++) {
                        matrixKatex += this.matrix[i][j]
                        if (j !== this.matrixTempWidth - 1) {
                            matrixKatex += ' & '
                        }
                    }
                    if (i !== this.matrixTempHeight - 1) {
                        matrixKatex += ' \\\\ '
                    }
                }
                matrixKatex += '\\end{' + matrixType + '}$'
                // this.matrixTempHeight = 1
                // this.matrixTempWidth = 1
                return matrixKatex
            },
            replaceNimFasele () {
                if (!this.currentQuestion.statement) {
                    this.currentQuestion.statement = ''
                }
                this.currentQuestion.statement = this.currentQuestion.statement.replace('¬', '‌')
            },
            markdownBtnAddString ({elem, key, string}) {
                this.addString(elem, key, string)
            },
            addString (elem, key, string) {
                if (!elem[key]) {
                    elem[key] = ''
                }
                Vue.set(elem, key, elem[key] + string)
                this.updateRendered()
            },
            getExamById (quizId) {
                return this.exams.find((quiz) => quiz.id === quizId )
            },
            getQuizById (quizId) {
                return this.quizList.list.find((quiz) => quiz.id === quizId )
            },
            updateRendered () {
                this.replaceNimFasele()
                this.questRendered = md.render(this.currentQuestion.statement.toString());
                for (let i = 0; i < 4; i++) {
                    const title = (typeof this.currentQuestion.choices.list[i] !== 'undefined') ? this.currentQuestion.choices.list[i].title : null
                    if (title) {
                        this.choiceRendered[i] = md.render(title.toString())
                    }
                }
                this.replaceNimFasele()
            },
            selectAllExams () {
                if (this.selectedQuizzes.length !== this.quizList.list.length) {
                    this.selectedQuizzes = []
                    this.quizList.list.forEach((item) => { this.selectedQuizzes.push(item.id) })
                }
            },
            submitQuestion () {
                if (this.editMode) {
                    this.currentQuestion.choices.list.forEach((item) => { item.answer = false })
                    this.currentQuestion.choices.list[this.trueChoiceIndex].answer = true
                    this.currentQuestion.update(API_ADDRESS.question.updateQuestion(this.currentQuestion.id))
                        .then(() => {
                            this.$notify({
                                group: 'notifs',
                                title: 'توجه',
                                text: 'ویرایش با موفقیت انجام شد',
                                type: 'success'
                            })

                        })
                    return
                }
                this.currentQuestion.choices.list.forEach((item) => { item.answer = false })
                this.currentQuestion.choices.list[this.trueChoiceIndex].answer = true
                this.currentQuestion.exams = this.exams
                this.currentQuestion.create()
                    .then(() => {
                        this.currentQuestion.statement = ''
                        this.currentQuestion.choices.list.forEach((item) => { item.title = '' })
                        this.$notify({
                            group: 'notifs',
                            title: 'توجه',
                            text: 'ثبت با موفقیت انجام شد',
                            type: 'success'
                        })

                    })

            }
            // changeTrueChoice (index) {
            //     for (let i = 0; i < 4; i++) {
            //         if (i === index) {
            //
            //         }
            //     }
            // }
        },
        mounted() {

            // this.rendered = md.render();
            // this.updateRendered();

            let that = this
            const mf = MathLive.makeMathField(
                document.getElementById('mathfield'),
                {
                    virtualKeyboardMode: 'manual',
                    onContentDidChange: (mf) => {
                        that.latexData = mf.getValue()
                    },
                });
            // mf.$setConfig(
            //     //{ macros: { ...mf.getConfig('macros'), smallfrac: '{}^{#1}\\!\\!/\\!{}_{#2}', }, }
            // );
            that.latexData = mf.getValue()
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
                this.currentQuestion.show(null, API_ADDRESS.question.updateQuestion(this.$route.params.id))
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
