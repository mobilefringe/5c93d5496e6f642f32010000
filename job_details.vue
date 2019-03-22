<template>
    <div> <!-- Without an outer container div this component template will not render -->
        <p>hello</p>
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_page_header" v-if="pageBanner" v-bind:style="{ background: 'linear-gradient(0deg, rgba(0,0,0,0.2), rgba(0,0,0,0.2)), #000 url(' + pageBanner.image_url + ') center center' }">
                    <div class="main_container position_relative">
                        <h2>Events</h2>
                    </div>
                </div>
                <div class="main_container">
                    <div class="row hidden-xs">
                        <div class="col-md-12">
                            <breadcrumb></breadcrumb>
                        </div>
                    </div>
                    <div  v-if="currentEvent">
                        <div class="row">
                            <div class="col-md-8">
                                <h4 class="event_name">{{ currentEvent.name }}</h4>
                                <p class="event_details_dates">
                                    <span v-if="isMultiDay(currentEvent)">{{ currentEvent.start_date | moment("MMMM D", timezone)}} - {{ currentEvent.end_date | moment("MMMM D", timezone)}}</span>
                                    <span v-else>{{ currentEvent.start_date | moment("MMMM D", timezone)}}</span>
                                </p>
                                <p class="event_details_dates">
                                    Location
                                </p>
                                <div class="event_desc event_details" v-html="currentEvent.rich_description"></div>
                            </div>
                            <div class="col-md-4">
                                <a :href="currentEvent.image_url" :data-lightbox="currentEvent.name">
                                    <img v-lazy="currentEvent.image_url" :alt="'Promotion: ' + currentEvent.name" class="margin_20 img_max"/>    
                                </a>
                            </div>
                        </div>
                        <div class="row margin_60">
                            <div class="col-md-12">
                                <div class="row margin_30">
                                    <div class="col-md-12">
                                        <router-link to="/events">
                    		                <div class="animated_btn pull-left">Back to Events</div>    
                    		            </router-link>    
                                    </div>
                                </div>
                                <social-sharing v-if="currentEvent" :url="shareURL(currentEvent.slug)" :title="currentEvent.name" :description="currentEvent.description" :quote="truncate(currentEvent.description)" :twitter-user="siteInfo.twitterHandle" :media="currentEvent.image_url" inline-template>
                                    <div class="social_share">
                                        <network network="facebook">
                                            <i class="fab fa-facebook"></i>
                                        </network>
                                        <!--<network network="twitter">-->
                                        <!--    <i class="fab fa-twitter"></i>-->
                                        <!--</network>-->
                                        <network network="email">
                                            <i class="fas fa-envelope"></i>
                                        </network>
                                    </div>
                                </social-sharing>
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
		return Vue.component("event-details-component", {
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
                    
					this.currentEvent = this.findEventBySlug(this.id);
					if (this.currentEvent === null || this.currentEvent === undefined) {
						this.$router.replace({ name: '404' });
					}
					else {
					    if (this.currentEvent.eventable_type === "Store"){
                            if (_.includes(this.currentEvent.event_image_url_abs, 'missing')) {
                                this.currentEvent.image_url = this.currentEvent.store.store_front_url_abs; 
                            }
                        } else {
                            if (_.includes(this.currentEvent.event_image_url_abs, 'missing')) {
                                this.currentEvent.image_url = "//codecloud.cdn.speedyrails.net/sites/5b8712636e6f641ebd220000/image/png/1529532187000/eventsplaceholder2@2x.png";    
                            }
                        }
					}
				// 	this.$breadcrumbs[1].meta.breadcrumb = this.currentEvent.name
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
                            this.currentEvent.store.store_front_url_abs = this.property.default_logo_url;
                        }
                        else if (this.currentEvent.store == null || this.currentEvent.store == undefined) {
                            this.currentEvent.store = {};
                            this.currentEvent.store.store_front_url_abs =  this.property.default_logo_url;
                        }
                        var vm = this;
                        var temp_event = [];
                        var current_id =_.toNumber(this.currentEvent.id);
                        _.forEach(this.currentEvent.store.event, function(value, key) {
                            if(_.toNumber(value) != current_id){
                                var current_event = vm.findEventById(value);
                                current_event.description_short = _.truncate(current_event.description, {'length': 70});
                                temp_event.push(current_event);
                            }
                        });
                        this.storeEvents = temp_event;
                    }
                    if(this.currentEvent.store) {
                        var storeHours = [];
                        var vm = this;
                        _.forEach(this.currentEvent.store.store_hours, function (value, key) {
                            var hour = vm.findHourById(value);
                            if(hour.day_of_week === 0){
                                hour.order = 7;
                            }
                            else {
                                hour.order = hour.day_of_week;
                            }
                            storeHours.push();
                        });
                        this.storeHours = _.sortBy(storeHours, [function(o) { return o.order; }]);;
                    }
                }
            },
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