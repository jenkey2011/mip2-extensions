<template>
  <div>
    <slot
      :user="userInfo"
      name="content"/>
  </div>
</template>

<script>

import util from './util'

let viewer = window.MIP.viewer
let fn = window.MIP.util.fn

export default {
  props: {
    config: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      /**
       * 用户信息
       *
       * @type {Object}
       */
      userInfo: null,

      /**
       * 会话标识
       *
       * @type {string}
       */
      sessionId: null,

      /**
       * 默认为未登录
       *
       * @type {boolean}
       */
      isLogin: false
    }
  },
  created () {
    // 检查配置数据
    this.checkConfig()
  },
  mounted () {
    // 熊掌号sdk
    let url = 'https://xiongzhang.baidu.com/sdk/c.js?appid=' + this.config.appid

    util.loadJS(
      url,
      () => {
        // 如果是自动登录，在检查完用户信息后没有登录时要求立即登录
        if (this.config.autologin) {
          return this.getUserInfo().then(() => {
            if (!this.isLogin) {
              this.login()
            }
          })
        }
        this.getUserInfo()
        this.bindEvents()
      },
      () => {
        throw new Error('加载资源出错')
      }
    )
  },
  methods: {
    bindEvents () {
      this.$element.customElement.addEventAction('login', () => {
        this.login()
      })
      this.$element.customElement.addEventAction('logout', () => {
        this.logout()
      })
    },
    /**
     * 检查配置
     */
    checkConfig () {
      let config = this.config
      let hasError = false

      if (!config.clientId) {
        this.error('组件必选属性 clientId 为空')
        hasError = true
      }
      if (!config.endpoint) {
        this.error('组件必选属性 endpoint 为空')
        hasError = true
      } else if (!/^(https:)?\/\//.test(config.endpoint)) {
        this.error('组件必选属性 endpoint 必须以 https:// 或者 // 开头')
        hasError = true
      }

      // 如果有 mip-bind 则必须有组件id -- 之后补充
      if (typeof MIP.setData === 'function' && !config.id) {
        this.error('和 mip-bind 配合使用必须设置登录组件 id')
        hasError = true
      }

      if (hasError) {
        throw new TypeError('[mip-inservice-login] 组件参数检查失败')
      }
    },
    /**
     * 输出错误信息到控制台
     *
     * @param {string} text 输出文本
     */
    error (text) {
      console.error('[mip-inservice-login] ', text, this)
    },
    /**
     * 用户登录
     */
    login () {
      if (this.isLogin) {
        return
      }
      let sourceUrl = util.getSourceUrl()

      window.cambrian && window.cambrian.authorize({
        data: {
          redirect_uri: sourceUrl,
          scope: 1,
          pass_no_login: 0,
          state: JSON.stringify({
            url: sourceUrl,
            r: Date.now()
          }),
          ifSilent: false,
          client_id: this.config.clientId
        },
        success (data) {
          viewer.open(
            util.getRedirectUrl(sourceUrl, data.result, 'query'),
            { isMipLink: true, replace: true }
          )
        },
        fail (data) {
          console.error(data.msg)
        }
      })
    },

    /**
     * 用户退出
     */
    logout () {
      let self = this

      if (!self.isLogin) {
        return
      }

      util.post(self.config.endpoint, {
        type: 'logout'
      }).then(function (res) {
        // 清空 sessionId
        util.store.remove(self.config.endpoint)

        if (res.data && res.data.url) {
          viewer.open(res.data.url, { isMipLink: true })
        } else {
          // 是否，需要补充多一点信息
          self.loginHandle('logout', false)
          // 设置数据
          self.setData()
        }
      }).catch(function (data) {
        // 清空 sessionId
        util.store.remove(self.config.endpoint)

        self.loginHandle('logout', false)
      })
    },

    /**
     * 登录统一处理
     *
     * @param  {string}  name    事件名称
     * @param  {boolean} isLogin 是否登录
     * @param  {Object|undefined}  data    用户数据
     */
    loginHandle (name, isLogin, data) {
      this.isLogin = isLogin
      this.userInfo = data || null
      this.trigger(name)
    },

    /**
     * 触发事件
     *
     * @param  {string} name  事件名称
     */
    trigger (name) {
      let event = {
        userInfo: this.userInfo,
        sessionId: this.sessionId
      }
      viewer.eventAction.execute(name, this.$element, event)
      // this.$emit(name, event);
    },

    /**
     * 获取用户信息
     *
     * @returns {Promise} 用户信息
     */
    getUserInfo () {
      let data = {
        type: 'check'
      }
      let code
      let callbackurl

      code = util.getQuery('code')

      if (code) {
        try {
          callbackurl = JSON.parse(util.getQuery('state')).url
        } catch (e) {}
      }

      if (code && callbackurl) {
        data.code = code
        data.redirect_uri = callbackurl
        data.type = 'login'
      }

      let self = this
      return util.post(self.config.endpoint, data).then(res => {
        // 记录 sessionId 到 ls 中，修复在 iOS 高版本下跨域 CORS 透传 cookie 失效问题
        if (res.sessionId) {
          self.sessionId = res.sessionId
          util.store.set(self.config.endpoint, res.sessionId)
        }

        if (data.type === 'login') {
          if (res.status === 0 && fn.isPlainObject(res.data)) {
            self.loginHandle('login', true, res.data)
          } else {
            throw new Error('登录失败', res)
          }
        } else if (res.status === 0 && res.data) {
          self.loginHandle('login', true, res.data)
        }

        // 设置数据
        self.setData()
      }).catch(err => {
        if (data.type === 'login') {
          this.loginHandle('error', false)
          throw err
        }
      })
    },

    /**
     * 配合 mip-bind 设置数据
     */
    setData () {
      if (typeof MIP.setData !== 'function') {
        return
      }
      let id = (this.config.isGlobal ? '#' : '') + this.config.id

      // 设置源数据
      let data = {}

      data[id] = {
        isLogin: this.isLogin
      }

      // fix 因为直接使用 null 时 mip-bind 报错
      if (this.userInfo) {
        data[id].userInfo = this.userInfo
      }
      if (this.sessionId) {
        data[id].sessionId = this.sessionId
      }

      MIP.setData(data)
    }
  }
}
</script>
