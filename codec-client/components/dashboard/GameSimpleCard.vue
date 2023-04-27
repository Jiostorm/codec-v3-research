<template>
    <b-list-group-item class="d-flex align-items-center mt-3" :variant="cardVariant">
        <span class="mr-auto"><strong>{{ label }}</strong></span>
        <h5><b-badge :variant="cardVariant"><b-icon v-if="label !== 'Your Points'" :icon="icon"></b-icon> {{ number }}</b-badge></h5>
    </b-list-group-item>
</template>
<script>
export default {
    props: {
        number: {
            type: Number,
            required: true
        },
        label: {
            type: String,
            required: true
        },
        cardVariant: {
            type: String,
            required: true
        },
        icon: {
            type: String,
            required: true
        },
        iconVariant: {
            type: String,
            required: true
        }
    }
}
</script>