<template>
  <div class="container">
    <div class="content">
      <br>
      <Row>
        <Col :xs="{push: 1}" :lg="{push: 0}">
          <Breadcrumb>
            <BreadcrumbItem :to="{name: 'home'}">{{ $t("header.index") }}</BreadcrumbItem>
            <BreadcrumbItem>{{ $t("search.title") }}</BreadcrumbItem>
          </Breadcrumb>
        </Col>
      </Row>
      <br>
      <div class="styles_herosection user-select-none">
        <div class="styles_bg"></div>
        <img class="styles_bg_img" src="../assets/images/hero-grid-overlay.png"/>
      </div>
      <div
          :class="`search-content ${(searchVal.length > 3 && searchPosting) ? 'search-content-mini' : ''}`">
        <Row type="flex" justify="center" :gutter="20" style="width: 100%">
          <Col :xs="{span: 24}" :sm="{span: 18}" :md="{span: 18}">

            <div class="search-input search-input-show ivu-input ivu-input-large">
              <Row :gutter="10">
                <Col flex="1">
                  <Dropdown style="width: 100%">
                    <Input
                        size="small"
                        clearable
                        v-model="searchVal"
                        :border="false"
                        :placeholder="$t('search.placeholder')"
                        @on-enter="handleSearch"
                        @on-search="handleSearch">
                    </Input>

                    <!-- Search history S -->
                    <div transfer slot="list" style="padding: 10px">
                      <Row :gutter="5" v-if="searchHistory.list.length > 0">
                        <Col flex="1">
                          <h5><b>{{ searchHistory.list.length }}</b><span>/10</span></h5>
                        </Col>
                        <Col>
                          <Button size="small" type="primary" @click="deleteSearchHistoryAll">
                            <Icon type="md-trash"/>
                          </Button>
                        </Col>
                      </Row>
                      <template v-if="searchHistory.list.length > 0">
                        <Row :gutter="5">
                          <Col v-for="(i, index) in searchHistory.list"
                               :key="index">
                            <Tag stype="border"
                                 type="dot"
                                 :color="i.count > 0 ? 'success' : 'default'"
                                 checkable
                                 closable
                                 @on-change="handleSearchHistoryClickTag(index)"
                                 @on-close="handleSearchHistoryClose(index)">
                              {{ $t('search.tabs.' + i.type) || '' }}:{{ i.keyword }}
                            </Tag>
                          </Col>
                        </Row>
                      </template>
                      <div v-else align="center">
                        🦖
                      </div>
                    </div>
                    <!-- Search history E -->

                  </Dropdown>
                </Col>
                <Col>
                  <OcrWidget :data="{}" @ok="onOcrOutput">
                    <Tooltip content="OCR">
                      <a href="javascript:void(0)">
                        <Icon type="md-qr-scanner"/>
                      </a>
                    </Tooltip>
                  </OcrWidget>
                </Col>
                <Col>
                  <Button type="primary" size="small" @click="handleSearch">
                    <Icon type="ios-search" v-if="!modalSpinShow"/>
                    <Icon type="md-refresh spin-icon-load" v-else/>
                  </Button>
                </Col>
              </Row>

            </div>
          </Col>
        </Row>
        <Row type="flex" justify="center" align="middle" class="checkboxGroup">
          <Col :xs="{span: 24}" :lg="{span: 6}" align="center">
            <Icon type="md-alert"/>
            {{ $t("search.describe") }}
          </Col>
          <Col :xs="{span: 0}" :lg="{span: 1}">
            <Divider type="vertical"/>
          </Col>
          <Col :xs="{span: 24}" :lg="{span: 6}" align="center">
            <a href="javascript:void(0)">{{ $t("search.collectionHint") }}</a>
          </Col>
        </Row>
      </div>

      <template v-if="searchVal.length >= 3 && searchPosting">
        <Tabs v-model="searchTypeValue" @on-click="onTabClick">
          <div slot="extra">
            <Button size="small" @click="handleSearch">
              <Icon type="md-refresh" :class="modalSpinShow ? 'spin-icon-load' : ''"/>
            </Button>
          </div>

          <TabPane name="player" :label="$t('search.tabs.player')"></TabPane>
          <TabPane name="user" :label="$t('search.tabs.user')" :disabled="!isLogin"></TabPane>
          <TabPane name="comment" :label="$t('search.tabs.comment')" :disabled="!isLogin"></TabPane>
        </Tabs>

        <!-- 玩家 S -->
        <template v-if="searchTypeValue == 'player'">
          <Row :gutter="10">
            <Col flex="1">
              <RadioGroup v-model="searchGameSort" type="button" @on-change="handleSearch">
                <Radio label="default">{{ $t('search.sort.default') }}</Radio>
                <Radio label="latest">{{ $t('search.sort.latest') }}</Radio>
                <Radio label="mostViewed">{{ $t('search.sort.mostViewed') }}</Radio>
                <Radio label="mostComments">{{ $t('search.sort.mostComments') }}</Radio>
              </RadioGroup>
            </Col>
            <Col>
              <Select v-model="searchGameValue" :transfer="true" class="search-input-show" @on-change="handleSearch">
                <Icon type="ios-funnel" slot="prefix" style="margin-left: 10px; margin-right: 5px; opacity: .6"/>
                <Option v-for="i in searchGameList" :value="i.value" :key="i.value">
                  {{ $t('basic.games.' + i.value) }}
                </Option>
              </Select>
            </Col>
            <Col>
              <DatePicker type="daterange"
                          placement="bottom-end"
                          split-panels
                          @on-change="handleCDatepicker"
                          :options="timeOptions"
                          :value="intervalTime"
                          style="width: 100%"></DatePicker>
            </Col>
          </Row>
          <Card dis-hover class="list" v-if="result.player.length > 0">
            <div v-for="(d, d_index) in result.player" :key="d_index" class="item-card" v-voice-button>
              <Badge :text=" d.viewNum > 100 && d.commentsNum > 10 ? 'hot': ''" style="width: 100%">
                <Card dis-hover :padding="10">
                  <Row :gutter="10" type="flex">
                    <Col :xs="{span: 5, push: 0,pull:0}" :lg="{span: 3, push: 0,pull:0}">
                      <!-- 头像 S -->
                      <Avatar :src="d.avatarLink"
                              class="default-avatar"
                              alt="avatar"
                              size="55"
                              v-if="d.avatarLink">
                      </Avatar>
                      <template v-else>
                        <Avatar icon="ios-person"
                                class="default-avatar"
                                size="55"></Avatar>
                      </template>
                      <!-- 头像 E -->
                    </Col>
                    <Col :xs="{span: 18, push: 0,pull:0}" :lg="{span: 17, push: 0,pull:0}">
                      <div style="display: flex; flex-direction: column;">
                        <Tooltip :content="$t('list.colums.playerId')">
                          <h2>
                            <router-link :to="{name: 'player', params: { ouid: `${d.originPersonaId}` }}">
                              {{ d.historyName }}
                            </router-link>
                          </h2>
                        </Tooltip>
                      </div>

                      <div>
                        {{ $t('list.colums.reportTime') }}
                        <Time v-if="d.createTime" :time="d.createTime"/>
                        <Divider type="vertical"/>
                        {{ $t('list.colums.updateTime') }}
                        <Time v-if="d.updateTime" :time="d.updateTime"/>
                      </div>
                    </Col>
                    <Col :xs="{span: 24, push: 0,pull:0}" :lg="{span: 3, push: 0,pull:0}" class="mobile-hide">
                      <Row type="flex" justify="end" align="middle" style="height: 100%">
                        <Col flex="auto" align="right" class="item-text">
                          <span>{{ d.viewNum || 0 }}</span>
                          <Icon type="md-eye" size="17" class="item-icon"/>
                        </Col>
                        <Col flex="auto" align="right" class="item-text">
                          <span>{{ d.commentsNum || 0 }}</span>
                          <Icon type="md-chatboxes" size="17" class="item-icon"/>
                        </Col>
                      </Row>
                    </Col>
                    <Col :xs="{span: 24, push: 0,pull:0}" :lg="{span: 1, push: 0,pull:0}"
                         align="center"
                         class="mobile-hide">
                      <Progress vertical :percent="d.status == 1 ? 99 : 100" hide-info status="wrong"/>
                    </Col>
                  </Row>
                </Card>
              </Badge>
            </div>

            <Page class="page"
                  size="small"
                  show-sizer
                  show-total
                  show-elevator
                  @on-change="handlePageChange"
                  @on-page-size-change="handlePageSizeChange"
                  :page-size="limit.player"
                  :current="skip.player"
                  :total="total.player"/>
          </Card>
          <Card dis-hover class="list" :padding="10" v-else align="center">
            <Empty></Empty>
          </Card>
        </template>
        <!-- 玩家 E -->

        <!-- 用户 S -->
        <template v-if="searchTypeValue == 'user'">
          <Row :gutter="10">
            <Col flex="1">
              <RadioGroup v-model="searchUserSort" type="button" @on-change="handleSearch">
                <Radio label="default">{{ $t('search.sort.default') }}</Radio>
                <Radio label="joinedAt">{{ $t("account.joinedAt") }}</Radio>
                <Radio label="lastOnlineTime">{{ $t("account.lastOnlineTime") }}</Radio>
              </RadioGroup>
            </Col>
            <Col>
              <DatePicker type="daterange"
                          placement="bottom-end"
                          split-panels
                          @on-change="handleCDatepicker"
                          :options="timeOptions"
                          :value="intervalTime"
                          style="width: 100%"></DatePicker>
            </Col>
          </Row>
          <Card dis-hover class="list" v-if="result.user.length > 0">
            <div v-for="(user, user_index) in result.user" :key="user_index" class="item-card" v-voice-button>
              <Card dis-hover :padding="10" @click.native="$router.push({path: '/account/' + user.dbId})">
                <Row :gutter="10" type="flex">
                  <Col :xs="{span: 5, push: 0,pull:0}" :lg="{span: 3, push: 0,pull:0}">
                    <!-- 头像 S -->
                    <Avatar :src="user.userAvatar"
                            alt="avatar"
                            size="55"
                            v-if="user.userAvatar">
                    </Avatar>
                    <Avatar alt="avatar"
                            size="55"
                            v-else> {{ user.username }}
                    </Avatar>
                    <!-- 头像 E -->
                  </Col>
                  <Col :xs="{span: 18, push: 0,pull:0}" :lg="{span: 17, push: 0,pull:0}">
                    <div style="display: flex; flex-direction: column;">
                      <Tooltip :content="$t('list.colums.playerId')">
                        <h2>
                          {{ user.username }}
                        </h2>
                      </Tooltip>
                    </div>

                    <div>
                      {{ $t("account.joinedAt") }}
                      <Time v-if="user.joinTime" :time="user.joinTime"/>
                      <Divider type="vertical"/>
                      {{ $t("account.lastOnlineTime") }}
                      <Time v-if="user.signoutTime" :time="user.signoutTime"/>
                    </div>
                  </Col>
                  <Col :xs="{span: 4, push: 0,pull:0}" align="right">
                    <PrivilegesTag :data="user.privilege"></PrivilegesTag>
                  </Col>
                </Row>
              </Card>
            </div>

            <Page class="page"
                  size="small"
                  show-sizer
                  show-total
                  show-elevator
                  @on-change="handlePageChange"
                  @on-page-size-change="handlePageSizeChange"
                  :page-size="limit.user"
                  :current="skip.user"
                  :total="total.user"/>
          </Card>
          <Card dis-hover class="list" :padding="10" v-else align="center">
            <Empty></Empty>
          </Card>
        </template>
        <!-- 用户 E -->

        <!-- 评论 S -->
        <template v-if="searchTypeValue == 'comment'">
          <Row :gutter="10">
            <Col flex="1"></Col>
            <Col>
              <DatePicker type="daterange"
                          placement="bottom-end"
                          split-panels
                          @on-change="handleCDatepicker"
                          :options="timeOptions"
                          :value="intervalTime"></DatePicker>
            </Col>
          </Row>
          <Card dis-hover class="list" v-if="result.comment.length > 0">
            <div v-for="(comment, comment_index) in result.comment" :key="comment_index" class="timeline-content">
              <div class="timeline-time">
                <Row>
                  <Col flex="1">
                    <router-link :to="{name: 'account', params: {uId: `${comment.byUserId}`}}">
                      <BusinessCard :id="comment.byUserId">
                        <u><b>{{ comment.username || comment.byUserId }}</b></u>
                      </BusinessCard>
                    </router-link>

                    {{ $t('basic.button.reply') }}
                  </Col>
                  <Col align="right">
                    <Time v-if="comment.createTime" :time="comment.createTime" type="datetime"></Time>
                  </Col>
                </Row>
              </div>
              <Card dis-hover :padding="0"
                    class="item-card" v-voice-button>

                <HtmlWidget :html="comment.content"></HtmlWidget>
              </Card>
              <br>
            </div>

            <Page class="page"
                  size="small"
                  show-sizer
                  show-total
                  show-elevator
                  @on-change="handlePageChange"
                  @on-page-size-change="handlePageSizeChange"
                  :page-size="limit.comment"
                  :current="skip.comment"
                  :total="total.comment"/>
          </Card>
          <Card dis-hover class="list" :padding="10" v-else align="center">
            <Empty></Empty>
          </Card>
        </template>
        <!-- 评论 E -->
      </template>

    </div>
  </div>
</template>

<script>
import {api, http, storage} from "../assets/js";

import Application from "../assets/js/application";
import OcrWidget from "@/components/OcrWidget";
import PrivilegesTag from "@/components/PrivilegesTag";
import HtmlWidget from "@/components/HtmlWidget";
import BusinessCard from "@/components/businessCard";
import Empty from "@/components/Empty";

import game from '../../public/config/gameName.json';

export default new Application({
  name: "search",
  data() {
    return {
      modalSpinShow: false,
      searchPosting: false,

      // Search box and search history
      searchVal: '',
      searchHistory: {
        list: []
      },

      // Search mode: [player,user,comment]
      searchTypes: ['player', 'user', 'comment'],
      searchTypeValue: 'player',

      // Search for games, used when the searchTypes field is player
      searchGameList: [{value: 'all'}].concat(game.child),
      searchGameValue: 'all',
      searchGameSort: 'default',
      searchUserSort: 'default',

      intervalTime: undefined,
      timeOptions: {
        disabledDate(date) {
          return date && date.valueOf() > Date.now();
        },
        shortcuts: [
          {
            text: this.$i18n.t('sitestats.timeRange.daily'),
            value() {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24);
              return [start, end];
            }
          },
          {
            text: this.$i18n.t('sitestats.timeRange.weekly'),
            value() {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              return [start, end];
            }
          },
          {
            text: this.$i18n.t('sitestats.timeRange.monthly'),
            value() {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              return [start, end];
            }
          },
          {
            text: this.$i18n.t('sitestats.timeRange.yearly'),
            value() {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30 * 12);
              return [start, end];
            }
          }
        ]
      },

      result: {player: [], user: [], comment: []},
      skip: {player: 1, user: 1, comment: 1},
      limit: {player: 40, user: 20, comment: 10},
      total: {player: 0, user: 0, comment: 0}
    }
  },
  components: {OcrWidget, PrivilegesTag, HtmlWidget, BusinessCard, Empty},
  created() {
    const {keyword, type, game, skip, limit} = this.$route.query;
    this.searchVal = keyword || '';
    this.searchTypeValue = type || 'player';
    this.searchGameValue = game || 'all';
    if (type && skip >= 1 && skip <= Number.MAX_VALUE) this.skip[this.searchTypeValue] = Number(skip) || this.skip[this.searchTypeValue];
    if (type && limit >= 0 && limit <= 40) this.limit[this.searchTypeValue] = Number(limit) || this.limit[this.searchTypeValue];

    this.getSearchHistory();
    this.searchVal ? this.handleSearch() : null
  },
  methods: {
    getSearchHistory() {
      let history = storage.get('search.history');

      if (history.code == -1) {
        this.setSearchHistoryValue([]);
      }

      this.searchHistory.list = history.data.value || [];
    },
    setSearchHistoryValue(value) {
      const maxHistory = 10;
      let historyList = value;

      for (let i = 0; i < historyList.length; i++) {
        if (historyList.length > maxHistory) historyList.pop();
      }

      storage.set('search.history', historyList);
    },
    handleSearchHistoryClose(index) {
      this.searchHistory.list.splice(index, 1);

      this.setSearchHistoryValue(this.searchHistory.list);
    },
    handleSearchHistoryClickTag(index) {
      if (this.modalSpinShow) return;

      this.searchVal = this.searchHistory.list[index].keyword;
      this.searchTypeValue = this.searchHistory.list[index].type;
      this.handleSearch();
    },
    handlePageChange(num) {
      this.skip[this.searchTypeValue] = num;
      this.handleSearch();
    },
    handlePageSizeChange(num) {
      this.limit[this.searchTypeValue] = num;
      this.handleSearch();
    },
    handleCDatepicker(date) {
      this.intervalTime = date;
      this.skip[this.searchTypeValue] = 1; // reset

      this.handleSearch();
    },
    /**
     * Ocr输出
     * @param val
     */
    onOcrOutput(data) {
      this.searchVal = data.value;
      this.handleSearch();
    },
    /**
     * 删除历史记录
     */
    deleteSearchHistoryAll() {
      storage.rem('search.history');
      this.searchHistory.list = [];
    },
    /**
     * tabs 标签单击触发
     * @param tagName
     */
    onTabClick(tagName) {
      this.searchTypeValue = tagName;
      this.handleSearch();
    },
    /**
     * 搜索
     */
    handleSearch() {
      const that = this;
      const keyword = this.searchVal.trim();
      let data = {keyword, type: this.searchTypeValue};

      this.searchPosting = false;
      data = Object.assign({
        game: this.searchGameValue,
        gameSort: this.searchGameSort,
      }, data)
      this.$router.push({name: this.$router.name, query: data});

      switch (this.searchTypeValue) {
        case 'player':
          if (this.skip[this.searchTypeValue] >= 1) data.skip = Number((this.skip[this.searchTypeValue] - 1) * this.limit[this.searchTypeValue]);
          if (this.limit[this.searchTypeValue] >= 0 && this.limit[this.searchTypeValue] <= 40) data.limit = Number(this.limit[this.searchTypeValue]);

          // Time limit
          if (this.intervalTime) {
            data.createTimeFrom = new Date(this.intervalTime[0]).getTime();
            data.createTimeTo = new Date(this.intervalTime[1]).getTime();
          } else {
            delete data.createTimeFrom;
            delete data.createTimeTo;
          }
          break;
        case 'user':
        case 'comment':
          if (this.skip[this.searchTypeValue] >= 1) data.skip = Number((this.skip[this.searchTypeValue] - 1) * this.limit[this.searchTypeValue]);
          if (this.limit[this.searchTypeValue] >= 0 && this.limit[this.searchTypeValue] <= 40) data.limit = Number(this.limit[this.searchTypeValue]);
          break;
      }

      // restriction
      if (keyword == '' || keyword.length <= 3 || this.modalSpinShow) {
        this.$Message.info(this.$i18n.t('signin.fillEverything'))
        this.modalSpinShow = false;
        return;
      }

      this.modalSpinShow = true;
      data.param = data.keyword;

      http.get(api["search"], {params: data}).then(res => {
        const d = res.data;
        let notRepeat = false;

        for (let index = 0; index < that.searchHistory.list.length; index++) {
          let i = that.searchHistory.list[index];
          if (i.keyword == keyword && i.type == this.searchTypeValue) {
            notRepeat = true;
          }
        }

        if (!notRepeat) {
          that.searchHistory.list.push({
            keyword,
            type: this.searchTypeValue,
            count: d.data.length || 0
          })
        }

        that.setSearchHistoryValue(this.searchHistory.list);

        if (d.success === 1) {
          this.result[this.searchTypeValue] = d.data;
          this.total[this.searchTypeValue] = d.total;

          if (d.data.length <= 0) {
            this.$Message.info(this.$i18n.t('search.notUserExist'))
            return;
          }
          return;
        }

        this.$Message.error(d.message || d.data.code)
      }).catch(err => {
        this.$Message.error(err)
      }).finally(() => {
        that.modalSpinShow = false;
        that.searchPosting = true;
      })
    },
  }
});
</script>

<style scoped lang="less">
@import "@/assets/css/avatar.less";
@import "@/assets/css/radio.less";
@import "@/assets/css/icon.less";
@import "@/assets/css/timeline.less";

.search-background {
  position: fixed;
  background-size: cover;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
  opacity: .15;
}

.search-content {
  display: flex;
  flex-flow: column;
  justify-content: center;
  align-items: center;
  align-content: center;
  transition: all .25s;
  min-height: 600px;
  height: 100%;
  padding: 0 30px;

  &.search-content-mini {
    min-height: 150px !important;
  }

  .checkboxGroup {
    width: 100%;
    margin-bottom: 5px;
    margin-top: 50px;
    font-size: 1rem;
    color: #a8a8a8;
  }

  .search-input-show {
    box-shadow: 0 10px 50px #0000002b;
  }

  .search-input {
    width: 100% !important;
  }
}

.list {
  margin: 20px 0 0 0;
  position: relative;

  .item-card {
    margin-bottom: 10px;
  }

  .item-text {
    white-space: nowrap;
    font-size: 12px;
    font-weight: 400;
  }

  .item-icon {
    margin: 0 20px 0 5px;
  }
}
</style>
