<template>
<!-- <h1> Hello {{this.$route.params.id}} </h1> -->
 <div class="write-container">
   <v-card>
     <v-form>
       <vue-post
         :title="post.title"
         :reward ="post.reward"
         :content="post.content"
         :category="category"
         :user-name="post.userName"
         :user-email="post.userEmail"
         :created-date="post.createdDate"
         :hasReward=true
         :hasLike="false"
       />
        <br><br>
       <v-textarea
         v-model="answer.content"
         full-width
         rows="15"/>
     </v-form>
   </v-card>
   <v-btn
     block
     color="#4E98A4"
     class="white--text"
     @click="updateAnswer">
     <strong>답변 수정</strong>
   </v-btn>
 </div>
</template>

<script>
import { mapGetters, mapState } from 'vuex'
import VuePost from '~/components/common/posts'
import VueAnswer from '~/components/common/answer'
export default {
 components: {
   VuePost,
   VueAnswer,
 },
 data: () => ({
   postId: '',
   postId_Q: '',
   post: {},
   answer: {}
 }),
 props: {
   titlePlaceHolder: {
     type: String,
     default: '제목'
   }
 },
 computed: {
   ...mapGetters({
     GET_USER: 'auth/GET_USER',
     GET_CATEGORIES: 'models/GET_CATEGORIES'
   }),
   category() {
      return this.$categoryMapper('question', this.post.categoryId)
   }
 },
 mounted() {
   this.postId = this.$route.params.postId
   this.answerId = this.$route.params.id
   this.fetchPost()
 },
 methods: {
    async fetchPost() {
     try {
       const options = {
         url: `post/question/${this.postId}`,
         method: 'get'
       }
       const { data } = await this.$axios(options)
       this.post = data
       this.postId_Q = data.id
        try{
          const options2 = {
            url: `post/answer/${this.answerId}`,
            method: 'get',
          }
          const ans = (await this.$axios(options2)).data
          this.answer = ans

        }catch (err) {
          console.error(err)
        }
       } catch (err) {
       console.error(err)
     }
   },
   async updateAnswer() {

     //alert(this.answer.content)
     try {
       const options = {
         url: `post/answer/${this.answerId}`,
         method: 'put',
         params: { postId_Q: this.postId_Q },
         data: {
           content: this.answer.content,
           userId: this.GET_USER.id
         }
       }
       //alert(this.postId_Q)
       if(this.answer.content) {
          await this.$axios(options)
          this.$router.back()         
       } else {
        this.$notifyWarning('게시글의 글을 작성해주세요.')
       }
       //alert(this.postId_Q)
       // await this.getReward("0x98FE5eaFd3D61af18fB2b2322b8346dF05057202")
     } catch (err) {
       this.$notifyError('올바르지 않는 수정입니다.')
       console.error(err)
     }
   },
 }
}
</script>

<style scoped>
 .separator {
   margin-top: -30px;
   margin-bottom: 10px;
 }
 .write-container {
   margin-bottom: 20px;
 }
</style>
