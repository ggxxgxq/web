<template>
  <div class="container">
    <Row>
      <iCol span="14">
        <!-- 轮播图 -->
        <Carousel
          class="carousel-container"
          loop
          autoplay
          :autoplay-speed="5000"
          radius-dot
        >
          <CarouselItem v-for="(item, index) in carouselList" :key="index">
            <div class="carousel-item">
              <a href="http://cst.nuc.edu.cn/" target="_blank">
                <img class="carousel-img" :src="item" />
              </a>
            </div>
          </CarouselItem>
        </Carousel>
        <!-- 轮播图 END -->
        <!-- 文章选项菜单 -->
        <!-- <sui-menu secondary>
          <sui-menu-item
            link
            v-for="item in menu.items"
            :key="item"
            :content="item"
            :active="item === menu.active"
            @click="select(item)"
          />
        </sui-menu> -->
        <!-- 文章选项菜单 END -->
        <!-- 文章列表 -->
        <sui-card class="fluid">
          <sui-card-content>
            <ul class="article-list">
              <li
                class="article-item-container"
                v-for="article in articlePage.records"
                :key="article.id"
              >
                <router-link
                  class="article-item"
                  :to="'/article/' + article.id"
                >
                  <!--<a class="article-title" :href="'/article/' + article.id">{{article.title}}</a>-->
                  <div class="article-info">
                    <div class="article-title">{{ article.title }}</div>
                    <div class="article-tool">
                      <div class="author">{{ article.author.name }}</div>
                      <div class="comment">
                        {{ article.comment_count }} 评论
                      </div>
                      <div class="date">{{ article.date }}</div>
                    </div>
                  </div>
                  <div class="article-img">
                    <img :src="article.img" />
                  </div>
                </router-link>
              </li>
            </ul>
          </sui-card-content>
          <sui-card-content extra>
            <template #right>
              <router-link to="/article"> 查看更多 </router-link>
            </template>
          </sui-card-content>
        </sui-card>
        <!-- 文章列表 END -->
      </iCol>
      <iCol span="10">
        <!-- 系统通告 -->
        <!-- 系统通告 END -->
        <!-- 文章类别 -->
        <sui-card class="fluid">
          <sui-message attached="top">
            <sui-message-header>
              <sui-icon name="bookmark" color="teal" />
              热榜
            </sui-message-header>
          </sui-message>
          <sui-card-content>
            <div class="article-category">
              <ol
                basic
                image
                v-for="(item, index) in articleCategory"
                :key="index"
                style="margin: 5px 10px 5px 10px"
              >
                <router-link :to="'/article/' + item.id"
                  >{{ index + 1 }}. {{ item.name }}</router-link
                >
                <!-- <a href="">{{ index + 1 }}. {{ item.name }}</a> -->
              </ol>
            </div>
          </sui-card-content>
        </sui-card>
        <!-- 文章类别 END-->
        <!-- 热门标签 -->
        <!-- <sui-card class="fluid">
          <sui-message attached="top">
            <sui-message-header>
              <sui-icon name="tags" color="orange" />
              热门标签
            </sui-message-header>
          </sui-message>
          <sui-card-content>
            <div class="article-hot-tag">
              <Tag
                color="orange"
                v-for="(tag, index) in hotTagList"
                :key="index"
                @click.native="toSearch(tag.name)"
              >
                {{ tag.name }}-{{ tag.count }}
              </Tag>
            </div>
          </sui-card-content>
        </sui-card> -->
        <sui-card class="fluid">
          <sui-message attached="top">
            <sui-message-header>
              <sui-icon name="tags" color="orange" />
              热门视频
            </sui-message-header>
          </sui-message>
          <sui-card-content>
            <div class="hot-video" v-for="item in hotVideoList" :key="item.id">
              <div class="video-cover">
                <img :src="item.img" :alt="item.title" />
                <div class="video-duration">{{ item.video.duration }}</div>
              </div>
              <div class="video-info">
                <div class="title">{{ item.title }}</div>
                <div class="like">{{ item.like }}</div>
                <div class="tool">
                  <span>{{ item.view_count }} 次观看</span>
                  <span class="author">{{ item.auth }}</span>
                </div>
              </div>
            </div>
          </sui-card-content>
        </sui-card>
        <!-- 热门标签 END -->
        <!-- 最新评论 -->
        <!-- 最新评论 END -->
      </iCol>
    </Row>
  </div>
</template>

<script>
export default {
  name: "Main",
  data() {
    return {
      menu: {
        active: "最新",
        items: ["最新", "最热"],
      },
      carouselList: [
        "https://q6.itc.cn/c_lfill,w_640,h_426,g_face/images03/20240409/a04ac43e1dfc44b89b0575749d4f3a7b.jpeg",
        "https://q4.itc.cn/c_lfill,w_640,h_426,g_face/images03/20240409/b71e3806e8f94c8692b3457a28495a96.jpeg",
        "https://q2.itc.cn/c_lfill,w_640,h_426,g_face/images03/20240409/88c352d205954fc082d6e0fe265ad517.jpeg",
        "https://q6.itc.cn/c_lfill,w_640,h_426,g_face/images03/20240409/a04ac43e1dfc44b89b0575749d4f3a7b.jpeg",
        "https://q2.itc.cn/c_lfill,w_640,h_426,g_face/images03/20240409/88c352d205954fc082d6e0fe265ad517.jpeg",
        "https://q5.itc.cn/q_70/images03/20240409/4a8fa7ba0ad94be9a2224926aabc8250.png",
        "https://q4.itc.cn/q_70/images03/20240409/aa222094a521464aadc471d7d2550807.jpeg",
      ],
      param: {
        // page & order 参数
        desc: "create_time",
        size: 20,
      },
      articlePage: {
        records: [
          {
            id: -1,
            comment_count: 2139,
            author: {
              name: "红星新闻",
            },
            title:
              "中国气象局：预计4-5月厄尔尼诺事件结束，夏季可能进入拉尼娜状态",
            date: "7天前",
            img: "https://p3-sign.toutiaoimg.com/tos-cn-i-tjoges91tu/f8a53bd045625d8c9d14e8d9003b114c~tplv-tt-cs0:524:345.jpg?_iz=31826&from=article.headline&lk3s=06827d14&x-expires=1715270881&x-signature=DEF5kaJ1AX7soCkqXf3Gpo%2BwkB8%3D",
          },
          {
            id: 1,
            comment_count: 2139,
            author: {
              name: "中国新闻周刊",
            },
            date: "7天前",
            title: "超800人死伤，台湾地震威力相当于32颗原子弹",
            img: "https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/9f8146b651684b9dbb5cfd8f99f83a39~tplv-tt-cs0:888:500.jpg?_iz=31826&from=article.headline&lk3s=06827d14&x-expires=1715270881&x-signature=Qn4jIAtUq5tG8BwSDDaUuZG9r%2BM%3D",
          },
        ],
        total: 7,
        size: 10,
        current: 1,
        searchCount: true,
        pages: 1,
      },

      articleLatestComment: [
        {
          articleId: "-1",
          articleTitle: "测试文章",
          userId: 1,
          userNickname: "管理员",
          userAvatar: "http://img.angus-liu.cn/avatar/avatar07.png",
          commentId: "-1",
          commentContent: "测试回复",
          commentCreateTime: "2024-05-22 09:57:25",
        },
      ],
      // 文章类别
      articleCategory: [
        {
          name: "习近平向中巴两党理论研讨会致贺信",
          id: 3,
        },
        {
          name: "网易回应儿子充值游戏父亲自扇耳光",
          id: 3,
        },
        {
          name: "美丽中国的经济热力",
          id: 3,
        },
        {
          name: "“老干部举报县领导”事件22人被处理",
          id: 3,
        },
        {
          name: "央媒评婚丧办酒要报备：脱离法治",
          id: 3,
        },
        {
          name: "云南香格里拉市发生4.7级地震",
          id: 3,
        },
        {
          name: "百亿级公司天瑞水泥闪崩跌超99%",
          id: 3,
        },
        {
          name: "辞职后旅居大理女生称想回家",
          id: 3,
        },
      ],
      hotTagList: [
        {
          name: "中北奖章",
          count: 1,
        },
        {
          name: "学院新闻",
          count: 1,
        },
        {
          name: "校长奖章",
          count: 1,
        },
      ],
      hotVideoList: [
        {
          id: 1,
          title: "香港女千里求爱，追到江西农村，穷小子艳福不浅",
          img: "https://p3-sign.toutiaoimg.com/tos-cn-i-pk90l89vgd/f91fa4c62d1d4fadb31252f427e8a7e4~tplv-pk90l89vgd-crop-center-v4:576:324.jpeg?_iz=31127&from=ttvideo.headline&lk3s=06827d14&x-expires=1713283683&x-signature=fyfuSa42x94XMxKk3Tm1vH8jQvM%3D",
          view_count: 173,
          auth: "海星大魔王",
          like: "1万点赞",
          video: {
            duration: "12:11",
            number: "1",
          },
        },
        {
          id: 2,
          title: "美国老丈人第一次做客中国家庭，被满满一桌子菜惊到：太丰盛了！",
          img: "https://p26-sign.toutiaoimg.com/tos-cn-i-pk90l89vgd/9dede7d522604a078473f1ff71683915~tplv-pk90l89vgd-crop-center-v4:576:324.jpeg?_iz=31127&from=ttvideo.headline&lk3s=06827d14&x-expires=1713283683&x-signature=id%2F20abJcvqt4FlLwMlfttWm3zI%3D",
          view_count: 1288,
          auth: "田纳西Jay和Ari（3号早8点春茶专场",
          like: "3万点赞",
          video: {
            duration: "09:20",
            number: "2",
          },
        },
        {
          id: 3,
          title: "美国花百亿对付的水葫芦，为什么被中国一招制服！用了什么办法？",
          img: "https://p3-sign.toutiaoimg.com/tos-cn-i-0026/oEcBAELCjIAOtAhZv76nQbAtKgDZXCnAmEepAe~tplv-pk90l89vgd-crop-center-v4:576:324.jpeg?_iz=31127&from=ttvideo.headline&lk3s=06827d14&x-expires=1713283683&x-signature=AHoA6EWJj%2Fk9AuXwMpmhtThh2QU%3D",
          view_count: 173,
          auth: "科学大魔王",
          like: "1万点赞",
          video: {
            duration: "03:49",
            number: "3",
          },
        },
      ],
    };
  },
  methods: {
    select(item) {
      this.menu.active = item;
      if (item === "最新") {
        this.param.desc = "create_time";
      } else {
        this.param.desc = "comment_count,zan_count";
      }
      this.getArticlePage(this.param);
    },
    // 获取文章分页
    getArticlePage(param) {
      this.$axios.get("/api/article/page", param).then((res) => {
        let result = res.data;
        this.articlePage = result.data;
      });
    },
    // 获取最新评论
    getArticleLatestComment() {
      this.$axios.get("/api/article/latest-comment").then((res) => {
        let result = res.data;
        this.articleLatestComment = result.data;
      });
    },
    // 获取热门标签
    getHotTag() {
      this.$axios.get("/api/article/hot-tag").then((res) => {
        let result = res.data;
        this.hotTagList = result.data;
      });
    },
    toSearch(q) {
      this.$router.push({ name: "search", query: { q } });
    },
  },
  mounted() {
    this.getArticlePage(this.param);
    this.getArticleLatestComment();
    this.getHotTag();
  },
};
</script>

<style lang="less" scoped>
.container {
  width: 1200px;
  margin: 10px auto;

  .ivu-col {
    padding: 7px;
  }

  .carousel-container {
    margin-bottom: 15px;
    box-shadow: 0 1px 2px 0 rgba(34, 36, 38, 0.15);

    .carousel-item {
      width: 100%;
      height: 400px;

      .carousel-img {
        min-height: 400px;
        width: 100%;
      }
    }
  }

  .article-list {
    margin: 10px 5px;
    list-style: none;

    .article-item {
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 16px 0;
      &:first-child {
        padding-top: 0;
      }
      .article-info {
        height: 104px;
        position: relative;
        padding-bottom: 20px;
        .article-tool {
          position: absolute;
          bottom: 0;
          .author,
          .comment,
          .date {
            margin-right: 20px;
            color: #5c5c5c;
          }
          display: flex;
          align-items: center;
        }
      }
      .article-img {
        width: 160px;
        height: 104px;
        margin-left: 20px;
        img {
          width: 100%;
          height: 100%;
        }
      }

      .article-title {
        display: inline-block;
        max-width: 400px;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
        font-size: 1.1em;
        color: #5c5c5c;
      }

      &:hover {
        cursor: pointer;
        .article-title {
          color: #ff8364;
        }
      }

      .article-time {
        color: #aaa;
        float: right;
      }
    }
  }

  .hot-video {
    display: flex;
    padding: 16px 0;
    cursor: pointer;
    .video-cover {
      width: 184px;
      height: 96px;
      margin-right: 20px;
      flex: 0 0 auto;
      position: relative;
      img {
        width: 100%;
        height: 100%;
      }
      .video-duration {
        position: absolute;
        font-size: 12px;
        font-weight: 400;
        background: rgba(0, 0, 0, 0.5);
        color: #fff;
        padding: 4px;
        right: 6px;
        bottom: 6px;
      }
    }
    .video-info {
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      overflow: hidden;

      .title {
        font-weight: bolder;
        overflow-wrap: break-word;
        display: -webkit-box;
        -webkit-line-clamp: 2;
        -webkit-box-orient: vertical;
      }
      .tool {
        display: flex;
        align-items: center;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
        color: #999;
        .author {
          margin-left: 10px;
        }
      }
      .like {
        font-size: 10px;
        line-height: 12px;
        color: #ff403a;
        background-color: rgba(255, 64, 58, 0.1);
        border-radius: 5px;
        padding: 1px 4px;
        max-width: fit-content;
      }
    }
  }

  .latest-comment-feed {
    .summary {
      max-width: 400px;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }

    a {
      color: #455a64;
    }

    a:hover {
      color: #ff8364;
    }
  }
}
</style>
