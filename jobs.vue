<template>
    <div>
        hello
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