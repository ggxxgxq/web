<template>
  <div class="container">
    <Row>
      <iCol span="18">
        <article>
          <!-- 正文 -->
          <sui-card class="fluid article-card">
            <sui-card-content>
              <h1 class="article-title">{{ article.title }}</h1>
              <p class="article-time">{{ article.createTime }}</p>
              <div
                class="markdown-body article-content"
                v-html="article.htmlContent"
              ></div>
            </sui-card-content>
          </sui-card>
          <!-- 正文 END -->
          <!-- 评论列表 -->
          <sui-card class="fluid comment-card">
            <sui-message attached="top"> 评论列表 </sui-message>
            <sui-card-content>
              <div class="article-comment">
                <!-- 评论框 -->
                <sui-form class="comment-input-form">
                  <sui-form-field>
                    <Input
                      ref="commentInput"
                      v-model="comment.value"
                      type="textarea"
                      :rows="3"
                      placeholder="添加评论"
                      @keydown.ctrl.enter.native="addComment"
                    />
                  </sui-form-field>
                  <sui-form-field>
                    <span
                      class="resource-info-item"
                      @click="addZan(article.resource)"
                    >
                      👍 {{ article.resource.zanCount }}
                    </span>
                    <span class="resource-info-item">
                      💬 {{ article.resource.commentCount }}
                    </span>
                    <sui-button
                      basic
                      floated="right"
                      content="评论(Ctrl+Enter)"
                      @click.prevent="addComment"
                    />
                  </sui-form-field>
                </sui-form>
                <!-- 评论框END -->
                <!-- 父评论 -->
                <sui-comment-group>
                  <sui-comment v-for="comment in commentList" :key="comment.id">
                    <sui-comment-avatar :src="comment.postUser.avatar" />
                    <sui-comment-content>
                      <a is="sui-comment-author">{{
                        comment.postUser.nickname
                      }}</a>
                      <sui-comment-metadata>
                        <Time :time="comment.createTime" />
                      </sui-comment-metadata>
                      <sui-comment-text>{{ comment.content }}</sui-comment-text>
                      <sui-comment-actions>
                        <sui-comment-action
                          @click="replyComment(comment.id, comment.postUser)"
                          >回复</sui-comment-action
                        >
                      </sui-comment-actions>
                    </sui-comment-content>
                    <!-- 子评论 -->
                    <sui-comment-group v-if="comment.children.length > 0">
                      <sui-comment
                        v-for="child in comment.children"
                        :key="child.id"
                      >
                        <sui-comment-avatar :src="child.postUser.avatar" />
                        <sui-comment-content>
                          <a is="sui-comment-author">{{
                            child.postUser.nickname
                          }}</a>
                          <sui-comment-metadata>
                            <Time :time="child.createTime" />
                          </sui-comment-metadata>
                          <sui-comment-text>
                            <a :href="child.replyUser.id"
                              >@{{ child.replyUser.nickname }}</a
                            >
                            {{ child.content }}
                          </sui-comment-text>
                          <sui-comment-actions>
                            <sui-comment-action
                              @click="replyComment(comment.id, child.postUser)"
                              >回复
                            </sui-comment-action>
                          </sui-comment-actions>
                        </sui-comment-content>
                      </sui-comment>
                    </sui-comment-group>
                    <!-- 子评论 END -->
                  </sui-comment>
                </sui-comment-group>
                <!-- 父评论 END -->
              </div>
            </sui-card-content>
          </sui-card>
          <!-- 评论列表 END -->
        </article>
      </iCol>
      <iCol span="6">
        <!-- 发布用户信息 -->
        <sui-card class="fluid user-info-card">
          <sui-image style="width: 100%" :src="article.user.avatar" />
          <sui-card-content>
            <sui-card-header>
              <router-link :to="'/user/' + article.user.id">
                {{ article.user.nickname }}
              </router-link>
            </sui-card-header>
            <sui-card-meta>{{ article.user.username }}</sui-card-meta>
            <sui-card-description>{{ article.user.sign }}</sui-card-description>
          </sui-card-content>
          <sui-card-content extra>
            <sui-icon name="user" />
            文章作者
          </sui-card-content>
        </sui-card>
        <!-- 发布用户信息 END -->
        <!-- 操作按钮 -->
        <div
          class="actions-buttons"
          v-if="$store.state.user && article.userId === $store.state.user.id"
        >
          <sui-button-group>
            <sui-button icon="pencil" content="编辑" basic positive fluid />
            <sui-button icon="delete" content="删除" basic negative fluid />
          </sui-button-group>
        </div>
        <!-- 操作按钮 END -->
        <!-- 文章信息 -->
        <sui-card class="fluid">
          <sui-message attached="top" :content="article.articleCategory" />
          <sui-card-content>
            <Tag
              type="dot"
              color="#ff8364"
              v-for="(tag, index) in article.tags"
              :key="index"
              >{{ tag }}</Tag
            >
          </sui-card-content>
        </sui-card>
        <!-- 文章信息 END -->
        <!-- 相关文章 -->
        <sui-card class="fluid">
          <sui-message attached="top" content="他的文章" />
          <sui-card-content> 暂无更多文章 </sui-card-content>
        </sui-card>
        <!-- 相关文章 END -->
      </iCol>
    </Row>
  </div>
</template>

<script>
export default {
  name: "Detail",
  props: ["id"],
  data() {
    return {
      // 文章信息
      article: {
        id: -1,
        userId: -1,
        resourceId: -1,
        articleCategory: "媒体文章",
        title: "测试文章",
        createTime: "2024-05-13",
        tags: ["测试标签1", "测试标签2"],
        markdownContent: "测试文章",
        htmlContent: "测试文章",
        user: {
          id: -1,
          username: "anonymous",
          nickname: "游客",
          avatar: require("../../assets/img/avatar/anonymous-avatar.jpg"),
          sex: "保密",
          sign: "一句话介绍自己",
        },
        resource: {
          id: -1,
          zanCount: 0,
          pvCount: 0,
          commentCount: 2,
        },
      },
      // 评论信息
      commentList: [
        {
          id: -1,
          content: "测试评论",
          createTime: "2024-05-21 20:15:39",
          postUser: {
            id: -1,
            username: "admin",
            nickname: "管理员",
            avatar: "http://img.angus-liu.cn/avatar/avatar07.png",
          },
          children: [
            {
              id: -2,
              content: "测试回复",
              createTime: "2024-05-21 20:15:46",
              postUser: {
                id: 1,
                username: "admin",
                nickname: "管理员",
                avatar: "http://img.angus-liu.cn/avatar/avatar07.png",
              },
              replyUser: {
                id: 1,
                username: "admin",
                nickname: "管理员",
                avatar: "http://img.angus-liu.cn/avatar/avatar07.png",
              },
            },
          ],
        },
      ],
      comment: {
        parentId: null,
        resourceId: null,
        replyUserId: null,
        value: "",
        content: "",
      },
    };
  },
  methods: {
    // 获取文章信息
    getArticle() {
      this.$axios.get(`/api/article/${this.id}`).then((res) => {
        let result = res.data;
        this.article = result.data;
        this.article.tags = JSON.parse(this.article.tags);
        // 获取评论信息
        this.getCommentList();
      });
    },
    // 获取评论信息
    getCommentList() {
      this.$axios.get(`/api/comment/${this.article.resourceId}`).then((res) => {
        let result = res.data;
        this.commentList = result.data;
      });
    },
    // 初始化评论
    initComment() {
      this.comment = {
        resourceId: null,
        parentId: null,
        replyUserId: null,
        value: "",
        content: "",
      };
    },
    // 添加评论
    addComment() {
      this.comment.resourceId = this.article.resourceId;
      // 判断是评论还是回复
      if (
        this.comment.value.startsWith("@") &&
        this.comment.replyUserId !== null
      ) {
        // 回复时去掉评论内容中的回复用户名
        let index = this.comment.value.indexOf(" ");
        this.comment.content = this.comment.value.substr(index + 1);
      } else {
        this.comment.parentId = null;
        this.comment.replyUserId = null;
        this.comment.content = this.comment.value;
      }
      this.$axios.post("/api/comment", this.comment).then((res) => {
        this.initComment();
        this.getCommentList();
        this.article.resource.commentCount++;
      });
    },
    // 回复评论
    replyComment(parentId, replyUser) {
      this.comment.parentId = parentId;
      this.comment.replyUserId = replyUser.id;
      this.comment.value = `@${replyUser.nickname} `;
      this.$refs.commentInput.focus();
    },
    // 点赞
    addZan(resource) {
      this.$axios
        .post("/api/zan", { resourceId: resource.id })
        .then((res) => resource.zanCount++);
    },
  },
  mounted() {
    this.getArticle();
  },
};
</script>

<style lang="less" scoped>
.container {
  width: 1140px;
  margin: 0 auto;
  padding: 5px;

  .ivu-col {
    padding: 7px;
  }
}

.article-card {
  margin-bottom: 10px;
  padding: 2em;

  .article-title {
    text-align: center;
    padding-bottom: 10px;
    border-bottom: 1px dashed #eee;
  }

  .article-time {
    text-align: center;
    color: #aaa;
  }
}

.user-info-card {
  .header {
    a {
      color: #373737;
    }
  }
}

.actions-buttons {
  text-align: center;

  .button {
    width: 134px;
  }
}

.comment-card {
  .article-comment {
    margin: 1em;

    .resource-info-item {
      margin-right: 15px;
      cursor: pointer;
    }

    .resource-info-item:hover {
      color: #ff8364;
    }

    .comment-input-form {
      overflow: hidden;
    }
  }
}
</style>
