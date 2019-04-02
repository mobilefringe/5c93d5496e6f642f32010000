<template>
    <div> <!-- for some reason if you do not put an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_page_header" v-if="pageBanner" v-bind:style="{ background: 'linear-gradient(0deg, rgba(0,0,0,0.2), rgba(0,0,0,0.2)), #000 url(' + pageBanner.image_url + ') center center' }">
                    <div class="main_container position_relative">
                        <h2>Leasing</h2>
                    </div>
                </div>
                <div class="main_container">
                    <div class="row">
                        <div class="col-md-12">
                            <breadcrumb></breadcrumb>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-md-12">
                            <h2>Thank you!</h2>
                            <p>Your subscription has been confirmed. You've been added to our list and will hear from us soon.</p>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>
<style>
    .row .col-md-12 h2{
        font-size: 16px;
        line-height: 1.5rem;
        margin:auto;
        color: #195573;
    }
</style>

<script>
	define(["Vue", "vuex", "json!site.json"], function(Vue, Vuex, Site) {
		return Vue.component("location-component", {
            template: template, // the variable template will be injected
            data: function () {
                return {
                    dataLoaded: false,
                    main: null,
                    leasingInfo: null,
                    leasingBooklet: null,
                    pageImages: null
                }
            },
            created() {
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Newsletter Banner');
                    if(temp_repo != null && temp_repo != undefined) {
                        this.pageBanner = temp_repo.images[0];
                    } else {
                        this.pageBanner = {
                            "image_url": "//codecloud.cdn.speedyrails.net/sites/5c93d5496e6f642f32010000/image/png/1553624485505/creekside_banner.png"
                        }
                    }
                    this.dataLoaded = true;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'repos',
                    'findRepoByName'
                ])
            },
        });
	});
</script>