<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_page_header" v-if="pageBanner" v-bind:style="{ background: 'linear-gradient(0deg, rgba(0,0,0,0.2), rgba(0,0,0,0.2)), #000 url(' + pageBanner.image_url + ') center center' }">
                    <div class="main_container position_relative">
                        <h2>Jobs</h2>
                    </div>
                </div>
                <div class="main_container">
                    <div class="row">
                        <div class="col-md-12">
                            <breadcrumb></breadcrumb>
                        </div>
                    </div>
                    <div v-if="toggleJobs">
                        <div v-if="eventList" v-for="(events, key) in eventList">
                            <div class="row">
                                <div class="col-md-12">
                                    <h3 class="event_date_heading">{{ key }}</h3> 
                                </div>
                            </div>
                            <div class="row event_container" v-for="event in events">
                                <div class="col-md-4">
                                    <img :src="event.image_url" :alt="'Event: ' + event.name" class="event_img img_max" />   
                                </div>
                                <div class="col-md-8">
                                    <h4 class="event_name">{{ event.name }}</h4>
                                    <p class="event_dates"><span v-if="event.tags && event.tags.length >0">{{event.tags[0]}} | </span> <span v-if="isMultiDay(event)">{{ event.start_date | moment("MMMM D", timezone)}} to {{ event.end_date | moment("MMMM D", timezone)}}</span><span v-else>{{ event.start_date | moment("MMMM D", timezone)}}</span></p>
                                    <div class="event_desc" v-html="event.description_short"></div>
                                    <router-link :to="{ name: 'eventDetails', params: { id: event.slug, banner: pageBanner }}">
                                        <div class="animated_btn event_link">View Event Details <i class="fas fa-angle-double-right"></i></div>
                                    </router-link>
                                    <hr class="event_seperator">
                                </div>
                            </div>
                        </div>
                        <div class="row margin_60" v-if="Object.keys(eventList).length === 0">
                            <div class="col-md-12">
                                <p>Sorry, there are no Jobs posted at this time. Please check back soon!</p>    
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>

<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "vue-meta", "vue-lazy-load", "vue-paginate"], function(Vue, Vuex, moment, tz, VueMoment, Meta, VueLazyload, VuePaginate) {
        Vue.use(Meta);
        Vue.use(VueLazyload);
        Vue.use(VuePaginate);
        return Vue.component("promos-component", {
            template: template, // the variable template will be injected
            props:['locale'],
            data: function() {
                return {
                    selectedDate: null,
                    filteredPromos:[],
                    dataloaded: false,
                    pageBanner: null,
                    // paginate: ['promos'],
                    promos : null,
                    incrementBy: 5,
                    showMore: 5,
                }
            },
            created() {
                this.loadData().then(response => {
                    this.dataloaded = true;
                    
                    var temp_repo = this.findRepoByName('Jobs Banner');
                    if(temp_repo && temp_repo.images) {
                        this.pageBanner = temp_repo.images[0];
                    }
                    else {
                        this.pageBanner = {};
                        this.pageBanner.image_url = "";
                    }
                    this.promos = this.promotions;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'processedJobs',
                    'findRepoByName',
                ]),
                promotions() {
                    var vm = this;
                    var temp_promo = [];
                    var temp_job = [];
                    _.forEach(this.processedJobs, function(value, key) {
                        today = moment().tz(vm.timezone);
                        webDate = moment(value.show_on_web_date).tz(vm.timezone);
                        if (today >= webDate) {
                            value.description_short = _.truncate(value.description, {
                                'length': 150
                            });
                            value.description_short_2 = _.truncate(value.description_2, {
                                'length': 150
                            });
                            // if (value.store != null && value.store != undefined && _.includes(value.store.store_front_url_abs, 'missing')) {
                            //     value.store.store_front_url_abs = vm.property.default_logo_url;
                            // }
                            // else if (value.store == null || value.store == undefined) {
                            //     value.store = {};
                            //     value.store.store_front_url_abs =  vm.property.default_logo_url;
                            // }
                            if (value.store  && _.includes(value.store.store_front_url_abs, 'missing')) {
                                // this.currentPromo.store.store_front_url_abs = this.property.default_logo_url;
                                value.store.no_store_logo = true;
                            }
                            else if (!value.store) {
                                value.store = {};
                                value.store.store_front_url_abs = vm.property.default_logo_url;
                            }
                            temp_promo.push(value);
                        }
                    });
                    temp_promo = _.sortBy(temp_promo, ['created_at', 'start_date']).reverse();
                    return temp_promo;
                },
            },
            methods: {
                loadData: async function() {
                    try {
                        // avoid making LOAD_META_DATA call for now as it will cause the entire Promise.all to fail since no meta data is set up.
                        let results = await Promise.all([this.$store.dispatch("getData", "jobs"), this.$store.dispatch("getData", "repos")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                loadMoreItems() {
                  if (this.showMore <= this.promos.length) {
                    var num = this.showMore + this.incrementBy;
                    this.showMore = num;
                  }
                }
            }
        });
    });
</script>