<template>
  <v-container text-xs-center mt-5 pt-5>
    <!-- Add Post Title -->
    <v-layout row wrap>
      <v-flex xs12 sm6 offset-sm3>
        <h1 class="primary--text">Add Post</h1>
      </v-flex>
    </v-layout>

    <!-- Add Post Form -->
    <v-layout row wrap>
      <v-flex xs12 sm6 offset-sm3>
        <v-form v-model="isFormValid" lazy-validation ref="form" @submit.prevent="handleAddPost">
          <!-- Title Input -->
          <v-layout row>
            <v-flex xs12>
              <v-text-field
                :rules="titleRules"
                v-model="title"
                label="Post Title"
                type="text"
                required
              ></v-text-field>
            </v-flex>
          </v-layout>

          <!-- Image Url Input -->
          <v-layout row>
            <v-flex xs12>
              <v-text-field
                :rules="imageRules"
                v-model="imageUrl"
                label="Image URL"
                type="text"
                required
              ></v-text-field>
            </v-flex>
            <v-btn
              @click="loadInstagramFeed(user)"
              color="info"
              floating
              fab
              small
              dark
              v-if="user.instagram"
            >
              <v-icon>instagram</v-icon>
            </v-btn>
          </v-layout>

          <!-- Image Preview -->
          <v-layout row>
            <v-flex xs12>
              <img :src="imageUrl" height="300px" />
            </v-flex>
          </v-layout>

          <!-- Categories Select -->
          <v-layout row>
            <v-flex xs12>
              <v-select
                v-model="categories"
                :rules="categoriesRules"
                :items="['Art', 'Education', 'Food', 'Furniture', 'Travel', 'Photography', 'Technology']"
                multiple
                label="Categories"
              ></v-select>
            </v-flex>
          </v-layout>

          <!-- Description Text Area -->
          <v-layout row>
            <v-flex xs12>
              <v-textarea
                :rules="descRules"
                v-model="description"
                label="Description"
                type="text"
                required
              ></v-textarea>
            </v-flex>
          </v-layout>

          <v-layout row>
            <v-flex xs12>
              <v-btn
                :loading="loading"
                :disabled="!isFormValid || loading"
                color="info"
                type="submit"
              >
                <span slot="loader" class="custom-loader">
                  <v-icon light>cached</v-icon>
                </span>
                Submit
              </v-btn>
            </v-flex>
          </v-layout>
        </v-form>
      </v-flex>
    </v-layout>

    <!-- Instagram feed -->
     <v-dialog xs12 sm6 offset-sm3 persistent v-model="instagramFeedDialog">
      <v-card>
        <v-card-title class="headline grey lighten-2">Instagram feed</v-card-title>
        <v-container>
          <v-form ref="form">
            <v-container class="mt-3">
              <v-flex xs12>
                <h2 class="font-weight-light">
                  Your Posts
                  <span class="font-weight-regular">({{instagramPosts.length}})</span>
                </h2>
              </v-flex>
              <v-layout row wrap>
                <v-flex xs12 sm6 v-for="post in instagramPosts" :key="post.id">
                  <v-card class="mt-3 ml-1 mr-2" hover>
                    <v-btn @click="handleUseInstagramPost(post)" color="info" floating fab small dark>
                      <v-icon>check</v-icon>
                    </v-btn>
                    <v-img height="30vh" :src="post.picture.url"></v-img>
                    <v-card-text>{{formatTitle(post.captionText)}}</v-card-text>
                  </v-card>
                </v-flex>
              </v-layout>
            </v-container>

            <v-divider></v-divider>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn class="error--text" flat @click="instagramFeedDialog = false">Cancel</v-btn>
            </v-card-actions>
          </v-form>
        </v-container>
      </v-card>
    </v-dialog> 
  </v-container>
</template>

<script>
import { mapGetters } from "vuex";
import * as instagram from "user-instagram";

export default {
  name: "AddPost",
  data() {
    return {
      isFormValid: true,
      title: "",
      imageUrl: "",
      categories: [],
      description: "",
      titleRules: [
        title => !!title || "Title is required",
        title => title.length < 20 || "Title must have less than 20 characters"
      ],
      imageRules: [image => !!image || "Image is required"],
      categoriesRules: [
        categories =>
          categories.length >= 1 || "At least one category is required"
      ],
      descRules: [
        desc => !!desc || "Description is required",
        desc =>
          desc.length < 200 || "Description must have less than 200 characters"
      ],
      instagramFeedDialog: false,
      instagramPosts: []
    };
  },
  computed: {
    ...mapGetters(["loading", "user"])
  },
  created() {      
  },
  methods: {
    formatTitle(title) {
      return title.length > 10 ? `${title.slice(0, 10)}...` : title;
    },
    handleGetInstagramPosts() {
      instagram(`https://www.instagram.com/${this.user.instagram}`)
          .then(data => {
            this.instagramPosts = data.posts;
        })
    },
    handleAddPost() {
      if (this.$refs.form.validate()) {
        this.$store.dispatch("addPost", {
          title: this.title,
          imageUrl: this.imageUrl,
          categories: this.categories,
          description: this.description,
          creatorId: this.user._id
        });
        this.$router.push("/");
      }
    },
    loadInstagramFeed({ _id, username, instagram }, instagramFeedDialog = true) {
      this.handleGetInstagramPosts();
      this.instagramFeedDialog = instagramFeedDialog;
    },
    handleUseInstagramPost(post) {
      this.instagramFeedDialog = false;
      this.imageUrl = post.picture.url;
      this.title = this.formatTitle(post.captionText);
    }
  }
};
</script>
