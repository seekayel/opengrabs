<template>
    <section class="section container">
        <div v-if="!expired.length && ! actives.length" class="box has-text-centered">
            <p>{{ $t('youDontHaveAnyTravel') }}.</p>
            <p>{{ $t('youCanPublish') }} <nuxt-link :to="localePath({ name: 'account-new', query: { adv: 'travel'}})">{{ $t('here') }}</nuxt-link></p>
        </div>
        <b-tabs v-else v-model="activeTab" position="is-centered" class="block" multiline>
            <b-tab-item v-if="expired.length">
                <template #header>
                    <span>{{ $t('expired') }}<b-tag rounded>{{ expired.length }}</b-tag></span>
                </template>
                <account-travels-expired :travels="expired" />
            </b-tab-item>

            <b-tab-item v-if="actives.length">
                <template #header>
                    <span>{{ $t('actives') }}<b-tag rounded>{{ actives.length }}</b-tag></span>
                </template>
                <account-travels-actives :travels="actives" />
            </b-tab-item>
        </b-tabs>
    </section>
</template>

<script>
import { mapState } from 'vuex'
export default {
    name: 'AccountTravels',
    middleware: 'auth',
    async fetch({ app, store }) {
        if (!store.state.account.travels.initiated) {
            const [expired, actives] = await Promise.all([
                app.$db.account.travels.filter('expired'),
                app.$db.account.travels.filter('actives')
            ])
            store.commit('account/travels/setExpired', expired)
            store.commit('account/travels/setActives', actives)
            store.commit('account/travels/setInitiated', true)
        } 
    },
    head() {
        return {
            title: this.$t('seo.myTravels')
        }
    },
    computed: {
        ...mapState({
            expired: (state) => state.account.travels.expired,
            actives: (state) => state.account.travels.actives
        }),
        activeTab: {
            get() {
                return this.$store.state.account.travels.activeTab
            },
            set(tab) {
                this.$store.commit('account/travels/setActiveTab', tab)
            },
        }
    }
}
</script>