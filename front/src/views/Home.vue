<template>
  <div>
    <div class="container">
      <div class="styles_herosection user-select-none">
        <div class="styles_bg"></div>
        <img class="styles_bg_img" src="../assets/images/hero-grid-overlay.png"/>
      </div>

      <div class="content">
        <div class="ivu-alert-with-banner home-banner">
          <Row :gutter="30">
            <Col :xs="{span: 22, offset: 1}" :sm="12" :md="12" :lg="{span: 10, offset: 0}">
              <h1 class="title">
                {{ $t("home.cover.h1") }}
              </h1>
              <h2>{{ $t(`home.cover.h2.hint${Math.floor(Math.random() * 2)}`) }}</h2>
              <h3>{{ $t("home.cover.h3") }}</h3>
              <br>
              <p>{{ $t("home.cover.h4") }}</p>

              <Divider/>

              <router-link v-if="isLogin" :to="{name: 'profile', params: {pagename: 'information'}}">
                <Button type="primary" v-voice-button>{{ $t("header.profile") }}</Button>
              </router-link>
              <router-link v-else :to="{name: 'signup'}">
                <Button type="primary" v-voice-button>{{ $t("signup.title") }}</Button>
              </router-link>
              <Divider v-if="!user.token" type="vertical"/>
              <router-link :to="{name: 'about'}">
                <Button type="text" v-voice-button>
                  <Icon type="ios-help-circle-outline"/>
                  {{ $t("home.cover.subButton") }}
                </Button>
              </router-link>

              <Row :gutter="10" style="margin-top: 50px">
                <Col span="12">
                  <router-link :to="{name: 'site_stats'}">
                    <Card dis-hover>
                      <h3>{{ statistics.reports || 0 }}</h3>
                      <span>{{ $t("home.cover.dataReceived") }}</span>
                      <Spin size="large" fix v-if="statisticsInfoLoad"></Spin>
                    </Card>
                  </router-link>
                </Col>
                <Col span="12">
                  <router-link :to="{name: 'site_stats'}">
                    <Card dis-hover>
                      <h3>{{ statistics.confirmed || 0 }}</h3>
                      <span>{{ $t("home.cover.confirmData") }}</span>
                      <Spin size="large" fix v-if="statisticsInfoLoad"></Spin>
                    </Card>
                  </router-link>
                </Col>
              </Row>
              <br>
              <p>{{ $t("home.cover.endTime", {time: bannerTime}) }}</p>
              <br>
            </Col>
            <Col class="mobile-hide" :lg="{span: 13, push: 1}" type="flex" align="center" justify="center"
                 style="display: flex; justify-content: center; align-items: center">

              <Card dis-hover :padding="0" v-if="bannerImage">
                <img :src="bannerImage"
                     @click="openBannerWindowBox"
                     width="100%" class="ivu-row-top" style="margin-bottom: -10px;border-radius: 5px;">
              </Card>

            </Col>
          </Row>
        </div>
      </div>
    </div>

    <div class="home-box mobile-hide ivu-primary" v-if="activities_l.length > 0">
      <div class="container">
        <Row>
          <Col :lg="{span: 10, push: 0}">
            <h1 align="left">{{ $t("home.activity.title") }}</h1>
            <h5 align="left"
                v-html="$t('home.activity.description', {report: statistics.reports || 0, cheater: statistics.confirmed || 0})"></h5>
          </Col>
          <Col :lg="{span: 11, push: 3}" type="flex" align="right" justify="center">
            <router-link :to="{name: 'player_list'}">
              <Button type="dashed" v-voice-button>
                {{ $t('home.activity.more') }}
              </Button>
            </router-link>
          </Col>
        </Row>
      </div>
      <div class="lean-box">
        <div class="wrapper" :style="'animation: rowup ' + activities_l.length * 2.8 + 's linear infinite;'">
          <div class="icon-pair" v-for="activity in activities_l" :key="activity.id">
            <Card class="icon" v-for="a_i in activity" :key="a_i.id">
              <div align="center" style="margin-top: -80px">
                <Avatar size="80" :src="a_i.playerAvatarLink">
                  {{ a_i.username || a_i.byUserName || a_i.toPlayerName || 'null' }}
                </Avatar>
                <p>
                  <br>
                  <Tag color="success" v-if="a_i.type == 'judgement'">
                    {{ $t("basic.privilege.admin") }}
                  </Tag>
                  {{ a_i.username || a_i.byUserName || a_i.toPlayerName || 'null' }}
                  <Divider type="vertical"/>
                  <Time v-if="a_i.createTime" :time="a_i.createTime"></Time>
                </p>
              </div>

              <span v-if="a_i.type === 'report'">
                <router-link :to="{name: 'account', params: {uId: `${a_i.byUserId}`}}">
                  {{ a_i.byUserName }}
                </router-link>
                {{ $t('home.activity.activities.report') }}
                <Tag>
                  {{ $t('basic.games.' + a_i.game) }}
                </Tag>
                <router-link
                    :to="{name: 'player', params: {game: `${a_i.game}`, ouid: `${a_i.playerOriginPersonaId}`}}">
                  {{ a_i.toPlayerName }}
                </router-link>
              </span>

              <span v-if="a_i.type === 'register'">
                <router-link :to="{name: 'account', params: {uId: `${a_i.byUserId}`}}">
                  {{ a_i.byUserName }}
                </router-link>
                {{ $t('home.activity.activities.join') }}
              </span>

              <span v-if="a_i.type === 'verify' || a_i.type === 'judgement'">
                <router-link :to="{name: 'account', params: {uId: `${a_i.byUserId}`}}">
                  <Tag v-if="a_i.privilege === 'admin'" color="success">
                    {{ $t('basic.privilege.admin') }}
                  </Tag>
                  <b>{{ a_i.byUserName }}</b>
                </router-link>

                {{ $t('detail.info.judge') }}

                <router-link :to="{name: 'player', params: {ouid: `${a_i.playerOriginPersonaId}`}}">
                  {{ a_i.toPlayerName }}
                </router-link>

                <Tag color="warning">
                  {{ $t(`basic.action.${a_i.action}.text`) }}
                </Tag>

                <span v-if="a_i.cheatMethods">
                  ，{{ $t('detail.info.cheatMethod') }}
                  <b>{{ convertCheatMethods(a_i.cheatMethods) }}</b>
                </span>
              </span>

              <span v-if="a_i.type === 'banAppeal'">
                <router-link :to="{name: 'account', params: {uId: `${a_i.byUserId}`}}">
                  {{ a_i.byUserName }}
                </router-link>
                {{$t('detail.appeal.info.content')}}
              </span>
            </Card>
          </div>
        </div>
      </div>
      <Spin size="large" fix v-if="activityLoad"></Spin>
    </div>

    <div class="container">
      <Tell class="mobile-hide content"></Tell>
    </div>
  </div>
</template>

<script>
import {api, http, util, time, regular, upload} from '../assets/js/index'

import Application from "../assets/js/application";
import Tell from "../components/Home_tell";

export default new Application({
  data() {
    return {
      bannerImage: '',
      bannerTime: '',

      statisticsInfoLoad: false,
      activityLoad: false,
      activities: [],
      activities_l: [],

      statistics: {
        reports: 0,
        confirmed: 0,
      },
    }
  },
  components: {Tell},
  watch: {
    '$route': 'loadData',
  },
  created() {
    this.loadData();
    this.getStatisticsInfo();
    this.getActivity();
  },
  methods: {
    async loadData() {
      await util.initUtil().then(res => {
        this.cheaterStatus = res.cheaterStatus;
        this.cheatMethodsGlossary = res.cheatMethodsGlossary;

        this.gameName = res.gameName;
      });

      this.bannerTime = time.appStart();

      try {
        this.bannerImage = require(`../assets/images/index-gl_${this.$i18n.locale || 'en-US'}.png`);
      } catch (e) {
        this.bannerImage = require(`../assets/images/index-gl_en-US.png`);
      }
    },
    /**
     * 查看首页介绍图片
     */
    openBannerWindowBox() {
      this.$viewerApi({
        options: {
          toolbar: false,
          navbar: false,
          keyboard: false,
          fullscreen: true,
        },
        images: [this.bannerImage],
      })
    },
    /**
     * 获取动态
     */
    getActivity() {
      this.activityLoad = true;

      http.get(api["activity"], {}).then(res => {
        const d = res.data;
        if (d.success === 1) {
          let activities = d.data;
          let new_activities = [];

          // slice array
          for (let i = 0; i < activities.length; i += 3) {
            new_activities.push(activities.slice(i, i + 3));
          }

          this.activities_l = new_activities;
        }
      }).finally(() => {
        this.activityLoad = false;
      })
    },
    /**
     * 获取统计
     */
    getStatisticsInfo() {
      this.statisticsInfoLoad = true;

      http.get(api["statistics"], {
        params: {
          reports: true, // show reports number
          players: true,	// show players that is reported number
          confirmed: true,	// show confirmed number
          registers: true,	// show register number
          banappeals: true,// show ban appeals number
          details: true,	// show number of each game, each status
          from: 1514764800000
        }
      }).then(res => {
        const d = res.data;

        if (d.success === 1) {
          this.statistics = d.data;
        }
      }).finally(() => {
        this.statisticsInfoLoad = false;
      })
    }
  },
  computed: {
    user() {
      return this.$store.state.user || {}
    }
  },
})
</script>

<style lang="less">
@keyframes rowup {
  from {
    transform: translateX(0%);
  }

  to {
    transform: translateX(-50%);
  }
}

.home-box {
  position: relative;
  padding-top: 100px;
  overflow: hidden;
  min-height: 850px;
  max-height: 1000px;
  margin: 50px 0 -20px 0;
  text-align: center;

  .lean-box {
    display: flex;
    transform: rotate(-5deg);
    margin-top: 50px;
  }
}

.home-banner {
  overflow: hidden;
  min-height: 600px;
  margin-top: 50px;
  background-size: 500px;
  background-repeat: no-repeat;
  background-position: right;

  .title {
    position: relative;
    border-bottom: 5px solid #fff13c;
    display: inline-flex;
    padding: 0 20px 0 0;
    margin-top: 80px;
    margin-bottom: 40px;
    font-weight: bold;
    font-size: 70px;
  }

  h2, h3, p {
    line-height: 1.5;
    margin-bottom: 10px;
  }
}

.wrapper {
  margin-top: 40px;
  display: flex;
  flex-wrap: nowrap;

  .icon {
    font-size: 12px;
    width: 280px;
    height: 160px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: bold;
    transform: translateX(0) translateY(40px);
    opacity: .6;
    transition: all 1s;
  }

  .icon:hover {
    opacity: 1;
  }

  .icon:nth-child(even) {
    margin-top: 105px;
    margin-left: 45px;
    transform: translateX(45px) translateY(-10px);
  }
}
</style>
