<template>
  <div>
    <div class="blog-list-wrapper">
      <div class="blog-intro-wrapper">
        <router-link to="/blog">
          <p class="coral-Detail read-more read-more-left coral-Detail--XL m-t-32 coral-Link">
            All Posts
          </p>
        </router-link>

        <h3 class="coral-Heading--XXL coral-Heading--heavy">{{ blogPost.title }}</h3>

        <p class="coral-Body--XL">{{ blogPost.excerpt }}</p>
        
        <hr class="coral-Divider--S m-t-24 m-b-8">

        <p class="coral-Detail coral-Detail--L opacity-80 m-b-32 m-l-8">
          <span class="coral-Detail--light">
            {{ getDate(blogPost.published_at) }}
          </span>
          <span class="m-l-4 m-r-4">•</span>
          <span class="coral-Detail--light">
            {{ blogPost.reading_time }} Min Read
          </span>
        </p>
        
        <div class="single-ad mobile-ad p-b-8">
          <NativeAd
            :adPosition="'Blog post top desktop'"
            :sponsored="true"
            :fullWidth="false"
            :key="$route.fullPath + 'ad'"
            adId="CWYD65Q7"
            zoneKey="CWYD65Q7"
          />
        </div>

        <figure class="post-full-image">
          <img :src="blogPost.feature_image"/>
        </figure>

      </div>

      <div class="blog-post-wrapper post-full-content" v-html="blogPost.html"> </div>

      <section class="p-t-32 m-b-48 p-b-40">
        
        <div class="">
          <hr class="coral-Divider--S">
        </div>

        <div class="">
          <hr class="coral-Divider--S">
        </div>

      </section>
    
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import { mapActions, mapGetters } from 'vuex';
import H3Description from '@/components/H3Description.vue'
import NativeAd from '@/components/NativeAd.vue'

import { getBlogPost } from '@/api/posts';
import localPosts from '@/api/posts.json';

import {
    getStrapiData,
  } from '@/api/strapi';

let name = "get"

export default {
  name: 'License',

  components: {
    H3Description,
    NativeAd
  },

  data: function(){
    return {
      blogPost: localPosts,
    }
  },

  mounted: async function(){

    const parent = this;

    let routerName = this.$router.currentRoute.value.params.post
    let storeBlogData = parent.$store.state.blogPosts
    let localBlogData = parent.$store.state.localPosts
    // console.log(localBlogData);

    let licenseData = await getStrapiData('licenses');
    console.log(licenseData);
    
    // Check if blog data has already been fetched, if not, fetch only the blog required
    if (storeBlogData.length == undefined) {
      
      //  Botched together to get local blog data while real blog is loading. Temporary fix, this will need to be server side rendered.
      for(let post in Object.keys(localBlogData)){
        try {
          if (localBlogData[post].slug == routerName) {
            // console.log(localBlogData[post].html);
            parent.blogPost = localBlogData[post];
          }
        } catch (error) {
          }
      }

      const blogPost = await getBlogPost(routerName);
      // If the blog post requested does not exists, redirect user to main blog page
      if (blogPost == undefined) { 
        parent.$router.push('/blog')
      }
      
      parent.blogPost = blogPost;

    } else{
      for(let post in Object.keys(storeBlogData)){
        try {
          if (storeBlogData[post].slug == routerName) {
            parent.blogPost = storeBlogData[post];
          }
        } catch (error) {
        }
      }
    }

  },

  methods:{
    ...mapActions(['adClick',]),

    getDate(dateString){
      // var date = dateString;
      let date = new Date(dateString);
      var monthDate = date.getDate();
      var year = date.getUTCFullYear();

      let weekDay = date.getUTCDay()
      switch (weekDay) {
        case 0:
          weekDay = "Monday"
          break;
        case 1:
          weekDay = "Tuesday"
          break;
        case 2:
          weekDay = "Wendesday"
          break;
        case 3:
          weekDay = "Thursday"
          break;
        case 4:
          weekDay = "Friday"
          break;
        case 5:
          weekDay = "Saturday"
          break;
        case 6:
          weekDay = "Sunday"
          break;
          
        default:
          break;
      }
      
      let month = date.getUTCMonth();
      switch (month) {
        case 0:
          month = "Jan"
          break;
        case 1:
          month = "Feb"
          break;
        case 2:
          month = "March"
          break;
        case 3:
          month = "Apr"
          break;
        case 4:
          month = "May"
          break;
        case 5:
          month = "Jun"
          break;
        case 6:
          month = "Jul"
          break;
        case 7:
          month = "Aug"
          break;
        case 8:
          month = "Sep"
          break;
        case 9:
          month = "Oct"
          break;
        case 10:
          month = "Nov"
          break;
        case 11:
          month = "Dec"
          break;
          
        default:
          break;
      }
      // var date = new Date(dateString);
      return month + " " + monthDate + ', ' + year
    }
  },

  metaInfo() {   
    return {
      title: this.blogPost.title,
      description: this.blogPost.excerpt,
      titleTemplate: '%s | macOSicons',
      image: this.blogPost.feature_image,
      meta:[
        // Facebook
        {
          property: 'og:url',
          content:  "https://macosicons.com" + this.$router.currentRoute.value.path
        },
        {
          property: 'og:title',
          content:  this.blogPost.title,
        },
        {
          property: 'og:description',
          content:  this.blogPost.excerpt,
        },
        {
          property: 'og:image',
          content:  this.blogPost.feature_image
        },

        // Twitter
        {
          property: 'twitter:url',
          content:  "https://macosicons.com" + this.$router.currentRoute.value.path
        },
        {
          property: 'twitter:title',
          content:  this.blogPost.title,
        },
        {
          property: 'twitter:description',
          content: this.blogPost.excerpt,
        },
        {
          property: 'twitter:image',
          content:  this.blogPost.feature_image
        },
      ]
    }
  },
  
  computed:{
  }
}
</script>

<style lang="less">
  @import '@/CSS/blog.less';
</style>