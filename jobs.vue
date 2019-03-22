<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_page_header" v-if="pageBanner" v-bind:style="{ background: 'linear-gradient(0deg, rgba(0,0,0,0.2), rgba(0,0,0,0.2)), #000 url(' + pageBanner.image_url + ') center center' }">
                    <div class="main_container position_relative">
                        <h2>Events & Promotions</h2>
                    </div>
                </div>
                <div class="main_container">
                    <div class="row">
                        <div class="col-md-12">
                            <breadcrumb></breadcrumb>
                        </div>
                    </div>
                    <div>hello</div>
                    <div v-if="toggleEvents">
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
                                <p>Sorry, there are no Events posted at this time. Please check back soon!</p>    
                            </div>
                        </div>
                    </div>
                    <div v-if="togglePromos">
                        <transition-group name="list" tag="div">
                            <div v-if="promos.length >= 1" v-for="item in promos" :key="item.id">
                                <div class="row event_container">
                                    <div class="col-sm-6 col-md-4">
                                        <img :src="item.image_url" :alt="'Promotion: ' + item.name" class="event_img img_max" />   
                                    </div>
                                    <div class="col-sm-6 col-md-8">
                                        <p v-if="item.promotionable_type == 'Property'" class="event_store_name">{{ property.name }}</p>
                                        <p v-else class="event_store_name">
                                            <router-link :to="{ name: 'storeDetails', params: { id: item.store.slug }}">
                                                {{ item.store.name }}
                                            </router-link>        
                                        </p>
                                        <h4 class="event_name">{{ item.name }}</h4>
                                        <p class="event_dates"><span v-if="isMultiDay(item)">{{ item.start_date | moment("MMMM D", timezone)}} - {{ item.end_date | moment("MMMM D", timezone)}}</span><span v-else>{{ item.start_date | moment("MMMM D", timezone)}}</span></p>
                                        <div class="event_desc" v-html="item.description_short"></div>
                                        <router-link :to="{ name: 'promotionDetails', params: { id: item.slug, banner: pageBanner }}">
                                            <div class="animated_btn event_link">View Promotion Details <i class="fas fa-angle-double-right"></i></div>
                                        </router-link>
                                        <hr class="event_seperator">
                                    </div>
                                </div>
                            </div>
                        </transition-group>
                        <div v-if="promos.length == 0">
                            <div class="row">
                                <div class="col-md-12">
                                    <p>Sorry, there are no Promotions posted at this time. Please check back soon!</p>    
                                </div>
                            </div>
                        </div>
                        <div class="row margin_60">
                            <div class="col-md-12">
                                <button class="animated_btn event_load_more" v-if="!noMorePromos" @click="handleButton">Load More</button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>

<script>
	define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "lightbox", "vue-lazy-load",  "vue-social-sharing", "json!site.json"], function(Vue, Vuex, moment, tz, VueMoment, Lightbox, VueLazyload, SocialSharing, site) {
        Vue.use(VueLazyload);
        Vue.component('social-sharing', SocialSharing);
		return Vue.component("events-and-promotions-component", {
			template: template, // the variable template will be injected,
			props: ['id'],
			data: function() {
				return {
					dataLoaded: false,
					pageBanner: null,
					currentEvent: null,
				    siteInfo: site,
				}
			},
			created() {
				this.$store.dispatch("getData", "events").then(response => {
				    var temp_repo = this.findRepoByName('Jobs Banner').images;
                    if(temp_repo != null) {
                        this.pageBanner = temp_repo[0];
                    } else {
                        this.pageBanner = {
                            "image_url": "//codecloud.cdn.speedyrails.net/sites/5b5f2c136e6f644fcb5b0100/image/jpeg/1529532304000/insidebanner2.jpg"
                        }
                    }
					this.dataLoaded = true;
				}, error => {
					console.error("Could not retrieve data from server. Please check internet connection and try again.");
				});
			},
			computed: {
				...Vuex.mapGetters([
					'property',
					'timezone',
					'processedEvents',
					'findEventBySlug',
					'findRepoByName'
				])
			},
			methods: {
				isMultiDay(currentEvent) {
					var timezone = this.timezone
					var start_date = moment(currentEvent.start_date).tz(timezone).format("MM-DD-YYYY")
					var end_date = moment(currentEvent.end_date).tz(timezone).format("MM-DD-YYYY")
					if (start_date === end_date) {
						return false
					} else {
						return true
					}
				},
				truncate(val_body) {
                    var truncate = _.truncate(val_body, { 'length': 99, 'separator': ' ' });
                    return truncate;
                },
				shareURL(slug) {
                    var share_url = window.location.href
                    return share_url
                },
			}
		});
	});
</script>