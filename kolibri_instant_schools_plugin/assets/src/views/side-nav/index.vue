<template>

  <div @keydown.esc="toggleNav" ref="sideNav">
    <transition name="side-nav">
      <div
        v-show="navShown"
        class="side-nav"
        :style="{ width: `${this.width}px` }"
      >
        <div
          class="side-nav-header"
          :style="{ height: headerHeight + 'px', width: `${width}px`, paddingTop: mobile ? '4px' : '8px' }"
        >
          <ui-icon-button
            ref="toggleButton"
            :aria-label="$tr('closeNav')"
            type="secondary"
            color="white"
            size="large"
            icon="close"
            @click="toggleNav"
          />
          <span class="side-nav-header-name">Kolibri</span>
        </div>

        <div
          class="side-nav-scrollable-area"
          :style="{ top: `${headerHeight}px`, width: `${width}px` }"
        >
          <ui-menu
            class="side-nav-scrollable-area-menu"
            role="navigation"
            :options="menuOptions"
            :hasIcons="true"
            :aria-label="$tr('navigationLabel')"
            @select="navigate"
          />

          <div class="side-nav-scrollable-area-footer">
            <logo class="side-nav-scrollable-area-footer-logo"/>
            <div class="side-nav-scrollable-area-footer-info">
              <p>{{ footerMsg }}</p>
              <!-- Not translated -->
              <p>© 2017 Learning Equality</p>
            </div>
          </div>
        </div>

      </div>
    </transition>

    <div
      v-show="navShown"
      class="side-nav-backdrop"
      @click="toggleNav"
    >
    </div>
  </div>

</template>


<script>

  import values from 'lodash/values';
  import {
    isUserLoggedIn,
    isSuperuser,
    isAdmin,
    isCoach,
    canManageContent,
  } from 'kolibri.coreVue.vuex.getters';
  import { kolibriLogout } from 'kolibri.coreVue.vuex.actions';
  import { TopLevelPageNames } from 'kolibri.coreVue.vuex.constants';
  import responsiveWindow from 'kolibri.coreVue.mixins.responsiveWindow';
  import responsiveElement from 'kolibri.coreVue.mixins.responsiveElement';
  import uiMenu from './keen-menu-port';
  import uiIconButton from 'keen-ui/src/UiIconButton';
  import logo from 'kolibri.coreVue.components.logo';

  export default {
    name: 'sideNav',
    components: {
      uiMenu,
      uiIconButton,
      logo,
    },
    mixins: [responsiveWindow, responsiveElement],
    $trs: {
      navigationLabel: 'Main user navigation',
      learn: 'Learn',
      facility: 'Facility',
      coach: 'Coach',
      device: 'Device',
      signIn: 'Sign in',
      profile: 'Profile',
      signOut: 'Sign out',
      about: 'About',
      closeNav: 'Close navigation',
      poweredBy: 'Kolibri {version}',
    },
    props: {
      topLevelPageName: {
        type: String,
        validator(value) {
          if (!value) {
            return true;
          }
          return values(TopLevelPageNames).includes(value);
        },
      },
      navShown: {
        type: Boolean,
        required: true,
      },
      headerHeight: {
        type: Number,
        required: true,
      },
      width: {
        type: Number,
        required: true,
      },
    },
    data() {
      return {
        previouslyFocusedElement: null,
      };
    },
    computed: {
      mobile() {
        return this.windowSize.breakpoint < 2;
      },
      footerMsg() {
        return this.$tr('poweredBy', { version: __version });
      },
      menuOptions() {
        const options = [
          {
            label: this.$tr('learn'),
            active: this.pageIsActive(TopLevelPageNames.LEARN),
            icon: 'school',
            href: '/learn',
          },
        ];
        if (this.isCoach || this.isAdmin || this.isSuperuser) {
          options.push({
            label: this.$tr('coach'),
            active: this.pageIsActive(TopLevelPageNames.COACH),
            icon: 'assessment',
            href: '/coach',
          });
        }
        if (this.isAdmin || this.isSuperuser) {
          options.push({
            label: this.$tr('facility'),
            active: this.pageIsActive(TopLevelPageNames.MANAGE),
            icon: 'settings_input_antenna',
            href: '/management/facility',
          });
        }
        if (this.canManageContent || this.isSuperuser) {
          options.push({
            label: this.$tr('device'),
            active: this.pageIsActive(TopLevelPageNames.DEVICE),
            icon: 'tablet_mac',
            href: '/management/device',
          });
        }
        options.push({ type: 'divider' });
        if (this.isUserLoggedIn) {
          options.push({
            label: this.$tr('profile'),
            active: this.pageIsActive(TopLevelPageNames.USER),
            icon: 'account_circle',
            href: '/user',
          });
          options.push({
            label: this.$tr('signOut'),
            icon: 'exit_to_app',
            action: this.signOut,
          });
        } else {
          options.push({
            label: this.$tr('signIn'),
            icon: 'exit_to_app',
            href: '/user',
          });
        }
        options.push({ type: 'divider' });
        return options;
      },
    },
    watch: {
      navShown(isShown) {
        this.$nextTick(() => {
          if (isShown) {
            window.addEventListener('focus', this.containFocus, true);
            this.previouslyFocusedElement = document.activeElement;
            this.$refs.toggleButton.$el.focus();
          } else {
            window.removeEventListener('focus', this.containFocus, true);
            this.previouslyFocusedElement.focus();
          }
        });
      },
    },
    methods: {
      navigate(option) {
        if (option.href) {
          window.location.href = option.href;
        } else if (option.action) {
          option.action();
        }
      },
      toggleNav() {
        this.$emit('toggleSideNav');
      },
      pageIsActive(pageName) {
        return this.topLevelPageName === pageName;
      },
      containFocus(event) {
        if (event.target === window) {
          return;
        }
        if (!this.$refs.sideNav.contains(event.target)) {
          this.$refs.toggleButton.$el.focus();
        }
      },
    },
    vuex: {
      actions: { signOut: kolibriLogout },
      getters: {
        session: state => state.core.session,
        isUserLoggedIn,
        isSuperuser,
        isAdmin,
        isCoach,
        canManageContent,
      },
    },
  };

</script>


<style lang="stylus" scoped>

  @require '~kolibri.styles.definitions'

  // matches angular material's spec
  $side-nav-box-shadow = 0 2px 4px -1px rgba(0, 0, 0, 0.2), 0 4px 5px 0 rgba(0, 0, 0, 0.14), 0 1px 10px 0 rgba(0, 0, 0, 0.12)

  // matches keen-ui toolbar's spec
  $side-nav-header-box-shadow = 0 0 2px rgba(black, 0.12), 0 2px 2px rgba(black, 0.2)

  .side-nav
    position: fixed
    top: 0
    bottom: 0
    z-index: 16
    color: $core-text-default
    background: $core-bg-light
    box-shadow: $side-nav-box-shadow

  .side-nav-enter
    transform: translate3D(-100%, 0, 0)

  .side-nav-enter-active
    transition: all 0.2s ease-in-out

  .side-nav-enter-to
    transform: translate3D(0, 0, 0)

  .side-nav-leave
    transform: translate3D(0, 0, 0)

  .side-nav-leave-active
    transition: all 0.2s ease-in-out

  .side-nav-leave-to
    transform: translate3D(-100%, 0, 0)

  .side-nav-header
    position: fixed
    top: 0
    left: 0
    z-index: 17
    font-size: 14px
    text-transform: uppercase
    background-color: $core-text-default
    box-shadow: $side-nav-header-box-shadow

  .side-nav-header-name
    margin-left: 8px
    vertical-align: middle
    color: $core-bg-light
    font-weight: bold
    font-size: 18px

  .side-nav-scrollable-area
    position: fixed
    left: 0
    bottom: 0
    overflow: auto

  .side-nav-scrollable-area-menu
    background: $core-bg-light

  .side-nav-scrollable-area-footer
    color: $core-text-annotation
    padding: 16px

  .side-nav-scrollable-area-footer-logo
    width: 40px
    height: 40px

  .side-nav-scrollable-area-footer-info
    display: inline-block
    margin-left: 8px
    width: 140px
    font-size: 10px
    line-height: 16px
    p
      margin: 0

  .side-nav-backdrop
    position: fixed
    top: 0
    left: 0
    width: 100%
    height: 100%
    background: rgba(0, 0, 0, 0.7)
    transition: opacity 0.3s ease
    background-attachment: fixed
    z-index: 15

</style>


<style lang="stylus">

  @require '~kolibri.styles.definitions'

  // Customize Keen UI Menu option
  .side-nav-scrollable-area-menu
    .ui-menu
      max-height: none
      padding: 0
      background: $core-bg-light
      border: none

    .ui-menu-option
      &:not(.is-divider)
        padding-top: 4px
        padding-bottom: 4px

        .ui-menu-option-text
          overflow: visible
          white-space: normal
          color: $core-text-default
          font-size: 14px

        .ui-menu-option-icon
          color: $core-text-default
          font-size: 1.2em

        &.is-active
          .ui-menu-option-text
            color: $core-accent-color
            font-weight: bold
            opacity: 1

          .ui-menu-option-icon
            color: $core-accent-color

      &.is-divider
        margin-top: 0
        margin-bottom: 0

</style>
