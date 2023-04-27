<template>
    <div>
        <b-container fluid>
            <!-- Breadcrumb -->
            <BreadCrumb
                v-if="links"
                :items="links"></BreadCrumb>
            <b-row class="mt-3">
                <b-col md="12" lg="3">
                    <b-card title="Live Coding" bg-variant="primary" text-variant="white"></b-card>
                    <!-- User List -->
                    <b-card title="List of Users">
                        <b-list-group>
                            <b-list-group-item :class="user.classes"> <!-- user.classes -->
                                <span>
                                    <b-avatar :src="getProfileImageLink(user.picture)" class="mr-3"></b-avatar>
                                    <span class="mr-auto">[{{ user.username }}] {{ user.firstName }}</span> <b-icon icon="circle-fill" variant="success"></b-icon> <b-icon v-if="user.isEditor" icon="pencil-square" variant="secondary"></b-icon>
                                </span>
                                <!-- <b-button v-show="!user.isEditor && user.isEditor" variant="outline-success" size="sm" @click="updateEditor(user.username)">Pass</b-button> -->
                            </b-list-group-item>
                            <template v-for="member in groupmates">
                                <b-list-group-item :key="member.id" :class="member.classes">
                                    <span>
                                        <b-avatar :src="getProfileImageLink(member.picture)" class="mr-3"></b-avatar>
                                        <span class="mr-auto">[{{ member.username }}] {{ member.firstName }}</span> <b-icon v-if="member.isOnline" icon="circle-fill" variant="success"></b-icon> <b-icon v-if="member.isEditor" icon="pencil-square" variant="secondary"></b-icon>
                                    </span>
                                    <!-- <b-button v-show="!user.isEditor && user.isEditor" variant="outline-success" size="sm" @click="updateEditor(user.username)">Pass</b-button> -->
                                    <b-button v-show="member.isOnline && user.isEditor" variant="success" size="sm" @click="updateEditor(member.username)">Pass</b-button>
                                </b-list-group-item>
                            </template>
                        </b-list-group>
                    </b-card>

                    <!-- Programming Language -->
                    <b-form-select v-model="selectedProgrammingLanguage" class="mt-3" :disabled="showCodeOverlay || !user.isEditor" :options="availableProgrammingLanguages" @change="updateProgrammingLanguage"></b-form-select>

                    <!-- Voice Call -->
                    <div class="mt-3">
                        <template v-if="call.stage === 0">
                            <b-button block variant="secondary" href="https://meet.google.com/new" target="_blank" :disabled="!user.isEditor" @click="callActions(1)">
                                <b-icon icon="google"></b-icon> Start call
                            </b-button>
                        </template>
                        <template v-else-if="call.stage === 1">
                            <b-input-group prepend="Call Link" class="mt-3">
                                <b-form-input v-model="call.link" type="text" ></b-form-input>
                                <b-input-group-append>
                                    <b-button block variant="success" @click="callActions(3)">Enter</b-button>
                                </b-input-group-append>
                            </b-input-group>
                        </template>
                        <template v-else-if="call.stage === 2">
                            <b-button block variant="success" :href="call.link" target="_blank" @click="callActions(3)">
                                <b-icon icon="telephone-fill"></b-icon> Join call
                            </b-button>
                        </template>
                        <template v-else-if="call.stage === 3">
                            <b-button v-if="user.isEditor" block variant="outline-danger" :disabled="!user.isEditor" @click="callActions(0)">
                                <b-icon icon="telephone-x-fill"></b-icon> End call
                            </b-button>
                            <b-button v-else block variant="outline-danger" @click="callActions(2)">
                                <b-icon icon="telephone-x-fill"></b-icon> Leave call
                            </b-button>
                        </template>
                    </div>
                    <!-- Return -->
                    <b-button class="mt-5 mb-3" block variant="outline-danger" @click="showReturnConfirmation">Return</b-button>
                </b-col>
                <b-col md="12" lg="9">
                    <b-overlay :show="showCodeOverlay" rounded="sm" variant="primary">
                        <div id="editor"></div>
                        <div class="mt-3">
                            <b-row>
                                <b-col cols="6">
                                    <b-button v-if="!showCustomInput" block variant="outline-dark" @click="showCustomInput = !showCustomInput">Custom Input</b-button>
                                    <b-button v-else block variant="dark" @click="showCustomInput = !showCustomInput">Show Output</b-button>
                                </b-col>
                                <b-col cols="6">
                                    <b-button ref="showCodeOverlay" block :disabled="showCodeOverlay || !user.isEditor" variant="outline-dark" @click="compileCode">Compile</b-button>
                                </b-col>
                            </b-row>
                        </div>
                        <b-card v-show="showCustomInput" class="mt-3" sub-title="Enter Custom Input">
                            <b-form-textarea
                                id="textarea"
                                v-model="customInput"
                                placeholder="Inputs go here"
                                rows="5"
                                max-rows="5"
                                :disabled="!user.isEditor"
                                @input="updatedTestCase"
                                >
                            </b-form-textarea>
                        </b-card>
                        <b-card v-show="!showCustomInput" class="mt-3">
                            <template v-if="testResponse !== ''">
                                <div class="table-responsive">
                                    <table>
                                        <thead>
                                            <tr>
                                                <td>Language</td>
                                                <td class="ml-2">Description</td>
                                                <td class="ml-2">Time (sec)</td>
                                                <td class="ml-2">Memory (KiB)</td>
                                            </tr>
                                        </thead>
                                        <tbody>
                                            <tr>
                                                <td v-if="testResponse !== ''" class="text-muted">{{ testResponse.language.name }}</td>
                                                <td class="text-muted">{{ testResponse.status.description }}</td>
                                                <td class="text-muted">{{ testResponse.time }}</td>
                                                <td class="text-muted">{{ testResponse.memory }}</td>
                                            </tr>
                                        </tbody>
                                    </table>
                                </div>
                                <div>
                                    <span>Input</span>
                                    <p class="text-muted" style="white-space: pre-line">{{ decodeBase64(testResponse.stdin) }}</p>
                                    <span>Output</span>
                                    <p class="text-muted" style="white-space: pre-line">{{ decodeBase64(testResponse.stdout) }}</p>
                                    <template v-if="testResponse.stderr !== null || testResponse.compile_output !== null ">
                                        <span>Error</span>
                                        <p class="text-muted" style="white-space: pre-line">{{ decodeBase64(testResponse.stderr) }}</p>
                                        <p class="text-muted" style="white-space: pre-line">{{ decodeBase64(testResponse.compile_output) }}</p>
                                    </template>
                                </div>
                            </template>
                            <p v-else>Compile to view the output of your code</p>
                        </b-card>
                        <template #overlay>
                            <div class="text-center text-white">
                                <b-icon icon="stopwatch" font-scale="3" animation="cylon"></b-icon>
                                <p id="cancel-label">Your program is compiling...</p>
                            </div>
                        </template>
                    </b-overlay>
                </b-col>
            </b-row>
        </b-container>
    </div>
</template>
<script>
import loader from '@monaco-editor/loader';
import { io } from "socket.io-client";
import { mapState } from 'vuex';    

export default {
    layout: 'problem',
    data() {
        return {
            roomTitle: '',
            user: {},
            receive: "",
            // Group ID:
            liveCodingID: "",
            // Editor
            editor: "",
            // Design
            showCodeOverlay: false,
            // User List
            groupmates: [],
            // Programming Language
            languageMap: {
                python: 71,
                cpp: 53,
                c: 50,
                java: 62,
            },
            selectedProgrammingLanguage: 'c',
            availableProgrammingLanguages: [
                { value: 'c', text: 'C', languageID: 50 },
                { value: 'cpp', text: 'C++', languageID: 53 },
                { value: 'java', text: 'Java', languageID: 62 },
                { value: 'python', text: 'Python', languageID: 71 },
            ],
            // Voice Call
            call: {
                stage: 0, // 0 for start call, 1 for form, 2 for join call, 3 for call in progress or end call
                link: ""
            },
            // Monaco
            options: {
                theme: "vs-dark",
                language: "cpp",
                selectOnLineNumbers: true,
                // readonly shall be TRUE
                readOnly: true,
                value: "",
                minimap: { enabled: true },
                automaticLayout: true
            },
            // Custom Input
            showCustomInput: false,
            customInput: "",
            monacoInstance: "",
            // Socket
            socket: null,
            joined: false,
            // Output
            testResponse: '',
            submissionInterval: null
        }
    },
	head() {
        return {
		    title: `Live Coding in ${this.roomTitle} / CodeC`
        }
	},
    computed: {
		...mapState({
			isDeployed: state => state.settings.isDeployed
		}),
        links() {
            const items = [
                {
                    text: 'Code Room Dashboard',
                    to: { name: 'dashboard' }
                },
                {
                    text: `${this.roomTitle}`,
                    to: { name: 'r-room', params: { room: this.$route.params.room } }
                },
                {
                    text: 'Live Coding',
                    active: true
                }
            ]

            return items
        },
        userType() {
            try {
                return this.$auth.user.type
            } catch {
                return ''
            }
        },
        slug() {
            // Remove extra character (security measure) 
            const paramsRoom = this.$route.params.room
            return paramsRoom.slice(0, paramsRoom.length - 1)
        },
    },
    beforeDestroy() {
        this.socket.disconnect()
        this.joined = false
        clearInterval(this.interval)
    },
    async mounted() {
        this.socket = io('SOCKET ENDPOINT')
        loader.init().then(monaco => {
            this.monacoInstance = monaco.editor.create(
                document.getElementById("editor"),
                this.options
            );
        });
        setInterval(() => {
            
            if (this.socket.connected && this.joined && this.user.isEditor) {
                this.socket.emit('update-on-type', this.monacoInstance.getValue())
                this.setCodeValue(this.monacoInstance.getValue())
            }
        }, 1000)
        // Get Room Information
        await this.$axios.get(`ROOM INFORMATION`)
            .then(res => {
                const data = res.data

                // Set Room Values
                this.roomTitle = data.name
            })
            .catch(err => {
                if (err) {
                    this.$bvToast.toast('Refresh the page in a while', {
                            title: `Error Retrieving Rooms`,
                            variant: 'danger',
                            toaster: 'b-toaster-bottom-right',
                            solid: true,
                            appendToast: true
                        })
                }
            })
        // Get Group ID from the user's room
        await setTimeout(() => {
            this.user = {
                username: this.$auth.user.auth.username,
                firstName: this.$auth.user.first_name,
                isEditor: false,
                picture: this.$auth.user.profile_image,
                classes: [],
                isOnline: true
            }

            this.$axios.get(`GET GROUP TRACKER ID`)
            .then(res => {
                const enrollees = res.data
                let curUser = ''
                for(const learner in enrollees) {
                    if(enrollees[learner].learner.id === this.$auth.user.id) curUser = enrollees[learner]
                }
                this.liveCodingID = curUser.group_progress.liveroom
                // get groupmates
                for(const learner in enrollees) {
                    if(enrollees[learner].group_progress.id === curUser.group_progress.id && enrollees[learner].learner.id !== this.$auth.user.id) {
                        this.groupmates.push({
                            username: enrollees[learner].learner.auth.username,
                            firstName: enrollees[learner].learner.first_name,
                            picture: enrollees[learner].learner.profile_image,
                            classes: [],
                            isEditor: false,
                            isOnline: false
                        })
                    }
                }

                this.$axios.get(`GET LIVE CODING ROOM DATA`)
                    .then(res => {
                        const data = res.data

                        if(data.learners.length === 0) {
                            this.user.isEditor = true
                            this.options.readOnly = false
                        } else {
                            this.groupmates.forEach(member => {
                                if(res.data.learners.find(learner => learner.auth.username === member.username)) {
                                    member.isOnline = true
                                }
                            })
                            this.setEditor(data.live_session.editor.auth.username)
                        }

                        this.socket.emit('active', this.user.username, this.liveCodingID)
                        this.socket.emit('join')

                        this.joined = true

                        this.$axios.get(`GET UPDATED LIVE CODING ROOM DATA`) 
                        .then(res => {
                            const data = res.data

                            if(data.live_session.language_used !== '') this.setProgrammingLanguage(data.live_session.language_used)
                            this.call.link = data.live_session.call_link
                            if(this.call.link !== '') this.call.stage = 2
                            this.setCodeValue(data.live_session.code)
                        })

                        this.setMonacoEditor()

                        this.userListUpdateClass()
                    })

                this.socket.on('joined', (username) => {

                    this.setOnlineUsers(username)
                    this.userListUpdateClass()
                })

                this.socket.on('editor-disconnected', (username, newEditor) => {

                    this.setOfflineUsers(username)
                    this.setEditor(newEditor)
                    this.userListUpdateClass()
                })

                this.socket.on('learner-disconnected', (username) => {

                    this.setOfflineUsers(username)
                    this.userListUpdateClass()
                })

                this.socket.on('updated-editor', (username) => {
                    this.setEditor(username)
                })

                this.socket.on('updated-language', (language) => {
                    this.setProgrammingLanguage(language)
                })

                this.socket.on('receive-call-link', (link) => {
                    this.call.link = link
                    this.call.stage = 2
                })

                this.socket.on('terminate-call', () => {
                    this.call.link = ''
                    this.call.stage = 0
                })

                this.socket.on('updated-type', (code) => {
                    this.setCodeValue(code)
                })

                this.socket.on('updated-test-case', (testcase) => {
                    this.customInput = testcase
                })

                this.socket.on('pass-output', (payload) => {
                    this.showCustomInput = payload.showCustomInput
                    this.testResponse = payload.testResponse

                    this.$bvToast.toast(`View output below the code editor`, {
                        title: 'Compilation Success!',
                        variant: 'success',
                        toaster: 'b-toaster-bottom-right',
                        solid: true,
                        appendToast: true,
                        
                    })
                })
            })
        }, 250)            
    },
    methods: {
        callActions(stage) {
            this.call.stage = stage
            // Socket events only happens when the user is an editor
            if(this.user.isEditor) {
                if(stage === 0) {
                    // Editor End Call upon end
                    this.socket.emit('end-call')
                } else if (stage === 3) {
                    // Editor Start Call upon enter
                    this.socket.emit('start-call', this.call.link)
                } 
            }
        },
        getProfileImageLink(picture){
			if (picture === '') {
				return 'https://i.pinimg.com/736x/2e/97/c5/2e97c5b9b9b1806d50f7fa25f01d035d.jpg'
			}

			return picture
        },
        setMonacoEditor() {
            // Update moanaco
            if (this.socket.connected && this.joined && this.monacoInstance) {
                this.monacoInstance.updateOptions({
                    readOnly: this.options.readOnly,
                    language: this.options.language,
                    value: this.options.value
                })
                const currentPosition = this.monacoInstance.getPosition();
                this.monacoInstance.getModel().setValue(this.options.value);
                this.monacoInstance.setPosition(currentPosition)
            }
        },
        setOnlineUsers(username) {
            this.groupmates.forEach(member => {
                if(member.username === username) {
                    member.isOnline = true
                }
            })
        },
        setOfflineUsers(username) {
            this.groupmates.forEach(member => {
                if(member.username === username) {
                    member.isOnline = false
                }
            })
        },
        setEditor(editor) {
            this.groupmates.forEach(member => {
                if(member.username === editor) {
                    member.isEditor = true
                } else {
                    member.isEditor = false
                }
            })

            if(editor === this.user.username) {
                this.user.isEditor = true
                this.options.readOnly = false
            } else {
                this.user.isEditor = false
                this.options.readOnly = true
            }

            this.setMonacoEditor()
            // Update classes
            this.userListUpdateClass()
        },
        setProgrammingLanguage(language) {
            this.selectedProgrammingLanguage = language
            this.options.language = language
            
            this.setMonacoEditor()
        },
        setCodeValue(code) {
            this.options.value = code
            
            this.setMonacoEditor()
        },
        userListUpdateClass() {
            if(this.user.isEditor) {
                this.user.classes = ['d-flex align-items-center text-white bg-primary']
            } else {
                this.user.classes = ['d-flex align-items-center']
            }

            if(this.groupmates.length !== 0) {
                this.groupmates.forEach (member => {
                    member.classes = 'd-flex align-items-center justify-content-between'
                    /* eslint-disable no-console */
                    console.log(member, member.isEditor)
                    if (member.isEditor) {
                        member.classes += " bg-primary text-white"
                    }
                })
            }
        },
        updateProgrammingLanguage() {
            this.socket.emit('update-language', this.selectedProgrammingLanguage)
            this.setProgrammingLanguage(this.selectedProgrammingLanguage)
        },
        updateEditor(username){
            this.socket.emit('update-editor', username)
            this.setEditor(username)
        },
        updatedTestCase() {
            this.socket.emit('update-test-case', this.customInput)
        },
        compileCode() {
            // LOGIC GOES HERE
        },
        showReturnConfirmation() {
            this.$bvModal.msgBoxConfirm('Return to the room page?', {
                title: 'Please Confirm',
                size: 'sm',
                headerBgVariant: 'danger',
                headerTextVariant: 'light',
                okVariant: 'danger',
                okTitle: 'Yes',
                cancelTitle: 'No',
                cancelVariant: 'outline-dark',
                footerClass: 'p-2',
                hideHeaderClose: false,
                centered: true
            })
            .then(value => {
                if (value === true) {
                    this.$router.push({ name: 'r-room', params: {room: this.$route.params.room}})
                }
            })
        },
        updateUserEditor() {
            this.user.isEditor = true
        },
        getLanguageID(language) {
            let languageID = null
            this.availableProgrammingLanguages.every(lang => {
                if(lang.value === language) {
                    languageID = lang.languageID
                    return false
                }
                return true
            })
            return languageID
        },
        decodeBase64(base64String) {
            if(base64String) {
                // decode the Base64-encoded string to a binary string
                const binaryString = atob(base64String);

                // convert the binary string to a UTF-8 string
                const utf8String = new TextDecoder().decode(new Uint8Array([...binaryString].map(c => c.charCodeAt(0))));

                return utf8String
            }
        }
    }
}
</script>
<style scoped> 
th, td {
    padding: 10px 10px 0px 0px;
}
#editor {
  resize: vertical;
  overflow: auto;
}
@media only screen and (min-width: 992px) {
    #editor {
        width: auto;
        height: 650px
    }
}

@media only screen and (max-width: 991px) {
    #editor {
        width: auto;
        height: 400px
    }
}
</style>