<template>
  <div class="container">
    <div class="content">
      <br>
      <Row>
        <Col :xs="{push: 1}" :lg="{push: 0}">
          <Breadcrumb>
            <BreadcrumbItem :to="{name: 'home'}">{{ $t("header.index") }}</BreadcrumbItem>
            <BreadcrumbItem>{{ $t("sitestats.title") }}</BreadcrumbItem>
          </Breadcrumb>
        </Col>
      </Row>
      <br>

      <Col>
        <Card :padding="0" dis-hover>
          <Button @click="loadData" class="sitestats-loading">
            <Icon type="md-refresh" :class="[load ? 'spin-icon-load' :'']"/>
          </Button>
          <v-chart class="chart" :option="chart.stats"/>
        </Card>
        <Spin size="large" fix v-if="load"></Spin>
      </Col>
      <br>

      <Card dis-hover>
        <Row type="flex" justify="space-between" align="middle">
          <Col :xs="{span: 12}" :lg="{span: 3}">
            <h2>
              {{ statistics.reports || 0 }}
            </h2>
            <span>{{ $t('sitestats.reports') }}</span>
          </Col>
          <Divider type="vertical" class="mobile-hide"/>
          <Col :xs="{span: 12}" :lg="{span: 3}">
            <h2>
              <Badge :color="chart.stats.color[0]"/>
              {{ statistics.players || 0 }}
            </h2>
            {{ $t('sitestats.players') }}
          </Col>
          <Divider type="vertical" class="mobile-hide"/>
          <Col :xs="{span: 12}" :lg="{span: 3}">
            <h2>{{ statistics.banAppeals || 0 }}</h2>
            {{ $t('sitestats.banAppeals') }}
          </Col>
          <Divider type="vertical" class="mobile-hide"/>
          <Col :xs="{span: 12}" :lg="{span: 3}">
            <h2>
              <Badge :color="chart.stats.color[2]"/>
              {{ statistics.registers || 0 }}
            </h2>
            <span>{{ $t('sitestats.registers') }}</span>
          </Col>
          <Divider type="vertical" class="mobile-hide"/>
          <Col :xs="{span: 12}" :lg="{span: 3}">
            <h2>
              <Badge :color="chart.stats.color[1]"/>
              {{ statistics.confirmed || 0 }}
            </h2>
            <span>{{ $t('sitestats.confirmed') }}</span>
          </Col>
          <Divider type="vertical" class="mobile-hide"/>
          <Col :xs="{span: 12}" :lg="{span: 3}">
            <h2>{{ admins || 0 }}</h2>
            {{ $t('sitestats.admins') }}
          </Col>
        </Row>
      </Card>
      <br>

      <!-- 排行统计 S -->
      <div style="position: relative">
        <div>
          <Row :gutter="20">
            <Col :xs="{span: 23, push: 1}" :lg="{push: 0}">
              <RadioGroup size="small" type="button" v-voice-button v-model="timeRange"
                          @on-change="getActiveStatistical">
                <Radio v-for="(i, index) in timeArray" :key="index"
                       :disabled="!isLogin && !show"
                       :label="i.value"
                       :value="i.value">{{ $t(i.name) }}
                </Radio>
              </RadioGroup>
            </Col>
          </Row>
        </div>
        <br>

        <Row :gutter="20">
          <Col :xs="{span: 24}" :lg="{span: 12}">
            <Card dis-hover>
              <div slot="title">{{ $t('sitestats.communityParticipation') }}</div>
              <Row :gutter="5">
                <Col span="12">
                  <ol class="sitestats-ul" v-if="active.community.length > 0">
                    <li v-for="(i, index) in active.community" :key="index">
                      <Row>
                        <Col flex="1">
                          <businessCard :id="i.id">
                            <router-link :to="{name:'account', params: { uId: i.id }}">{{ i.username }}</router-link>
                          </businessCard>
                        </Col>
                        <Col>{{ i.total.toFixed(2) || 0 }}</Col>
                      </Row>
                    </li>
                  </ol>
                  <Empty v-else></Empty>
                </Col>
                <Col span="12">
                  <v-chart class="chart chart-min" :option="active.communityConf"/>
                </Col>
              </Row>
              <Spin size="large" fix v-show="active.load">
                <Icon type="ios-loading" size="50" class="spin-icon-load"></Icon>
              </Spin>
            </Card>
          </Col>
          <Col :xs="{span: 24}" :lg="{span: 12}">
            <Card dis-hover>
              <div slot="title">{{ $t('sitestats.reportRanking') }}</div>
              <ol class="sitestats-ul" v-if="active.report.length > 0">
                <li v-for="(i, index) in active.report" :key="index">
                  <Row>
                    <Col flex="1">
                      <businessCard :id="i.id">
                        <router-link :to="{name:'account', params: { uId: i.id }}">{{ i.username }}</router-link>
                      </businessCard>
                    </Col>
                    <Col>{{ i.total.toFixed(0) || 0 }}</Col>
                  </Row>
                </li>
              </ol>
              <Empty v-else></Empty>
              <Spin size="large" fix v-show="active.load">
                <Icon type="ios-loading" size="50" class="spin-icon-load"></Icon>
              </Spin>
            </Card>
          </Col>
        </Row>
        <br>
        <Row :gutter="20">
          <Col :xs="{span:24}" :lg="{span:12}">
            <Card dis-hover>
              <div slot="title">{{ $t('sitestats.trend') }}</div>
              <ol class="sitestats-ul" v-if="trend.list.length > 0">
                <li v-for="(i, index) in trend.list" :key="index">
                  <Row :gutter="10">
                    <Col flex="1">
                      <router-link :to="{name:'player', params: { ouid: i.originPersonaId }}">
                        {{ i.originName }}
                      </router-link>
                    </Col>
                    <Col>
                      <Icon type="md-chatbubbles" /> {{ i.commentsNum.toFixed(0) || 0 }}
                      <Divider type="vertical"></Divider>
                      <Icon type="md-eye" /> {{ i.viewNum.toFixed(0) || 0 }}
                    </Col>
                    <Col>
                      <Tag color="error">
                        <Icon type="ios-flame" /> {{ i.hot.toFixed(0) || 0 }}
                      </Tag>
                    </Col>
                  </Row>
                </li>
              </ol>
              <Empty v-else></Empty>
              <Spin size="large" fix v-show="trend.load">
                <Icon type="ios-loading" size="50" class="spin-icon-load"></Icon>
              </Spin>
            </Card>
          </Col>
        </Row>

        <Spin size="large" fix v-show="!isLogin">
          <div>
            <Icon type="md-lock" size="100" />
          </div>
          <br>
          <Button :to="{name: 'signin'}">{{ $t("header.signin") }}</Button>
        </Spin>
      </div>
      <!-- 排行统计 E -->

    </div>
  </div>
</template>

<script>
import Application from "../assets/js/application";
import Empty from "@/components/Empty"
import businessCard from "@/components/businessCard";
import * as echarts from "echarts";

import {http, api, account_storage} from '../assets/js/index'

export default new Application({
  data() {
    return {
      load: false,
      statistics: {},
      admins: 0,
      show: false,
      chart: {
        'stats': {
          color: ['#fff13c', '#401486', '#ed4014'],
          tooltip: {
            trigger: 'axis',
            axisPointer: {
              type: 'cross',
              label: {
                backgroundColor: '#f2f2f2',
                color: '#535353'
              }
            }
          },
          xAxis: {
            type: 'time',
            boundaryGap: true,
            splitLine: {
              show: false,
            }
          },
          yAxis: {
            type: 'value',
            boundaryGap: [0, '30%'],
            splitLine: {
              show: false,
            }
          },
          series: [],
          grid: {x: -1, y: -1, x2: 0, y2: 30},
        },
      },
      chartConf: {
        array: [{
          name: "players",
          valName: 'playerStats',
          lineColor: '#fff13c'
        }, {
          name: "confirmed",
          valName: 'confirmStats',
          lineColor: '#401486'
        }, {
          name: "registers",
          valName: 'userStats',
          lineColor: '#ed4014'
        }],
      },

      timeArray: [
        {name: 'sitestats.timeRange.daily', value: 'daily', show: false},
        {name: 'sitestats.timeRange.weekly', value: 'weekly', show: false},
        {
          name: 'sitestats.timeRange.monthly',
          value: 'monthly',
          show: false,
          privileges: ['admin', 'spuer', 'root', 'dev']
        },
        {
          name: 'sitestats.timeRange.yearly',
          value: 'yearly',
          show: false,
          privileges: ['admin', 'spuer', 'root', 'dev']
        }
      ],
      timeRange: 'weekly',

      trend: {
        load: false,
        list: []
      },

      isIncludingRobots: true,
      active: {
        load: false,
        communityConf: {
          tooltip: {
            trigger: 'item'
          },
          series: [
            {
              type: 'pie',
              itemStyle: {
                borderRadius: 5,
                borderWidth: 3,
              },
              radius: ['50%', '70%'],
              avoidLabelOverlap: false,
              label: {show: false, position: 'center'},
              labelLine: {show: false},
              data: []
            }
          ]
        },
        community: [],
        report: [],
      }
    }
  },
  components: {businessCard, Empty},
  created() {
    this.loadData();
  },
  methods: {
    async loadData() {
      this.init();

      this.load = true;

      await this.getSiteStats();
      await this.getStatisticsInfo();
      await this.getActiveStatistical();
      await this.getTrend();

      setTimeout(() => {
        this.load = false;
      }, 1000);
    },
    init() {
      this.chart['stats'].series = [];

      this.chartConf.array.map(i => {
        this.chart['stats'].series.push({
          name: this.$i18n.t(`sitestats.${i.name}`),
          valName: i.valName,
          type: 'line',
          smooth: true,
          symbol: 'none',
          stack: 'Total',
          showSymbol: true,
          lineStyle: {
            color: i.lineColor,
            width: 2
          },
          markLine: {
            symbol: ['none', 'none'],
            label: {show: false},
            data: [{xAxis: 1}, {xAxis: 3}, {xAxis: 5}, {xAxis: 7}]
          },
          areaStyle: {
            opacity: 1,
            color: i.lineColor
          },
          data: []
        });
      });

      this.timeArray.map(i => {
        i.show = account_storage.checkPrivilegeGroup(this.currentUser.userinfo, i.privileges);
      })
    },
    getActiveStatistical() {
      if (
          !this.isLogin &&
          !account_storage.checkPrivilegeGroup(this.currentUser.userinfo, this.timeRange.privileges)
      ) {
        // 例子
        let exp = [
          {username: 'Tom', total: 10},
          {username: 'Rudi', total: 5},
          {username: 'まどか', total: 3},
          {username: 'David', total: 5.1},
          {username: '张小明', total: 2.4},
          {username: 'Auston', total: 1},
          {username: 'Marcia', total: 1}
        ];
        this.active.report = exp;
        this.active.community = exp;
        this.active.community.forEach(i => {
          this.active.communityConf.series[0].data.push({value: i.total.toFixed(2), name: i.username})
        })
        return;
      }

      this.active.load = true;

      let selectTime = this.timeArray.filter(i => i.value == this.timeRange)[0].value;

      http.get('/activeStatistical', {
        params: {
          isBot: this.isIncludingRobots,
          time: selectTime,
          report: true,
          community: true
        }
      }).then(res => {
        const d = res.data;

        if (d.success == 1) {
          this.active = Object.assign(this.active, d.data);

          this.active.communityConf.series[0].data = [];
          d.data.community.forEach(i => {
            this.active.communityConf.series[0].data.push({value: i.total.toFixed(2), name: i.username})
          })
        }
      }).finally(() => {
        this.active.load = false;
      });

      this.getTrend();
    },
    /**
     * 获取话题排行
     */
    getTrend () {
      this.trend.load = true;

      http.get(api['trend'], {
        params:{
          limit: 10,
          time: this.timeRange
        }
      }).then(res => {
        const d = res.data;

        if (d.success == 1) {
          this.trend.list = d.data;
        }
      }).catch(res => {
        this.$Message.error(res.message);
      }).finally(() => {
        this.trend.load = false;
      });
    },
    /**
     * 统计信息
     * api/siteStats
     */
    async getSiteStats() {
      http.get(api['siteStats'], {}).then(res => {
        const d = res.data;

        if (d.success == 1) {
          for (const dKey in d.data) {
            this.chart['stats'].series.filter(i => {
              if (i.valName == dKey) {
                d.data[dKey].forEach(t => {
                  i.data.push([t.time, t.num]);
                })
              }
            })
          }
        }
      }).catch(res => {
        this.$Message.error(res.message);
      });

      return;
    },
    /**
     * 获取统计
     */
    getStatisticsInfo() {
      http.get(api["statistics"], {
        params: {
          reports: true,
          players: true,
          confirmed: true,
          registers: true,
          banAppeals: true,
          admins: true,
          from: 1514764800000
        }
      }).then(res => {
        const d = res.data;
        if (d.success == 1) {
          this.statistics = d.data;
        }
      }).finally(() => {
        this.getAdmins();
      });

      return;
    },
    /**
     * 获取社区管理
     */
    getAdmins() {
      http.get(api["admins"], {}).then(res => {
        const d = res.data;

        if (d.success == 1) {
          this.admins = d.data.length;
        }
      })
    }
  },
});
</script>

<style lang="less" scoped>
@import "@/assets/css/icon.less";

.chart {
  height: 300px;
}

.chart-min {
  width: 100%;
  height: 250px;
}

.sitestats-ul {
  padding-left: 25px;

  li {
    padding-left: 5px;
  }

  li:nth-child(1)::marker,
  li:nth-child(2)::marker,
  li:nth-child(3)::marker {
    font-size: 20px;
  }

  li:nth-child(3) {
    margin-bottom: 5px;
  }

  li::marker {
    font-size: 16px;
    display: block;
    padding-left: 20px;
  }
}

.sitestats-loading {
  position: absolute;
  z-index: 10;
  top: 10px;
  left: 10px;
}

.sitestats-avatar {
  position: relative;
  padding-top: 30px;
  padding-left: 40px;
}

.sitestats-avatar span {
  margin-left: -15px;
  margin-bottom: 10px;
  box-shadow: 0 0 10px #00000017;
}
</style>
