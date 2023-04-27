<template>
    <div>
        <b-list-group-item class="mt-3 d-flex justify-content-between align-items-center" variant="secondary">
            <div>
                <h4 class="text-primary">Your Badges</h4>
                <span class="mr-auto">View Your Badges and Pin a Badge!</span>
            </div>
            <div v-b-modal.badge-collection>
                <b-avatar v-if="badges && hasNewBadge()" badge badge-top badge-variant="danger" icon="arrow-right-circle" variant="none"></b-avatar>
                <b-avatar v-else icon="arrow-right-circle" variant="none"></b-avatar>
            </div>
            <b-modal id="badge-collection" title="Pin Your Desired Badge By Clicking On It" ok-only header-bg-variant="dark" header-text-variant="light" body-bg-variant="dark" footer-bg-variant="dark" @hidden="viewBadges">
                <b-row>
                    <template v-for="badge in badges">
                        <b-col :key="badge.id" class="d-flex justify-content-center mt-3" cols="4" @click="updateFeaturedBadge(badge)">
                            <b-avatar v-b-tooltip.hover :title="getBadgeDescription(badge.name, badge.description)" :src="`GET BADGE IMAGE`" variant="none" class="badges" :badge-variant="getBadgeColor(badge.id, badge.name, badge.is_viewed)">
                                <template v-if="badge.id === localFeaturedBadge.id && badge.name !== 'Locked'" #badge><b-icon icon="star-fill"></b-icon></template>
                                <template v-else-if="!badge.is_viewed && badge.name !== 'Locked'" #badge><b-icon icon="circle" :variant="getBadgeColor(badge.id, badge.name, badge.is_viewed)"></b-icon></template>
                            </b-avatar>
                        </b-col> 
                    </template>
                </b-row> 
            </b-modal>
        </b-list-group-item>
    </div>
</template>
<script>
import { mapState } from 'vuex';
export default {
    props: {
        slug: {
            type: String,
            required: true
        },
        username: {
            type: String,
            required: true
        },
        featuredBadge: {
            type: Object,
            required: true
        }
    },
    emits: ['update-dashboard', 'display-toast'],
    data() {
        return {
            badges: [],
            localFeaturedBadge: {}
        }
    },
    computed:{
		...mapState({
			isDeployed: state => state.settings.isDeployed
		}),
    },
    async mounted() {
        this.localFeaturedBadge = await this.featuredBadge
        await setTimeout(() => {
            this.$axios.get(`GET BADGES`)
            .then(res => {
                
                this.badges = res.data
                const badgeCount = this.badges.length - 1
                for(let i = 0; i < (12 - badgeCount); i++) {
                    this.badges.push({
                        name: 'Locked',
                        filename: 'locked.png'
                    })
                }
            })
            .catch(() => {
                const badgeCount = this.badges.length - 1
                for(let i = 0; i < (12 - badgeCount); i++) {
                    this.badges.push({
                        name: 'Locked',
                        filename: 'locked.png'
                    })
                }
            })
        }, 100)
    },
    methods: {
        getBadgeColor(id, name, isViewed) {
            if (name !== 'Locked' && id) {
                if(id === this.localFeaturedBadge.id) return 'warning'
                if(isViewed === 0 ) return 'danger'
            }
        },
        getBadgeDescription(name, description){
            if (name && description !== undefined) return `${name} - ${description}`
        },
        updateFeaturedBadge(badge) {
            if(badge.id) {
                const payload = {
                    username: this.username,
                    badge: badge.id
                }

                this.$axios.patch(`PIN BADGE`, payload)
                .then(() => {
                    this.localFeaturedBadge = badge
                    this.localFeaturedBadge.is_viewed = true
                    
                    this.$bvToast.toast('Refresh to see the changes in the Leaderboards', {
                        title: `Congratulations, ${this.username}`,
                        variant: 'success',
                        toaster: 'b-toaster-bottom-right',
                        solid: true,
                        appendToast: true,
                        
                    })
                })
                .catch()

                this.viewBadges()
            }
        },
        viewBadges() {
            this.badges.forEach((badge, index) => {
                if(badge.name !== 'Locked' && badge.is_viewed === 0) {
                    badge.is_viewed = 1
                    this.$axios.patch(`VIEW BADGE`)
                }
            })
        },
        hasNewBadge() {
            const newBadgeCount = this.badges.filter(badge => badge.is_viewed === 0).length
            if(newBadgeCount > 0) return true
            return false
        }
    }
}
</script>
<style scoped>
.badges {
    width: 100px;
    height: auto;
}
.badges:hover {
    cursor: pointer;
}
.popover-background {
    background-color: #737184;
}
</style>