<template>

    <b-card class="mt-3">
        <b-row class="d-flex align-items-end">
            <template v-for="(req, index) in groupProgress.requirements">
                <b-col :key="req.id" cols="3">
                    <b-img-lazy :id="`popover-${index}-plant`" class="plant" :src="`STATIC IMAGE LINK`" ></b-img-lazy>
                    <b-popover :target="`popover-${index}-plant`" variant="success" placement="top" triggers="hover">
                        <template v-if=" getNextMilestone(req) !== -1" #title>Next milestone: {{ getNextMilestone(req) }}</template>
                        <template v-else #title>Congratulations!</template>
                        <!-- <template v-else #title>Congratulations!</template> -->
                        {{ req.description }} ({{ req.contributions }})
                    </b-popover>
                </b-col>
            </template>
        </b-row>
    </b-card>
    </template>
    <script>
    import { mapState } from 'vuex';
    export default {
        props: {
            groupProgress: {
                type: Object,
                required: true
            }
        },
        computed: {
            ...mapState({
                isDeployed: state => state.settings.isDeployed
            }),
        },
        methods: {
            getPlantImage(req) {
                let currentMilestone = 0
                for(let i = 0; i < req.milestones.length; i++) {
                    if(req.contributions >= req.milestones[i]) {
                        currentMilestone += 1
                    } 
                }
                return req.filenames[currentMilestone - 1]
            },
            getNextMilestone(req) {
                let currentMilestone = 0
                for(let i = 0; i < req.milestones.length; i++) {
                    if(req.contributions >= req.milestones[i]) {
                        currentMilestone += 1
                    } 
                }
                if(currentMilestone >= req.milestones.length) return -1
                return req.milestones[currentMilestone]
            }
        }
    }
    </script>
    <style scoped>
    /* Extra small devices (phones, 600px and down) */
    @media only screen and (max-width: 600px) {
        .plant {
            height: auto;
            width: 50px;
        }
    }
    
    /* Medium devices (landscape tablets, 768px and up) */
    @media only screen and (max-width: 768px) {
        .plant {
            height: auto;
            width: 60px;
        }
    }
    
    /* Large devices (laptops/desktops, 992px and up) */
    @media only screen and (max-width: 1199px) {
        .plant {
            height: auto;
            width: 70px;
        }
    }
    
    /* Extra large devices (large laptops and desktops, 1200px and up) */
    @media only screen and (min-width: 1200px) {
        .plant {
            height: auto;
            width: 80px;
        }
    }
    </style>