<template>
  <div>
    <Form :model="formItem" label-position="top">
      <Row :gutter="30">
        <Col span="12">
          <FormItem>
            <div slot="label">
              <Icon type="md-key"/>
              {{ $t('signup.form.username') }}
            </div>
            <Input v-model="formItem.username" placeholder="" disabled>
              <a slot="append" @click="modal_setusername.show = !modal_setusername.show">
                <Icon type="md-create" size="15"/>
              </a>
            </Input>
          </FormItem>
        </Col>
        <Col span="12">
          <FormItem>
            <div slot="label">
              <Icon type="md-lock"/>
              {{ $t('signup.form.password') }}
            </div>
            <Input v-model="formItem.password" disabled type="password">
              <a slot="append" @click="modal_changePassword.show = !modal_changePassword.show">
                <Icon type="md-create" size="15"/>
              </a>
            </Input>
          </FormItem>
        </Col>
        <Col span="12">
          <Card dis-hover>
            <FormItem :label="$t('account.joinedAt')">
              <Tag>
                <Time :time="formItem.joinTime || new Date().getTime()"/>
              </Tag>
            </FormItem>
          </Card>
        </Col>
        <Col span="12">
          <Card dis-hover>
            <FormItem :label="$t('account.lastOnlineTime')">
              <Tag>
                <Time :time="formItem.lastOnlineTime || new Date().getTime()"/>
              </Tag>
            </FormItem>
          </Card>
        </Col>
      </Row>

      <Divider dashed></Divider>
      <template v-if="!isBindAccount">
        <Alert show-icon type="error">
          {{ $t('account.bindOrigin.title') }}
          <Icon type="ios-bulb-outline" slot="icon"></Icon>
          <template slot="desc">
            <p>{{ $t('account.bindOrigin.content') }}</p><br>
            <router-link to="/bindOrigin">
              <Button>{{ $t('account.bindOrigin.travel') }}</Button>
            </router-link>
          </template>
        </Alert>
      </template>
      <template v-else>
        <Row :gutter="30">
          <Col flex="1">
            <FormItem :label="$t('signup.form.originName')">
              <Input v-model="formItem.origin.originName"
                     type="text"
                     disabled
                     :autosize="{minRows: 2,maxRows: 5}"
                     placeholder=""></Input>
            </FormItem>
          </Col>
          <Col flex="1">
            <FormItem :label="$t('signup.form.originId')">
              <Input v-model="formItem.origin.originUserId"
                     type="text"
                     disabled
                     :autosize="{minRows: 2,maxRows: 5}"
                     placeholder=""></Input>
            </FormItem>
          </Col>
        </Row>
      </template>

      <Divider dashed></Divider>

      <Row :gutter="30">
        <Col span="12">
          <FormItem :label="$t('profile.account.form.language')">
            <Row>
              <Col>
                <Checkbox v-model="langLoaclSync" @on-change="switchLangLocalSync"></Checkbox>
              </Col>
              <Col flex="1">
                <Select v-model="formItem.attr.language"
                        class="switch-language"
                        prefix="md-globe"
                        placement="top-end"
                        :disabled="!langLoaclSync"
                        @on-change="switchLanguage">
                  <Option v-for="item in languages" :value="item.name" :key="item.name" :disabled="item.ignoreSave">
                    {{ item.label }}
                  </Option>
                </Select>
              </Col>
            </Row>
            <Alert show-icon v-if="langLoaclSync">{{ $t('profile.account.form.languageSyncDescribe') }}</Alert>
          </FormItem>
        </Col>
        <Col span="12">
          <FormItem :label="$t('profile.account.form.showOrigin')">
            <Alert show-icon>{{ $t('profile.account.form.showOriginDescribe') }}</Alert>
            <i-switch v-model="formItem.attr.showOrigin"/>
          </FormItem>
        </Col>
        <Col span="12">
          <FormItem :label="$t('profile.account.form.allowDM')">
            <Alert show-icon>{{ $t('profile.account.form.allowDMdescribe') }}</Alert>
            <i-switch v-model="formItem.attr.allowDM"/>
          </FormItem>
        </Col>
      </Row>

      <Affix :offset-bottom="0">
        <Card dis-hover :padding="8">
          <Row>
            <Col :xs="{span: 0}" :lg="{span: 20}">
            </Col>
            <Col :xs="{span: 24}" :lg="{span: 4}">
              <Button type="primary" long :loading="formLoad" @click="onSave">
                {{ $t("basic.button.save") }}
              </Button>
            </Col>
          </Row>
        </Card>
      </Affix>
    </Form>

    <!-- 修改名称 S -->
    <Modal v-model="modal_setusername.show" @on-cancel="modal_setusername.index = 0"
           :footer-hide="modal_setusername.index >= 2">
      <p slot="header">
        <span>{{ $t('profile.account.modifyName.title') }}</span>
      </p>
      <div>
        <Steps :current="modal_setusername.index" size="small">
          <Step :title="$t('profile.account.modifyName.steps[0].title')"></Step>
          <Step :title="$t('profile.account.modifyName.steps[1].title')"></Step>
          <Step :title="$t('profile.account.modifyName.steps[2].title')"></Step>
        </Steps>
        <Form ref="formValidate" label-position="top">
          <div v-if="modal_setusername.index == 0">
            <br>
            <Alert type="warning">
              <template slot="desc">
                <p v-html="$t('profile.account.modifyName.specification1')"></p>
                <p>
                  {{
                    $t(
                        'profile.account.modifyName.residueDegree',
                        {changeNameLeft: formItem.attr.changeNameLeft || 0}
                    )
                  }}
                </p>
                <br>
                <b> {{ $t('profile.account.modifyName.specification2') }}</b>
              </template>
            </Alert>
          </div>
          <div v-if="modal_setusername.index == 1">
            <br>
            <FormItem :label="$t('signup.form.username')">
              <Input v-model="formItem.username" size="large" disabled/>
            </FormItem>
            <FormItem :label="$t('signup.form.newUsername')">
              <Input v-model="formItem.newname" size="large" :placeholder="$t('signup.form.newUsername')"/>
            </FormItem>

            <FormItem :label="$t('captcha.title')" prop="captcha">
              <Input type="text" v-model="usernameCaptcha" size="large" maxlength="4"
                     :placeholder="$t('captcha.title')">
                <div slot="append" class="captcha-input-append" :alt="$t('captcha.get')">
                  <Captcha id="usernameCaptcha" ref="usernameCaptcha"></Captcha>
                </div>
              </Input>
            </FormItem>
          </div>
          <div v-if="modal_setusername.index == 2" align="center">
            <br>
            <Icon type="ios-cloud-done" size="200" color="#19be6b"/>
            <br>
          </div>
        </Form>
      </div>
      <div slot="footer">
        <Button size="large"
                :disabled="formItem.newname == ''"
                :loading="modal_setusername.load"
                v-if="modal_setusername.index > 0 && modal_setusername.index <= 1"
                @click="modal_setusername.index = 1 ? setUserName() : null">
          {{ $t('basic.button.next') }}
        </Button>
        <Button type="warning"
                size="large"
                v-if="modal_setusername.index <= 0"
                :disabled="formItem.attr.changeNameLeft == 0"
                @click="modal_setusername.index = 1">
          {{ $t('basic.button.submit') }}
        </Button>
      </div>
    </Modal>
    <!-- 修改名称 E -->

    <!-- 修改密码 S -->
    <Modal v-model="modal_changePassword.show">
      <p slot="header">{{ $t("reset.title") }}</p>
      <div>
        <Form ref="formValidate" label-position="top">
          <FormItem :label="$t('reset.form.oldPassword')">
            <Input v-model="modal_changePassword.oldpassword" placeholder="******" minlength="6"></Input>
          </FormItem>
          <FormItem :label="$t('reset.form.newPassword')">
            <Input v-model="modal_changePassword.newpassword" placeholder="******" minlength="6"></Input>
          </FormItem>

          <FormItem :label="$t('captcha.title')" prop="captcha">
            <Input type="text" v-model="passwordCaptcha" size="large" maxlength="4"
                   :placeholder="$t('captcha.title')">
              <div slot="append" class="captcha-input-append" :alt="$t('captcha.get')">
                <Captcha id="passwordCaptcha" ref="passwordCaptcha"></Captcha>
              </div>
            </Input>
          </FormItem>
        </Form>
      </div>
      <div slot="footer">
        <Button size="large"
                :disabled="!modal_changePassword.oldpassword && !modal_changePassword.newpassword && !passwordCaptcha"
                :loading="modal_changePassword.load"
                @click="handChangePassword()">
          {{ $t('reset.title') }}
        </Button>
      </div>
    </Modal>
    <!-- 修改密码 E -->

  </div>
</template>

<script>
import Captcha from "../../components/Captcha";

import {api, http, http_token, account_storage} from "../../assets/js";

export default {
  name: "account",
  data() {
    return {
      privileges: [],
      languages: [],
      formLoad: false,
      langLoaclSync: false,

      modal_changePassword: {
        load: false,
        show: false,
        newpassword: "",
        oldpassword: ""
      },

      modal_setusername: {
        load: false,
        show: false,
        index: 0
      },

      usernameCaptcha: "",
      passwordCaptcha: "",
    }
  },
  components: {Captcha},
  created() {
    this.http = http_token.call(this);

    this.ready();
  },
  methods: {
    async ready() {
      const languages = await import('/public/config/languages.json');
      const privileges = await import('/public/config/privilege.json');
      this.privileges = this.privileges.concat(privileges.child)
      this.languages = this.languages.concat(languages.child)

      await this.getUserinfo();
    },
    /**
     * 修改密码
     */
    handChangePassword() {
      const {newpassword = '', oldpassword = ''} = this.modal_changePassword;
      this.modal_changePassword.load = true;
      this.http.post(api["user_changePassword"], {
        data: {
          data: {
            newpassword,
            oldpassword
          },
          encryptCaptcha: this.$refs.passwordCaptcha.hash,
          captcha: this.passwordCaptcha
        }
      }).then(res => {
        const d = res.data;
        if (d.success === 1) {
          this.$Message.success(d.message);

          this.$store.dispatch('signout').then(() => {
            this.signout();
          });

          this.modal_changePassword.show = false;

          this.$router.push({name: "signin"});
          return;
        }

        this.$Message.error(d.message);
      }).finally(() => {
        this.modal_changePassword.load = false;
      });
    },
    /**
     * 注销
     */
    signout() {
      http.post(api["account_signout"], {
        headers: {
          'x-access-token': this.$store.state.user.token
        }
      }).then(res => {
        const d = res.data;
        if (d.success == 1) {
          this.$router.push('/signin');
        }
      })
    },
    /**
     * 切换语言
     * @param lang
     */
    switchLanguage(lang) {
      this.formItem.attr.language = lang;
    },
    /**
     * 修改名字
     */
    setUserName() {
      if (!this.formItem.newname) return;

      this.modal_setusername.load = true;

      this.http.post(api["user_changeName"], {
        data: {
          data: {
            newname: this.formItem.newname,
          },
          encryptCaptcha: this.$refs.usernameCaptcha.hash,
          captcha: this.usernameCaptcha
        }
      }).then(res => {
        const d = res.data;

        if (d.success == 1) {
          this.modal_setusername.index = 2;
          this.$Message.success(d.code);
          return;
        }

        this.modal_setusername.index = 0;
        this.$Message.error(d.message || d.code);
      }).finally(async () => {
        this.modal_setusername.load = false;
        this.usernameCaptcha = "";

        await this.getUserinfo();
      })
    },
    /**
     * 保存表单
     */
    onSave() {
      const {
        attr = {language: this.$root.$i18n.locale, showOrigin: false, allowDM: false}
      } = this.formItem;

      this.formLoad = true;
      this.http.post(api["user_me"], {
        data: {
          data: {attr}
        }
      }).then(res => {
        const d = res.data;

        if (d.success === 1) {

          // 同步本地语言
          if (d.data.attr.language) {
            this.$store.dispatch('setLang', d.data.attr.language);
          }

          this.$Message.success(d.code);
        }
      }).finally(() => {
        this.formLoad = false;
      })
    },
    /**
     * 获取用户信息
     */
    async getUserinfo() {
      return new Promise(resolve => {
        this.http.get(api["user_me"], {}).then(res => {
          const d = res.data;

          if (d.success === 1) {
            this.$store.dispatch('setUserInfo', d.data);

            this.formItem = Object.assign(this.formItem, d.data);
          }
        }).finally(() => {
          this.checkLangLocalSync();
          resolve();
        })
      })
    },
    /**
     * 切换本地同步
     * 是否同步再登录后同步语言
     */
    switchLangLocalSync(val) {
      account_storage.updateConfiguration('langLoaclSync', this.langLoaclSync);
    },
    checkLangLocalSync() {
      this.langLoaclSync = account_storage.getConfiguration('langLoaclSync');
    }
  },
  computed: {
    formItem: {
      set() {
        return true
      },
      get() {
        this.checkLangLocalSync();
        let data = Object.assign({
          password: '******',
          origin: null,
          userAvatar: '',
          newname: '',
          username: '',
          attr: {
            language: ''
          }
        }, this.$store.state.$userinfo);
        return data;
      }
    },
    isBindAccount() {
      let formItem = this.formItem;
      if (!formItem.origin) return false;
      if (formItem.origin.originName || formItem.origin.originUserId) return true;
      return false;
    }
  }
}
</script>

<style lang="less" scoped>

</style>
