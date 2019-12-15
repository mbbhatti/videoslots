<template>
  <div class="container">
    <section v-if="error">
        <p>We're sorry, we're not able to retrieve this information at the moment, please try back later</p>
    </section>
    <section v-else>        
        <h1>{{ msg }}</h1>        
        <div v-if="loading">Loading...</div>        
        <input 
            type="text" 
            class="form-control" 
            v-model="search" 
            placeholder="Filter by author name"
        />        
        <div class="row">
            <div 
                class="column" 
                v-for="(photo, index) in filteredAuthor" 
                :key="index"
            >
                <img 
                  v-bind:src="photo.id | changeImageLink(url, imageWidth, imageHeight)" 
                  alt="Image"
                  v-on:click="modelBox(index)"
                  style="cursor:pointer;"
                />
                <div class="text">
                    <div class="text-left">{{photo.author}}</div>
                    <div class="text-right">#{{photo.id}}</div>
                </div>  
                <div class="modal hide" v-bind:id="`modal${index}`">
                    <span 
                        class="close" 
                        v-on:click="modelBox(index)"
                    >
                        &times;
                    </span>
                    <img 
                        v-bind:src="photo.download_url" 
                        class="modal-content" 
                    />
                    <div id="caption">#{{photo.id}}</div>
                </div>              
            </div>
        </div>
        <div v-if="photos" class="pagination">
            <button 
                class="btn prev" 
                rel="prev" 
                v-bind:data-page="prevPage" 
                v-on:click="paginate($event)"
            >
                Prev
            </button>
            <button class="btn">
                {{currentPage}}
            </button>
            <button class="btn next" 
                rel="next" 
                v-bind:data-page="nextPage" 
                v-on:click="paginate($event)"
            >
                Next
            </button>
        </div>
    </section>
</div>
</template>

<script>
import axios from 'axios'

export default {
    props: {
        msg: String
    },
    data() {
        return {
            photos: null,
            loading: true,
            error: false,
            prevPage: 1,
            nextPage: 0,
            currentPage: 1,
            limit: 30,
            url: process.env.VUE_APP_GALLERY_URL,
            imageWidth: 367,
            imageHeight: 267,
            search: ''
        }
    },
    filters: {
        changeImageLink(id, url, width, height) {            
            return url + 'id/' + id + '/'+width+'/'+height;
        }
    },
    methods: {
        getPhotoData: function(url) {
            axios.get(url)
                .then(response => {
                    this.photos = response.data;
                    const pageLinks = response.headers.link.split(',');
                    if (pageLinks.length == 1) {
                        if (pageLinks[0].includes("next")) {
                            this.nextPage = pageLinks[0].split(';')[0].match(/page=(\d*)/)[1];
                        } else if (pageLinks[0].includes("prev")) {
                            this.prevPage = pageLinks[0].split(';')[0].match(/page=(\d*)/)[1];
                        }
                    } else {
                        this.prevPage = pageLinks[0].split(';')[0].match(/page=(\d*)/)[1];
                        this.nextPage = pageLinks[1].split(';')[0].match(/page=(\d*)/)[1];
                    }

                }).catch(err => {
                    console.log(err);
                    this.error = true;
                })
                .finally(() => this.loading = false)
        },
        paginate: function(event) {
            const page = event.target.getAttribute('data-page');
            if (page < this.currentPage || page > this.currentPage) {
                const url = this.url + 'v2/list?page=' + page + '&limit=' + this.limit;
                this.getPhotoData(url);
            }
            this.currentPage = page;
        },
        modelBox: function(index) {
            const modelId = "modal"+index;
            const modal = document.getElementById(modelId);             
            if (modal.className.includes('hide')) {
                modal.classList.remove('hide');
                modal.classList.add('show');
            } else {
                modal.classList.remove('show');
                modal.classList.add('hide');
            }                     
        }        
    },
    mounted() {
        this.getPhotoData(this.url+'v2/list');        
    },
    computed: {
        filteredAuthor () {     
            if(this.search) {       
                return this.photos.filter(photo => {
                    return photo.author.toLowerCase().includes(this.search.toLowerCase())
                });
            } else {
                return this.photos;
            }
        }
    }
}
</script>

<style scoped lang="scss">
@import './../../src/assets/scss/style.scss';
</style>