<template>
  <div class="container">
    <br>
    <Row>
      <Col flex="1" :xs="{push: 1}" :lg="{push: 0}">
        <Breadcrumb>
          <BreadcrumbItem :to="{name: 'home'}">{{ $t("header.index") }}</BreadcrumbItem>
          <BreadcrumbItem>{{ $t("account.title") }}</BreadcrumbItem>
        </Breadcrumb>
      </Col>
      <Col v-if="currentUser && currentUser.userinfo">
        <Poptip trigger="hover">
          <Button @click="openMessage" :disabled="!account.attr.allowDM || account.id == currentUser.userinfo.userId">
            <Icon type="ios-chatbubbles"/>
            {{ $t("account.message.chat") }}
          </Button>
          <div slot="content">
            <Alert show-icon type="error" v-if="!account.attr.allowDM"> {{ $t("account.message.hint.taOffChat") }}</Alert>
            <Alert show-icon type="error" v-if="account.id == currentUser.userinfo.userId">
              {{ $t("account.message.hint.selfTalk") }}
            </Alert>
          </div>
        </Poptip>
      </Col>
    </Row>
    <br>

    <template v-if="account">
      <div dis-hover bordered>
        <Row type="flex" justify="center" align="middle">
          <Col justify="center" align="middle">
            <br>
            <Avatar shape="square"
                    style="background-color: yellow"
                    size="150"
                    icon="ios-person"
                    :src="account.userAvatar ? `${account.userAvatar}` : ''"></Avatar>

            <h1 :title="$t('account.username')" class="account-username">
              {{ account.username || 'username' }}
            </h1>

            <Row :gutter="20" type="flex" justify="center" align="middle">
              <Col>
                <PrivilegesTag :data="account.privilege" v-if="account.privilege"></PrivilegesTag>
                <p v-else>-</p>
                <p class="account-info-p">{{ $t("account.role") }}</p>
              </Col>
              <Divider type="vertical"/>
              <Col>
                <Tag type="border" size="large" color="primary" v-if="account.joinTime">
                  <Time :time="account.joinTime || new Date()"/>
                </Tag>
                <p v-else>-</p>
                <p class="account-info-p">{{ $t("account.joinedAt") }}</p>
              </Col>
              <Divider type="vertical"/>
              <Col>
                <Tag type="border" size="large" color="#df22ff" v-if="account.lastOnlineTime">
                  <Time :time="account.lastOnlineTime || new Date()"/>
                </Tag>
                <p v-else>-</p>
                <p class="account-info-p">{{ $t("account.lastOnlineTime") }}</p>
              </Col>
              <Divider type="vertical"/>
              <Col>
                <h3>{{ account.reportnum || '-' }}</h3>
                <p class="account-info-p">{{ $t("account.reportNum") }}</p>
              </Col>
            </Row>
          </Col>
        </Row>
      </div>

      <br/>

      <div class="content">
        <Row :gutter="isAttachedContent ? 0 : 15">
          <Col :xm="{span: 24}" :lg="isAttachedContent ? {span: 24} : {span: 17}"
               style="width: 100%">
            <Card dis-hover :padding="0">
              <Table show-header
                     :border="false"
                     :no-data-text="$t('basic.tip.noReports')"
                     :columns="report.columns"
                     :data="report.data"></Table>
            </Card>

            <br/>
            <Row>
              <Col :xs="{span: 23, push: 1}" :lg="{span: 24, push: 0}">
                <Page
                    show-total
                    @on-change="getReports"
                    :page-size="limit"
                    :current="page"
                    :total="total"
                    class="page"
                    size="small"/>
              </Col>
            </Row>
          </Col>
          <Col :xm="{span: 22, push: 2}" :lg="isAttachedContent ? {span: 0, push: 0} : {span: 7, push: 0}">
            <Card v-if="account.attr.introduction" dis-shadow>
              <div v-html="account.attr.introduction"></div>
            </Card>
            <br v-if="account.attr.introduction">

            <Card v-if="account.origin && account.origin.originName" dis-shadow>
              <b>origin id:</b>
              <p>{{ account.origin.originName }}</p>
            </Card>
            <br v-if="account.origin && account.origin.originName">
          </Col>
        </Row>
      </div>

      <Modal v-model="message.show" @on-ok="onPushMessage">
        <Form>
          <FormItem :label="$i18n.t('account.message.chat')">
            <Input v-model="message.content"
                   type="textarea" :autosize="{minRows: 5,maxRows: 10}"></Input>
          </FormItem>
        </Form>
      </Modal>
    </template>
    <template v-else>
      <div align="center">
        <Empty></Empty>
      </div>
    </template>
  </div>
</template>

<script>
import Application from "../assets/js/application";
import Empty from "@/components/Empty";
import {api, http, http_token} from '../assets/js/index'

import PrivilegesTag from "/src/components/PrivilegesTag";

import games from '/public/config/gameName.json'

export default new Application({
  data() {
    return {
      games: games.child,
      load: false,
      account: {
        username: "",
        originId: "",
        privilege: "",
        createDatetime: "",
        attr: {
          allowDM: false,
        }
      },
      report: {
        columns: [
          {
            title: this.$i18n.t("account.reported"),
            key: 'originName',
            fixed: "left",
            minWidth: 200,
            maxWidth: 300,
            ellipsis: true,
            tooltip: true,
            render: (h, params) => {
              const that = this;
              return h('row', {
                props: {
                  type: 'flex',
                  align: 'middle',
                }
              }, [
                h('col', [
                  h('Avatar', {
                    props: {
                      src: params.row.avatarLink
                    },
                    style: {
                      margin: '0 10px 0 0'
                    }
                  })
                ]),
                h('col', [
                  h('a', {
                    attr: {
                      title: params.row.originName
                    },
                    style: {
                      "overflow": "hidden",
                      "display": "block",
                      "width": "100px",
                      "text-overflow": "ellipsis",
                      "white-space": "nowrap"
                    },
                    on: {
                      click() {
                        that.$router.push({
                          name: 'player',
                          params: {
                            ouid: params.row.originPersonaId
                          }
                        })
                      }
                    }
                  }, params.row.originName)
                ])
              ]);
            }
          },
          {
            title: this.$i18n.t('signup.form.originId'),
            key: 'originPersonaId',
            ellipsis: true,
            width: 200,
            align: 'center',
            render: (h, params) => {
              return h('div', [
                h('p', params.row.originPersonaId)
              ]);
            }
          },
          {
            title: this.$i18n.t("account.status"),
            key: 'status',
            width: 200,
            align: 'center',
            render: (h, params) => {
              return h('div', [
                h('Tag', {
                  props: {
                    color: 'error'
                  }
                }, `${this.$i18n.t(`basic.status[${params.row.status}]`)}`)
              ]);
            }
          },
          {
            title: this.$i18n.t("list.colums.reportTime"),
            key: 'createTime',
            sortable: true,
            minWidth: 150,
            render: (h, params) => {
              return h('Tag', {
                props: {
                  color: 'primary'
                }
              }, [
                h('Time', {
                  props: {
                    time: params.row.createTime,
                    type: 'datetime'
                  }
                })
              ]);
            }
          },
          {
            title: this.$i18n.t("list.colums.updateTime"),
            key: 'updateTime',
            align: 'center',
            width: 200,
            sortable: true,
            render: (h, params) => {
              return h('tag', {
                props: {
                  color: "primary"
                }
              } ,[
                h('Time', {
                  props: {
                    time: params.row.updateTime
                  }
                })
              ]);
            }
          }
        ],
        data: []
      },
      limit: 20,
      page: 1,
      total: 100,

      message: {
        id: '',
        show: false,
        load: false,
        content: '',
      }
    };
  },
  watch: {
    $route: "loadData",
  },
  components: {PrivilegesTag, Empty},
  created() {
    this.http = http_token.call(this);

    this.loadData();
  },
  methods: {
    async loadData() {
      const {uId} = this.$route.params;

      this.getUserInfo(uId);
    },
    /**
     * 获取用户信息
     */
    getUserInfo(uId) {
      this.$Loading.start();

      http.get(api["user_info"], {
        params: {
          id: uId
        }
      }).then(res => {
        const d = res.data;

        if (this.$route.query.repeat) {
          this.openMessage()
        }

        if (d.success === 1) {
          this.account = d.data;
          return;
        }

        this.account = "";
        this.$Message.warning(d.code);

      }).catch(err => {
        this.$Loading.error();
      }).finally(() => {
        this.getReports()
      });
    },
    /**
     * 打开消息
     */
    openMessage() {
      if (!this.account.attr.allowDM) {
        this.$Message.error(this.$i18n.t("account.message.hint.taOffChat"))
        return
      }
      if (this.account.id == this.currentUser.userinfo.userId) {
        this.$Message.error(this.$i18n.t("account.message.hint.selfTalk"))
        return
      }

      this.message.show = true;
    },
    /**
     * 发送消息
     */
    onPushMessage() {
      const {uId} = this.$route.params;

      if (!uId) return;

      this.http.post(api["user_message"], {
        data: {
          data: {
            toUserId: this.message.id || uId,
            type: 'direct',
            content: this.message.content,
          }
        }
      }).then(res => {
        if (res.data.success == 1) {
          this.$Message.success(res.data.message);

          return;
        }

        switch (res.data.error) {
          case 'message.denied':
            this.$Message.error(this.$i18n.t("account.message.hint.denied"));
            break;
          case 'message.blocked':
            this.$Message.error(this.$i18n.t("account.message.hint.taOffChat"));
            break;
          default:
            this.$Message.error(res.data.message);
        }
      }).finally(() => {
        this.message.load = false;
        this.message.show = false;
      })
    },
    /**
     * 获取举报信息
     * @param uId
     */
    getReports(pageNum) {
      const {uId} = this.$route.params;

      if (!uId) return;

      this.load = true;

      http.get(api["user_reports"], {
        params: {
          id: uId,
          skip: (pageNum || 1) - 1,
          limit: this.limit,
        }
      }).then(res => {
        const d = res.data;
        let reportData = [];

        if (d.success === 1) {
          d.data.forEach(i => {
            reportData.push(i);
          });

          this.report.data = reportData;
          this.total = Number(this.account.reportnum);
        }
      }).finally(() => {
        this.$Loading.finish();
        this.load = false;
      });
    },
  },
  computed: {
    /**
     * 是否包含用户附带的额外内容
     * 如果自我描述以及attr特定属性不显示，则关闭右侧一栏
     * @returns {boolean}
     */
    isAttachedContent () {
      return !this.account.attr.introduction && !this.account.origin;
    }
  }
});
</script>

<style lang="less">
.account-username {
  margin: 2rem 0 2rem 0;
}

.account-info-p {
  margin: .2rem;
  font-size: 12px;
  opacity: .6;
}
</style>
