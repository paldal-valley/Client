<template>
  <vue-board-container>
    <!-- sidebar -->
    <vue-board-sidebar
      :buttons="GET_QUESTION_META.sidebarButtons"
      :buttons-downside="GET_POST_META.sidebarButtonsDownside"/>

    <vue-post-container>
      <!-- buttons -->
      <vue-button-group :buttons="GET_POST_META.icon_buttons" />
      <!-- posts -->
      <div class="post-content">
        <vue-post
          :title="post.title"
          :reward ="post.reward"
          :content="post.content"
          :category="category"
          :user-name="post.userName"
          :user-email="post.userEmail"
          :created-date="post.createdDate"
          :view="post.view"
          :hasAnswerBtn=true
          :hasReward="true"
          :isSelected="selectStatus(postId_Q)"
        />
        <br>
        <h2> {{length}}개의 답변이 존재합니다. </h2>
        <br>
  <!-- 원래 hasAnswerBtn = true 였음 -->
        <vue-answer
          v-for="answer in answers"
          :key="answer.id"
          :content="answer.content"
          :user-name="answer.userName"
          :user-email="answer.userEmail"
          :created-date="answer.createdDate"
          :answerId="answer.id.toString()"
          :questionId="postId_Q.toString()"
          :postId="postId"
          :hasUpdateBtn="onGetAuthority(answer.userId)"
          :hasDeleteBtn="onGetAuthority(answer.userId)"
          :hasSelect="hasSelect(answer.postId_A)"
          @answers-changed="fetchPost"
          :hasSelectBtn="selectBtn(GET_USER.id)"

        />
        <!-- comments -->
        <vue-category-separator
          :category-name="commentText"/>

        <vue-comment-textarea
          @comment-created="fetchPost"/>

        <transition-group name="fade" tag="div">
          <vue-comment-card
            v-for="comment in post.comments"
            :key="comment.id"
            :comment="comment"
            @comments-changed="fetchPost"/>
        </transition-group>
      </div>
    </vue-post-container>
    <div
      v-if="GET_USER.id === post.userId"
      class="float-btn-group">
      <vue-float-btn
        class="float-btn"
        button-text="수정하기"
        @click="$router.push(`${$route.path}/edit`)"/>
      <vue-float-btn
        class="float-btn"
        color="red"
        button-text="삭제하기"
        @click="deletePost"/>
    </div>
  </vue-board-container>
</template>

<script>
// containers
import VueBoardContainer from '~/containers/board'
import VuePostContainer from '~/containers/post'

// components
import VueBoardSidebar from '~/components/each-page/post/sidebar'
import VuePost from '~/components/common/posts'
import VueFloatBtn from '~/components/common/buttons/float'
import VueAnswer from '~/components/common/answer'
import VueCommentCard from '~/components/common/cards/comment'
import VueCommentTextarea from '~/components/common/textareas/comment'
import VueCategorySeparator from '~/components/common/separators/category'
import VueButtonGroup from '~/components/common/buttons/with-icon/group'

import { mapGetters } from 'vuex'

export default {
  middleware: ['isLoggedIn'],
  components: {
    VuePostContainer,
    VueBoardContainer,
    VueBoardSidebar,
    VuePost,
    VueFloatBtn,
    VueAnswer,
    VueCommentCard,
    VueCommentTextarea,
    VueCategorySeparator,
    VueButtonGroup
  },
  computed: {
    ...mapGetters({
      GET_POST_META: 'page-meta/GET_POST_META',
      GET_QUESTION_META: 'page-meta/GET_QUESTION_META',
      GET_USER: 'auth/GET_USER'
    }),
    category() {
      return this.$categoryMapper('question', this.post.categoryId)
    },
    commentText() {
      if (typeof this.post.comments !== 'undefined') {
        return this.post.comments.length
          ? `${this.post.comments.length}개의 댓글이 존재합니다.`
          : `작성된 댓글이 없습니다.`
      }
    }
  },
  data: () => ({
    postId: '',
    postId_Q: '',
    length: '',
    post: {},
    answers: [],
    selectedItem: [],
    selected: '',
    selectedOne: '',
    isSelected: '',
    selectedId: 0,
    postId_A: '',
    gotSelected: ''
  }),
  mounted() {
    this.postId = this.$route.params.postId
     this.fetchPost().then(()=>{
      this.viewPost()
      this.getPostSelected()
    })


  },
  methods: {
    async getPostSelected() {
          try {
            
            const options = {
              url: `post/question/getSelected/${this.postId_Q}`,

              method: 'get',
              params: { postId_Q: this.postId_Q  }
            }
            const { data } = await this.$axios(options)
            // alert(this.answer.isSelected)
              // this.isSelected = data[0].isSelected
              
              this.gotSelected = 0;
              for(var i = 0; i<= this.selected; i++) {
                if(data[i].isSelected == 1)
                  this.gotSelected = 1;
              }



            // alert(data.isSelected)
      //만약 postid랑 로그인 된 아이디랑 같으면 noti 날리고 -1로 돌아가게 해야함.
      //if(this.GET_USER.id)
              // alert(this.selected)
            }
            catch (err){
              console.error(err)
            }
    },
    async fetchPost() {
      try {
        const options = {
          url: `post/question/${this.postId}`,
          method: 'get'
        }
        this.isSelected = 0
        const { data } = await this.$axios(options)
        this.post = data
        this.postId_Q = data.id

        try{
          const options2 = {
            url: `post/answer`,
            method: 'get',
            params: { postId_Q: this.postId_Q }
          }
          const ans = (await this.$axios(options2)).data
          this.answers = ans

          // this.postId_A = this.answers.id
          this.length = ans.length

          try {
            const options3 = {
              url: `post/answer/isSelected/${this.postId_Q}`,
              method: 'get',
              params: { postId_Q: this.postId_Q  }
            }
              const { data } = await this.$axios(options3)
              this.selectedItem = data
              this.selectedOne = this.selectedItem[0].id
              this.selected = this.selectedItem.length

              this.selectedId = this.selectedOne

              // alert(this.selected)
            }
            catch (err){
              console.error(err)
            }

        
        
        }catch (err) {
          console.error(err)
        }

        
      } catch (err) {
        console.error(err)
      }
    },
    async viewPost() {
      this.post.view++;
      const options = {
          url: `post/view/${this.postId}`,
          method: 'put',
          data: {
            view: this.post.view
          }
        }
        try{
          await this.$axios(options)
        }
        catch(err){
          console.error(err)
        }
    },
    async deletePost() {
      const options = {
        url: `post/${this.postId}`,
        method: 'delete'
      }

      try {
        if (confirm('포스트를 정말 삭제하시겠습니까?')) {
          await this.$axios(options)
          this.$notifySuccess('정상적으로 삭제되었습니다.')
          this.$router.back()
        }
      } catch (err) {
        console.error(err)
        this.$notifyError('에러가 발생했습니다.')
      }
    },
   onWriteClick() {
      const postId = this.$route.params.postId
      this.$router.push(`../${postId}/answer`)
      //this.$router.push(`../question/${routerid}`)
    },
   onGetAuthority(userId) {
     if(userId === this.GET_USER.id){
       //alert(this.GET_USER.id)
       return true;
     }
   },
   hasRewardBox(){
    //  alert(this.boardTitle)
    
     if(this.boardTitle == "qna"){
       return true
     }
   },

    selectBtn(loginId) {
        //요부분 API 추가해서 수정
      try {

            if( this.selected > 0){
              return false
            }
            if(loginId === this.post.userId){
                return true
             }

        }catch (err) {
         console.error(err)
        }



    },
    selectStatus(){


      //selected Item에 채택된 답변의 id가 들어감.
      if(this.gotSelected == 1){
        return true
      }
      else
        return false


    },
    hasSelect(ansId){

      if(ansId === this.selectedOne){
        return true
      }
      else{
        // alert(this.selectedOne)
        return false
      }
    }

  }
}
</script>

<style lang="scss" scoped>
  .float-btn-group {
    position: fixed;
    margin-left: 60%;
    margin-top: 10%;
    .float-btn {
      margin-bottom: 10px;
    }
  }
  .post-content {
    width: 100%;
    display: flex;
    flex-direction: column;
  }
  .fade-enter-active, .fade-leave-active {
    transition: opacity 1.5s;
  }
  .fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
    opacity: 0;
  }
</style>