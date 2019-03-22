<template>
	<div class="event_dets_container">
		hello
	</div>
</template>

<script>
    define(['Vue', 'vuex', 'moment', 'vue-lazy-load'], function(Vue, Vuex, moment, VueLazyload) {
        Vue.use(VueLazyload);
        return Vue.component("event-details-component", {
            template: template, // the variable template will be injected,
            data: function() {
                return {
                    currentEvent: null,
                    storeEvents : null,
                    storeHours : null,
                    pageBanner : null
                }
            },
            props:['id', 'locale'],
            beforeRouteUpdate(to, from, next) {
                this.currentEvent = this.findEventBySlug(to.params.id);
                    if (this.currentEvent === null || this.currentEvent === undefined){
                        this.$router.replace('/');
                    }
                next();
            },
            created(){
                this.loadData().then(response => {
                    this.updatecurrentEvent(this.id);
                    var temp_repo = this.findRepoByName('Events Banner');
                    if(temp_repo && temp_repo.images) {
                        this.pageBanner = temp_repo.images[0];
                    }
                    else {
                        this.pageBanner = {};
                        this.pageBanner.image_url = "";
                    }
                    this.events = this.event;
                });
            },
            // watch: {
            //     currentEvent : function (){
            //         if(this.currentEvent != null) {
            //             if (this.currentEvent.store != null && this.currentEvent.store != undefined && _.includes(this.currentEvent.store.store_front_url_abs, 'missing')) {
            //                 this.currentEvent.store.store_front_url_abs = this.property.default_logo_url;
            //             }
            //             else if (this.currentEvent.store == null || this.currentEvent.store == undefined) {
            //                 this.currentEvent.store = {};
            //                 this.currentEvent.store.store_front_url_abs =  this.property.default_logo_url;
            //             }
            //             var vm = this;
            //             var temp_event = [];
            //             var current_id =_.toNumber(this.currentEvent.id);
            //             _.forEach(this.currentEvent.store.event, function(value, key) {
            //                 if(_.toNumber(value) != current_id){
            //                     var current_event = vm.findEventById(value);
            //                     current_event.description_short = _.truncate(current_event.description, {'length': 70});
            //                     temp_event.push(current_event);
            //                 }
            //             });
            //             this.storeEvents = temp_event;
            //         }
            //         if(this.currentEvent.store) {
            //             var storeHours = [];
            //             var vm = this;
            //             _.forEach(this.currentEvent.store.store_hours, function (value, key) {
            //                 var hour = vm.findHourById(value);
            //                 if(hour.day_of_week === 0){
            //                     hour.order = 7;
            //                 }
            //                 else {
            //                     hour.order = hour.day_of_week;
            //                 }
            //                 storeHours.push();
            //             });
            //             this.storeHours = _.sortBy(storeHours, [function(o) { return o.order; }]);;
            //         }
            //     }
            // },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'processedEents',
                    'findEventBySlug',
                    'findEventById',
                    'timezone',
                    'findRepoByName',
                    'findHourById'
                ]),
                allEvents() {
                    return this.processedEvents;
                },
            },
            methods: {
                updatecurrentEvent (id) {
                    this.currentEvent = this.findEventBySlug(id);
                    if (this.currentEvent === null || this.currentEvent === undefined){
                        this.$router.replace('/');
                    }
                },
                loadData: async function() {
                    try {
                        // avoid making LOAD_META_DATA call for now as it will cause the entire Promise.all to fail since no meta data is set up.
                        let results = await Promise.all([this.$store.dispatch("getData", "events"), this.$store.dispatch("getData", "repos")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                }
            }
        });
    });
</script>