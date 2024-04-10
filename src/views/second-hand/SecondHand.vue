<template>
  <div class="container">
    <!-- 菜单 -->
    <Row class="second-hand-menu">
      <iCol class="second-hand-category" span="15">
        <a
          is="sui-label"
          v-if="activeSecondHandCategory === '所有'"
          color="orange"
          @click="selectCategory('所有')"
        >
          所有
        </a>
        <a is="sui-label" v-else @click="selectCategory('所有')">所有</a>
        <template v-for="item in secondHandCategory">
          <a
            is="sui-label"
            v-if="activeSecondHandCategory === item.name"
            color="orange"
            @click="selectCategory(item.name)"
            >{{ item.name }}</a
          >
          <a is="sui-label" v-else @click="selectCategory(item.name)">{{
            item.name
          }}</a>
        </template>
      </iCol>
      <iCol span="6">
        <sui-input
          class="fluid"
          placeholder="搜好物"
          icon="search"
          v-model="param.general"
          @keydown.enter="getSecondHandPage(param)"
        />
      </iCol>
      <iCol class="second-hand-add" span="3">
        <sui-button
          fluid
          color="orange"
          content="发布信息"
          @click="secondHanAddModal.show = true"
        />
      </iCol>
    </Row>
    <!-- 菜单 END -->
    <!-- 二手列表 -->
    <div class="second-hand-list">
      <sui-card
        class="fluid second-hand-card"
        v-for="secondHand in secondHandPage.records"
        :key="secondHand.id"
      >
        <sui-image class="second-hand-image" :src="secondHand.images[0]" />
        <sui-card-content>
          <sui-card-header>
            <router-link
              class="second-hand-title"
              :to="'/second-hand/' + secondHand.id"
              >{{ secondHand.title }}
            </router-link>
          </sui-card-header>
          <sui-card-description class="second-hand-description">{{
            secondHand.description
          }}</sui-card-description>
          <span class="second-hand-price">￥{{ secondHand.price }}</span>
          <span slot="right">
            <a is="sui-label" basic @click="addZan(secondHand.resource)"
              >👍 {{ secondHand.resource.zanCount }}</a
            >
            <a
              is="sui-label"
              basic
              @click="getCommentList(secondHand.resource)"
            >
              💬 {{ secondHand.resource.commentCount }}
            </a>
          </span>
        </sui-card-content>
        <sui-card-content extra>
          <sui-image :src="secondHand.user.avatar" avatar />
          <span style="line-height: 28px">{{ secondHand.user.nickname }}</span>
          <span slot="right" style="line-height: 28px"
            ><Time :time="secondHand.createTime"
          /></span>
        </sui-card-content>
      </sui-card>
    </div>
    <!-- 二手列表 END -->
    <!-- 加载更多 -->
    <sui-button
      class="fluid"
      basic
      content="加载更多"
      :disabled="this.param.current >= this.secondHandPage.pages"
      @click="loadMore"
    />
    <!-- 加载更多 END -->
    <!-- 二手交易添加模态框 -->
    <Modal
      v-model="secondHanAddModal.show"
      title="发布闲置"
      width="600"
      scrollable
    >
      <Form :model="secondHanAddModal.form" :label-width="50">
        <FormItem label="类别" required>
          <Select
            v-model="secondHanAddModal.form.secondHandCategory"
            size="large"
          >
            <Option
              v-for="item in secondHandCategory"
              :key="item.name"
              :value="item.name"
              :label="item.name"
            />
          </Select>
        </FormItem>
        <FormItem label="标题" required>
          <Input
            v-model="secondHanAddModal.form.title"
            size="large"
            placeholder="清晰的标题能让更多人注意到"
          />
        </FormItem>
        <FormItem label="物品" required>
          <Input
            v-model="secondHanAddModal.form.name"
            size="large"
            placeholder="请填写物品名"
          />
        </FormItem>
        <FormItem label="价格" required>
          <InputNumber
            v-model="secondHanAddModal.form.price"
            :min="0"
            size="large"
            placeholder="请输入物品价格"
          />
        </FormItem>
        <FormItem label="地点" required>
          <Input
            v-model="secondHanAddModal.form.address"
            size="large"
            placeholder="请填写交易地点"
          />
        </FormItem>
        <FormItem label="描述" required>
          <Input
            v-model="secondHanAddModal.form.description"
            type="textarea"
            size="large"
            :autosize="{ minRows: 5, maxRows: 10 }"
            placeholder="输入描述信息，有助于他人了解"
          />
        </FormItem>
        <FormItem label="图片" style="margin-bottom: 0" required>
          <ImageUploader
            ref="imageUploader"
            @images-change="
              (images) => (secondHanAddModal.form.images = images)
            "
          />
        </FormItem>
      </Form>
      <template #footer>
        <Button type="text" @click="secondHanAddModal.show = false"
          >取消</Button
        >
        <Button @click="addSecondHand">确定发布</Button>
      </template>
    </Modal>
    <!-- 二手交易添加模态框 END -->
    <!-- 二手交易详情模态框 -->
    <Modal
      v-model="commentAddModal.show"
      title="物品详情"
      width="600"
      footer-hide
    >
      <Input
        ref="commentInput"
        v-model="commentAddModal.form.value"
        type="textarea"
        :rows="3"
        placeholder="添加评论"
        @on-enter="addComment"
      />
      <sui-comment-group class="comment-group">
        <sui-comment v-for="comment in commentList" :key="comment.id">
          <sui-comment-avatar :src="comment.postUser.avatar" />
          <sui-comment-content>
            <a is="sui-comment-author">{{ comment.postUser.nickname }}</a>
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
            <sui-comment v-for="child in comment.children" :key="child.id">
              <sui-comment-avatar :src="child.postUser.avatar" />
              <sui-comment-content>
                <a is="sui-comment-author">{{ child.postUser.nickname }}</a>
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
                    >回复</sui-comment-action
                  >
                </sui-comment-actions>
              </sui-comment-content>
            </sui-comment>
          </sui-comment-group>
          <!-- 子评论 END -->
        </sui-comment>
      </sui-comment-group>
    </Modal>
    <!-- 二手交易详情模态框 END -->
  </div>
</template>

<script>
import ImageUploader from "@/components/common/ImageUploader";

export default {
  name: "SecondHand",
  components: {
    ImageUploader,
  },
  data() {
    return {
      activeSecondHandCategory: "所有",
      secondHandCategory: [
        {
          name: "测试",
          code: "-1",
        },
      ],
      secondHanAddModal: {
        show: false,
        form: {
          secondHandCategory: "",
          title: "",
          name: "",
          price: 0,
          description: "",
          address: "",
          images: "",
        },
      },
      param: {
        // query 参数
        secondHandCategory: null,
        general: null,
        // page & order 参数
        desc: "create_time",
        current: 1,
      },
      secondHandPage: {
        records: [
          {
            id: 1,
            userId: 1,
            resourceId: 37,
            secondHandCategory: "数码",
            title: "小米手机9，低价转",
            name: "小米手机9",
            price: 599,
            address: "文瀛13#105",
            description: "买了华为P30，这部小米手机就低价转了，9.5成新",
            images: [
              "https://g-search2.alicdn.com/img/bao/uploaded/i4/i2/1714128138/O1CN018kA6uv29zFiGLMEsL_!!0-item_pic.jpg_250x250.jpg",
            ],
            createTime: "2024-05-23 15:56:27",
            user: {
              id: 1,
              username: "admin",
              nickname: "管理员",
              avatar: "http://img.angus-liu.cn/avatar/avatar07.png",
              sign: "一句话介绍自己",
            },
            resource: {
              id: 37,
              zanCount: 0,
              pvCount: 0,
              commentCount: 0,
            },
          },
        ],
        total: 20,
        size: 1,
        current: 1,
        searchCount: true,
        pages: 20,
      },
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
      commentAddModal: {
        show: false,
        form: {
          parentId: null,
          resourceId: null,
          replyUserId: null,
          value: "",
          content: "",
        },
      },
      currentResource: {},
    };
  },
  methods: {
    // 类别切换
    selectCategory(secondHandCategory) {
      this.activeSecondHandCategory = secondHandCategory;
      if (secondHandCategory === "所有") {
        this.param.secondHandCategory = null;
      } else {
        this.param.secondHandCategory = secondHandCategory;
      }
      this.param.current = 1;
      this.getSecondHandPage(this.param);
    },
    // 初始化二手交易添加模态框
    resetSecondHanAddModal() {
      this.secondHanAddModal = {
        show: false,
        form: {
          secondHandCategory: "",
          title: "",
          name: "",
          price: 0,
          description: "",
          address: "",
          images: "",
        },
      };
      this.$refs.imageUploader.clearImages();
    },
    // 获取二手交易类别
    getSecondHandCategory() {
      this.$axios
        .get("/api/dic/item", { dicTypeCode: "second-hand-category" })
        .then((res) => {
          let result = res.data;
          this.secondHandCategory = result.data;
        });
    },
    // 获取二手交易分页
    getSecondHandPage(param) {
      this.$axios.get("/api/second-hand/page", param).then((res) => {
        let result = res.data;
        this.secondHandPage = result.data;
        this.secondHandPage.records.forEach((secondHand) => {
          if (secondHand.images && secondHand.images.length > 0) {
            secondHand.images = JSON.parse(secondHand.images);
          }
        });
      });
    },
    // 添加二手交易
    addSecondHand() {
      let secondHandForm = this.secondHanAddModal.form;
      secondHandForm.images = JSON.stringify(secondHandForm.images);
      this.$axios.post("/api/second-hand", secondHandForm).then((res) => {
        let result = res.data;
        if (result.success) {
          this.$Notice.success({ title: "Bingo", desc: "发布成功" });
          this.resetSecondHanAddModal();
          this.getSecondHandPage(this.param);
        }
      });
    },
    // 加载更多
    loadMore() {
      this.param.current++;
      this.$axios.get("/api/second-hand/page", this.param).then((res) => {
        let result = res.data;
        let moreSecondHandPage = result.data;
        moreSecondHandPage.records.forEach((secondHand) => {
          if (secondHand.images && secondHand.images.length > 0) {
            secondHand.images = JSON.parse(secondHand.images);
          }
        });
        this.secondHandPage.records.push(...moreSecondHandPage.records);
      });
    },
    // 点赞
    addZan(resource) {
      this.$axios
        .post("/api/zan", { resourceId: resource.id })
        .then((res) => resource.zanCount++);
    },
    // 获取评论
    getCommentList(resource) {
      this.currentResource = resource;
      this.$axios.get(`/api/comment/${resource.id}`).then((res) => {
        let result = res.data;
        this.commentList = result.data;
        // 展示评论
        this.commentAddModal.show = true;
      });
    },
    // 初始化评论
    initComment() {
      this.commentAddModal.form = {
        resourceId: null,
        parentId: null,
        replyUserId: null,
        value: "",
        content: "",
      };
    },
    // 添加评论
    addComment() {
      // 设置资源 ID
      this.commentAddModal.form.resourceId = this.currentResource.id;
      // 判断是评论还是回复
      if (
        this.commentAddModal.form.value.startsWith("@") &&
        this.commentAddModal.form.replyUserId !== null
      ) {
        // 回复时去掉评论内容中的回复用户名
        let index = this.commentAddModal.form.value.indexOf(" ");
        this.commentAddModal.form.content =
          this.commentAddModal.form.value.substr(index + 1);
      } else {
        this.commentAddModal.form.parentId = null;
        this.commentAddModal.form.replyUserId = null;
        this.commentAddModal.form.content = this.commentAddModal.form.value;
      }
      this.$axios
        .post("/api/comment", this.commentAddModal.form)
        .then((res) => {
          this.initComment();
          this.currentResource.commentCount++;
          this.getCommentList(this.currentResource);
        });
    },
    // 回复评论
    replyComment(parentId, replyUser) {
      this.commentAddModal.form.parentId = parentId;
      this.commentAddModal.form.replyUserId = replyUser.id;
      this.commentAddModal.form.value = `@${replyUser.nickname} `;
      this.$refs.commentInput.focus();
    },
  },
  mounted() {
    this.getSecondHandCategory();
    this.getSecondHandPage(this.param);
  },
};
</script>

<style lang="less" scoped>
.container {
  width: 1200px;
  margin: 10px auto;
  padding: 5px;
  overflow-x: hidden;

  .ivu-col {
    padding: 5px;
  }

  .second-hand-message {
    margin-top: 7px;
  }

  .second-hand-menu {
    .second-hand-category {
      padding-left: 0;
      padding-bottom: 0;

      .label {
        padding: 12px 15px;
        font-size: 1em;
        margin: 0 7px 7px 0;
      }
    }

    .second-hand-add {
      padding-right: 0;

      .button {
        padding: 12px;
      }
    }
  }

  .second-hand-list {
    margin: 15px auto 0;
    padding: 1px;
    width: 1230px;
    overflow: auto;

    .second-hand-card {
      width: 216px;
      height: 373px;
      overflow: auto;
      margin: 0 27px 27px 0;
      float: left;

      .second-hand-image {
        width: 216px;
        height: 216px;
      }

      a.second-hand-title {
        color: #2f2f2f;
      }

      a.second-hand-title:hover {
        color: #ff8364;
      }
    }

    .header,
    .second-hand-description {
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }

    .header {
      font-size: 1em;
    }

    .second-hand-description {
      font-size: 0.9em;
      margin-bottom: 10px;
    }

    .second-hand-price {
      display: inline-block;
      color: red;
      font-size: 1.1em;
      padding-top: 5px;
    }
  }
}

.ivu-input-number {
  width: 100%;
}
</style>
