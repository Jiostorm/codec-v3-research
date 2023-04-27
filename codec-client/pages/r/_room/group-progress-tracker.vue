<template>
    <b-container fluid>
        <!-- Breadcrumb -->
        <BreadCrumb :items="links"></BreadCrumb>
        <!-- Card Header -->
        <DashboardCardHeader
            v-if="roomTitle && roomAuthor && roomDescription && roomType"
            :room-code="slug"
            :room-title="roomTitle"
            :room-author="roomAuthor"
            :room-description="roomDescription"
            :room-type="roomType">
        </DashboardCardHeader>
        <!-- Group Progress Tracker Table (Scrollable) -->
        <div class="table-responsive">
            <b-table class="mt-3" hover :items="record" :fields="recordFields" table-variant="secondary" thead-class="text-center" tbody-tr-class="text-center">
                <template #cell(groupName)="data">
                    Team {{ getTeamNumber(data.item.groupID) }}
                </template>
                <template #cell(groupMembers)="data">
                    <div class="text-left">
                        <span style="white-space: pre-line">{{ processArray(data.item.groupMembers) }}</span>
                    </div>
                </template>
                <template #cell(submittedSolutions)="data">
                    {{ data.item.submittedSolutions }}
                </template>
                <template #cell(dayStreaks)="data">
                    {{ data.item.dayStreaks }}
                </template>
                <template #cell(points)="data">
                    <span>{{ data.item.points }}</span>
                </template>
                <template #cell(twoProgrammingLanguages)="data">
                    {{ data.item.twoProgrammingLanguages }}
                </template>
                <template #cell(requirements)="data">
                    <div class="d-flex justify-content-center align-items-end">
                        <template v-for="req in data.item.requirements">
                            <b-img-lazy :key="req.id" class="plants" :src="`PLANT IMAGES`"></b-img-lazy>
                        </template>
                    </div>
                </template>
            </b-table>
        </div>
    </b-container>
</template>
<script>
import { mapState } from 'vuex';
export default {
    layout: 'default',
    data() {
        return {
            // General
            roomTitle: '',
            roomType: '',
            // Mentor
            roomAuthor: '',
            roomDescription: '',
            // Students
            enrollees: '',
            // Group Progress Tracker Record
            recordFields: [
                {
                    key: 'groupName',
                    sortable: false
                },
                {
                    key: 'groupMembers',
                    sortable: false
                },
                {
                    key: 'submittedSolutions',
                    sortable: true
                },
                {
                    key: 'dayStreaks',
                    sortable: true
                },
                {
                    key: 'points',
                    sortable: true
                },
                {
                    key: 'twoProgrammingLanguages',
                    sortable: true
                },
                {
                    key: 'requirements',
                    sortable: false
                }
            ],
            record: [],
            teams: new Set()
        }
    },
	head() {
        return {
		    title: `Group Progress Trackers in ${this.roomTitle} / CodeC`
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
                    text: 'Group Progress Trackers',
                    active: true
                }
            ]

            return items
        },
        slug() {
            // Remove extra character (security measure) 
            const paramsRoom = this.$route.params.room
            return paramsRoom.slice(0, paramsRoom.length - 1)
        }
    },
    async mounted() {
        await setTimeout(() => {
            // Get Room Information
            this.$axios.get(`ROOM INFORMATION`)
                .then(res => {
                    const data = res.data

                    // Set Room Values
                    this.roomTitle = data.name
                    this.roomDescription = data.description
                    this.roomType = data.type

                    // Set Student Values
                    this.enrollees = data.enrollees

                    // Set Mentor Values
                    this.roomAuthor = `${data.mentor.first_name} ${data.mentor.last_name}`
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
                
            // Get GPTs
            this.$axios.get(`GROUP PROGRESS TRACKERS`)
                .then(res => {
                    const gps = res.data

                    gps.forEach(tracker => {
                        const processedTracker = {
                            groupID: tracker.id,
                            groupMembers: this.enrollees.filter(enrollee => enrollee.group_progress.id === tracker.id).sort(this.compareViaLastName),
                            submittedSolutions: tracker.requirements[0].contributions,
                            dayStreaks: tracker.requirements[1].contributions,
                            points: tracker.requirements[2].contributions,
                            twoProgrammingLanguages: tracker.requirements[3].contributions,
                            requirements: tracker.requirements
                        }

                        this.record.push(processedTracker)
                        this.teams.add(tracker.id)
                    })

                    const teamsArray = Array.from(this.teams)
                    this.teams = teamsArray.sort()
                })
                .catch(err => {
                    if (err) {
                        this.$bvToast.toast('Refresh the page in a while', {
                                title: `Error Retrieving Trackers`,
                                variant: 'danger',
                                toaster: 'b-toaster-bottom-right',
                                solid: true,
                                appendToast: true
                            })
                    }
                })
        }, 100)
    },
    methods:{
        compareViaLastName(a, b) {
            const nameA = a.learner.last_name.toUpperCase(); // ignore upper and lowercase
            const nameB = b.learner.last_name.toUpperCase(); // ignore upper and lowercase
            if (nameA < nameB) {
                return -1;
            }
            if (nameA > nameB) {
                return 1;
            }

            // names must be equal
            return 0;
        },
        getTeamNumber(team) {
            return this.teams.indexOf(team)
        },
        processArray(data) {
            let output = ""
            for (let i = 0; i < data.length; i++) {
                output += data[i].learner.first_name + ' ' + data[i].learner.last_name
                if (i !== data.length - 1) {
                    output += "\n"
                }
            }

            return output
        },
        getPlantImage(req) {
            let currentMilestone = 0
            for(let i = 0; i < req.milestones.length; i++) {
                if(req.contributions >= req.milestones[i]) {
                    currentMilestone += 1
                } 
            }
            return req.filenames[currentMilestone - 1]
        }
    }
}
</script>
<style scoped>
.plants {
    width: 25px;
    height: auto;
}
</style>