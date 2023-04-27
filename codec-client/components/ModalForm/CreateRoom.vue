<template>
    <!-- BUG: ADD FORM AND LOGIC -->
    <!-- Create Room Modal -->
        <b-modal 
        id="modal-create-room" 
        v-model="openModal"
        title="Create Code Room" size="lg" header-bg-variant="success" header-text-variant="light" ok-title="Create" ok-variant="success" cancel-variant="outline-dark"
        @show="openModal = true"
        @hide="onHide"
        @hidden="resetModal"
        @ok="handleOk">
            <b-container>
                <ValidationObserver ref="observer" v-slot="{ handleSubmit }">
                <b-form ref="form" @submit.prevent="handleSubmit(processSubmit)" @input="formModified = true">
                <b-row>
                    <b-col lg="6">
                        <ValidationProvider
                            v-slot="validationContext"
                            rules="required"
                            >
                                <!-- Room Name -->
                                <b-form-group
                                invalid-feedback="Room Name is required"
                                >
                                <label label-for="room-name-input" class="d-block">
                                    Room Name
                                </label>
                                    <b-form-input
                                    id="room-name-input"
                                    v-model="form.roomName"
                                    :state="getValidationState(validationContext)"
                                ></b-form-input>
                                <span class="form-text text-muted" name="room-name">
                                    <small>[{{ roomNameLength }}/35] 35 characters is optimal. Exceeding characters are truncated.</small>
                                </span>
                                </b-form-group>
                            </ValidationProvider>
                    </b-col>
                    <b-col lg="6">
                        <ValidationProvider
                            v-slot="validationContext"
                            rules="required"
                            >
                            <b-form-group
                                label="Room Type"
                                label-for="room-type-input"
                                invalid-feedback="Select a Room Type"
                                >
                                <b-form-select id="room-type-input" v-model="form.roomType" :options="roomTypeChoices" :state="getValidationState(validationContext)"></b-form-select>
                            </b-form-group>
                        </ValidationProvider>
                    </b-col>
                </b-row>
                <b-row>
                    <b-col lg="6">
                        <b-form-group
                            invalid-feedback="Room Description is required"
                            >
                            <label label-for="room-description-input" class="d-block">
                                Room Description
                            </label>
                            <b-form-input
                                id="room-description-input"
                                v-model="form.roomDescription"
                            ></b-form-input>
                            <span class="form-text text-muted" name="room-description">
                                <small>[{{ roomDescriptionLength }}/28] 28 characters is optimal. Exceeding characters are truncated.</small>
                            </span>
                        </b-form-group>
                    </b-col>
                </b-row>
                </b-form>
                </ValidationObserver>
            </b-container>
    </b-modal>  
</template>
<script>
import { ValidationObserver } from 'vee-validate';
import { mapState } from 'vuex';
export default {
    components: { ValidationObserver },
    emits: ["update-dashboard"],
    data() {
        return {
            form: {
                roomName: "",
                roomDescription: "",
                roomType: 'Competitive',
            },
            roomTypeChoices: [
                { value: "Competitive", text: "Competitive" },
                { value: "Cooperative", text: "Cooperative" }
            ],
            openModal: false,
            formModified: false,
            isOk: false
        };
    },
    computed: {
        roomNameLength() {
            return this.form.roomName.length;
        },
        roomDescriptionLength() {
            return this.form.roomDescription.length;
        },
        roomCodeLength() {
            return this.roomCode.length;
        },
        ...mapState({
            isDeployed: state => state.settings.isDeployed
        })
    },
    methods: {
		getValidationState({ dirty, validated, valid = null }) {
			return dirty || validated ? valid : null;
		},
        onHide(event) {
            if(!this.isOk) {
                if (this.formModified) {
                    event.preventDefault()
                    this.$bvModal.msgBoxConfirm('Are you sure you want to close this? Your changes may not be saved.', {
                        title: 'Please Confirm',
                        size: 'sm',
                        headerBgVariant: 'danger',
                        headerTextVariant: 'light',
                        okVariant: 'danger',
                        okTitle: 'Yes',
                        cancelVariant: 'outline-dark',
                        cancelTitle: 'No',
                        footerClass: 'p-2',
                        hideHeaderClose: false,
                        centered: true
                    })
                    .then(value => {
                        if (value === true) {
                            this.openModal = false
                            this.formModified = false
                        } 
                    })
                } else {
                    this.openModal = false
                }
            }
        },
        resetModal() {
            this.form.roomName = "";
            this.form.roomDescription = "";
            this.roomCode = "";
            this.form.roomType = null;
            this.isOk = false
            this.formModified = false
        },
        async handleOk(bvModalEvent) {
            this.isOk = true
            // Prevent modal from closing
            bvModalEvent.preventDefault();

            await this.processSubmit()
        },
        async processSubmit() {
            // variables
            const payload = {
                name: this.form.roomName.trim(),
                description: this.form.roomDescription.trim(),
                type: this.form.roomType,
                mentor: this.$auth.user.auth.username
            };
            // Send to database
            await this.$axios.post(`ROOM PUBLISH API HERE`, payload)
                .then(res => {
                if (this.form.roomType === "Cooperative") {
                    const gpsPayload = {
                        requirements: [
                            {
                                type: "sub",
                                description: "Submitted Solutions as a group",
                                milestones: [0, 9, 24, 45],
                                // milestones: [0, 1, 2, 3],
                                contributions: 0,
                                filenames: ["tracker-1-1.png", "tracker-1-2.png", "tracker-1-3.png", "tracker-1-4.png"]
                            },
                            {
                                type: "stk",
                                description: "Day Streak as a group",
                                milestones: [0, 4, 10, 16],
                                // milestones: [0, 1, 2, 3],
                                contributions: 0,
                                filenames: ["tracker-2-1.png", "tracker-2-2.png", "tracker-2-3.png", "tracker-2-4.png"]
                            },
                            {
                                type: "pts",
                                description: "Points as a group",
                                milestones: [0, 120, 270, 450],
                                // milestones: [0, 2, 4, 6],
                                contributions: 0,
                                filenames: ["tracker-3-1.png", "tracker-3-2.png", "tracker-3-3.png", "tracker-3-4.png"]
                            },
                            {
                                type: "two",
                                description: "Accepted Solution for the Same Problem Using at Least Two Different Programming Languages",
                                milestones: [0, 5, 12, 20],
                                // milestones: [0, 1, 2, 3],
                                contributions: 0,
                                filenames: ["tracker-4-1.png", "tracker-4-2.png", "tracker-4-3.png", "tracker-4-4.png"]
                            },
                        ],
                        room: res.data.slug
                    };
                    // repeat the request 10 times on deployment
                    for (let i = 0; i < 10; i++) {
                        this.$axios.post(`GROUP PROGRESS SETUP`, gpsPayload)
                            .then(res);
                    }
                }
                setTimeout(() => {
                    this.$bvToast.toast(`You have successfully created ${payload.name}`, {
                        title: "You're good to go!",
                        variant: "success",
                        toaster: "b-toaster-bottom-right",
                        solid: true,
                        appendToast: true,
                    });
                });
                this.$emit("update-dashboard", this.$auth.user.auth.username, this.$auth.user.type);
                this.openModal = false
            })
                .catch(err => {;
                if (err) {
                    this.openModal = true
                    this.$bvToast.toast(`Provide the room details`, {
                        title: "Oops, something happened",
                        variant: "danger",
                        toaster: "b-toaster-bottom-right",
                        solid: true,
                        appendToast: true,
                    });
                }
            });
        }
    },
}
</script>