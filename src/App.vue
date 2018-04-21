<template>
  <div id="app"
    @touchstart="onTouchStart"
    @touchend="onTouchEnd">
    <newshow-header
      :items="navItems"
      :activeItem="activeItem"
      @route-to="routeTo"
      @route-to-current-user-profile="routeTo(currentUserPath)"
      @login="login"
      @logout="logout"
      @toggle-sidebar="toggleSidebar"
      ref="header"
      />
    <newshow-sidebar
      :isShow="isShowSidebar"
      :items="navItems"
      :activeItem="activeItem"
      @route-to="routeTo"
      @toggle-sidebar="toggleSidebar"
      />
    <router-view :style="{minHeight: mainMinHeight}" class="content"/>
    <newshow-footer :copyrightText="copyrightText" ref="footer"/>
    <loading-snippet v-show="isLoading" class="loading-snippet"/>
  </div>
</template>

<script>
// use <newshow-header> rather than <header>
// to avoide conflict with reserve tag name
import newshowHeader from 'components/header/header.vue'
import newshowSidebar from 'components/header/sidebar.vue'
import newshowFooter from 'components/footer/footer.vue'
import loadingSnippet from 'components/common/loading-snippet.vue'
import { mapGetters } from 'Vuex'

export default {
  name: 'App',
  components: {
    newshowHeader,
    newshowSidebar,
    newshowFooter,
    loadingSnippet
  },
  data () {
    return {
      navItems: [
        {title: '时间轴', routeName: 'photo-wall'},
        {title: '相册', routeName: 'photo-album'},
        {title: '最佳', routeName: 'best-photo'}
      ],
      touchStart: {},
      isShowSidebar: false
    }
  },
  mounted () {
    this.$router.afterEach(() => {
      this.isShowSidebar = false
    })
  },
  computed: {
    ...mapGetters(['isLoading']),
    activeItem () {
      const curRouteName = this.$route.name
      const activeItems = this.navItems.filter(item => {
        // 默认主页home为图片墙
        if (curRouteName === 'home' && item.routeName === 'photo-wall') {
          return true
        }
        return item.routeName === curRouteName
      })
      return activeItems.length ? activeItems[0] : null
    },
    // TODO: 当前已登入用户的路由path
    currentUserPath () {
      return 'user/1'
    },
    copyrightText () {
      // 其实这里应该用服务器端时间, 而不是本地时间
      return `Copyright © 2015-${new Date().getFullYear()} qiniu`
    },
    // 设置内容区最小高度, 防止footer“挤”上来
    mainMinHeight () {
      const heightSum = document.body.clientHeight
      console.log(heightSum)
      // root font size
      const rootFontSize = +getComputedStyle(document.querySelector('html')).fontSize.replace(/px$/, '')
      // acorrding to `variables.scss`
      console.log(rootFontSize)
      console.log()
      return heightSum - // visible area sum height
        3.5 * rootFontSize - // header.clientHeight 3.5rem
        3 * rootFontSize + // footer.clientHeight 3rem
        'px' // remember to apply unit
    }
  },
  methods: {
    routeTo (routerName) {
      this.$router.push({
        name: routerName
      })
    },
    login () {
      this.$router.push({
        name: 'login'
      })
    },
    // TODO: 注销当前用户操作
    logout () {
    },
    toggleSidebar (boolean) {
      this.isShowSidebar = typeof boolean === 'boolean' ? boolean : !this.isShowSidebar
    },
    // side swipe
    onTouchStart (e) {
      this.touchStart = {
        x: e.changedTouches[0].clientX,
        y: e.changedTouches[0].clientY
      }
    },
    onTouchEnd (e) {
      const dx = e.changedTouches[0].clientX - this.touchStart.x
      const dy = e.changedTouches[0].clientY - this.touchStart.y
      if (Math.abs(dx) > Math.abs(dy) && Math.abs(dx) > 40) {
        if (dx > 0 && this.touchStart.x <= 80) {
          this.toggleSidebar(true)
        } else {
          this.toggleSidebar(false)
        }
      }
    }
  }
}
</script>

<style lang="scss">
@import './styles/common.scss';
@import './styles/fonts.css';

.content {
  padding: 0;
  box-sizing: border-box;
  padding-top: $globalPadding * 1.5;
  padding-bottom: $globalPadding * 1.5;
}
.loading-snippet {
  top: $navbarHeight / 4 * 3;
}
@media (max-width: $MQMobile) {
  .content {
    padding-top: $globalPadding + 0.2rem;
    padding-bottom: $globalPadding + 0.2rem;
  }
}
</style>