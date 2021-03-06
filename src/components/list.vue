<template lang="pug">
  #content
    mu-snackbar(v-if="snackbar", :message="message", action="登陆", @actionClick="login", @close="close_snackbar")
    mu-row(v-for="post in posts")
      mu-col(width="95", tablet="85", desktop="80", class="post")
        mu-card.card
          div(@click="read_more(post.number)")
            mu-card-title(:title="post.title", :subTitle="'Posted at '+ format_date(post.date) + format_comments_count(post)")
          mu-card-text(v-html="marked(init(post.body))", class="markdown-body text")
          .tags_more
            .tags
              mu-chip(v-for="tag in post.tags", :style="tag_color(tag)") {{tag.name}}
            mu-card-actions.actions
              mu-raised-button(icon="library_books", class="more", label="MORE", secondary, @click="read_more(post.number)")
</template>

<style lang="stylus">
@import '../assets/variables'

#content
  &
    display flex
    flex-flow column nowrap
    min-height calc(100vh - 196px - 64px)
  .post:first-child
    margin-top 20px
  .card
    &
      padding 20px
      border-radius 5px
    @media screen and (max-width: 700px)
      &
        padding: 0
        padding-bottom 10px
  .mu-card-title
    cursor pointer
    user-select none
  .mu-card-sub-title
    &::before
      font-family "Material Icons"
      content "\E916"
      display inline-block
      vertical-align top
    &
      padding-bottom 3px
      border-bottom 1px dashed #ccc
  .tags_more
    display flex
    flex-flow row nowrap
    justify-content space-between
    margin-left 16px
    margin-right 16p
  .tags
    &
      flex 1
      display flex
      flex-flow row nowrap
      align-items center
    .mu-chip
      &
        margin-right 1px
      &:hover
        animation pulse 1s infinite
  .text
    margin-bottom 60px
  .actions
    .mu-raised-button-secondary
      background-color $secondary_color
  .row
      justify-content: center
</style>

<script>
import { mapState } from 'vuex'
import jump from 'jump.js'
import { owner, repo, site_name } from '../const'
import Util from '../mixin'
import { Post } from '../model'
import { loginByGithub } from '../util'

export default {
  name: 'List',
  mixins: [Util],
  data() {
    return {
      snackbar: false,
      message: ''
    };
  },
  computed: mapState({
    posts: 'current_list'
  }),
  async created() {
    try {
      this.$Progress.start();
      await this.$store.dispatch('fetchList');
      this.$Progress.finish();
    } catch (e) {
      this.$Progress.fail();
      this.message = '加载出错，你可以尝试登陆后重试';
      this.snackbar = true;
    }
  },
  mounted() {
    document.title = site_name;
  },
  methods: {
    async login() {
      const code = await loginByGithub();
      await this.$store.dispatch('exchangeToken', code);
      await this.$store.dispatch('fetchList');
      this.snackbar = false;
    },
    read_more(number) {
      this.$router.push(`/post/${number}`);
      jump('#header', {
        duration: 300
      });
    },
    close_snackbar() {
      this.snackbar = false;
    },
    format_comments_count(post) {
      if(post.comments){
        return ` | ${post.comments} 条评论`;
      }else{
        return ' | 无评论';
      }
    }
  }
};
</script>
