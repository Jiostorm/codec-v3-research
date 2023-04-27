<template>
    <div class="table-responsive">
        <b-card class="mt-3" title="Leaderboards">
            <b-table v-if="leaderboards && leaderboardsFields" hover :items="leaderboards" :fields="leaderboardsFields" table-variant="secondary" thead-class="text-center" tbody-tr-class="text-center">
                <template #cell(name)="data">
                    <div class="d-flex justify-content-begin align-items-center text-left">
                        <b-avatar :src="getProfileImageLink(data.item.learner.profile_image)"></b-avatar>
                        <span class="ml-2">{{ data.item.learner.first_name }} {{ data.item.learner.last_name }}</span>
                    </div>
                </template>
                <template #cell(points)="data">
                    {{ data.item.points }}
                </template>
                <template #cell(badge)="data">
                    <b-avatar v-if="data.item.featured_badge !== null" v-b-tooltip.hover :title="data.item.featured_badge.name" :src="`http://${isDeployedLink('static')}/badges/${data.item.featured_badge.filename}`" variant="none" ></b-avatar>
                </template>
            </b-table>
        </b-card>
    </div>
</template>
<script>
import { mapState } from 'vuex';
export default {
    props: {
        slug: {
            type: String,
            required: true
        }
    },
    data() {
        return {
            leaderboards: '',
            leaderboardsFields: [
                {
                    key: 'name',
                    sortable: false
                },
                {
                    key: 'points',
                    sortable: false
                },
                {
                    key: 'badge',
                    sortable: false
                },
            ],
            mainProps: { blank: true, width: 40, height: 40, class: 'm1' }
        }
    },
    computed: {
		...mapState({
			isDeployed: state => state.settings.isDeployed
		}),
    },
    mounted() {
        setTimeout(() => {
            if(this.$route.params.room.includes('comp-')) {
                this.$axios.get(`GET LEADERBOARDS`)
                    .then(res => {
                        
                        this.leaderboards = res.data
                    })
                    .catch(() => {
                        this.leaderboards = []
                    })
            }
        }, 100)
    },
    methods: {
        getProfileImageLink(picture){
			if (picture === '') {
				return 'https://i.pinimg.com/736x/2e/97/c5/2e97c5b9b9b1806d50f7fa25f01d035d.jpg'
			}

			return picture
        },
    },
}
</script>