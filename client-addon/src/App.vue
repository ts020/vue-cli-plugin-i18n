<template>
  <div class="project-localizations">
    <ContentView :title="$t('views.project-localizations.title')">
      <template slot="actions">
        <VueSelect v-model="current" button-class="primary">
          <VueSelectButton
            v-for="locale in locales"
            :key="locale"
            :value="locale"
            :label="locale === defaultLocale ? `${locale} (default)` : locale"
          />
        </VueSelect>
        <VueButton
          class="primary round"
          label="Add locale"
          icon-left="add"
          @click="onClickAddLocale"
        />
      </template>

      <div class="container">
        <div class="shell">
          <div class="content">
            <div class="localization-header">
              <div class="path">
                <div class="title">
                  <h3>Paths</h3>
                </div>
                <div class="actions">
                  <VueButton
                    class="icon-button"
                    :icon-left="sortIcon"
                    @click="onClickSort"
                  />
                </div>
              </div>
              <div class="locale">
                <div class="title">
                  <h3>Messages</h3>
                </div>
              </div>
            </div>

            <ul class="localization-body">
              <li class="item" v-for="path in currentPaths" :key="path">
                <PathField :path="path"/>
                <MessageField :path="path" :message="getLocaleMessage(path)"/>
              </li>
              <li class="item add">
                <PathAddField/>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </ContentView>

    <VueModal
      v-if="modal"
      title="Add a locale"
      class="medium"
      @close="modal = false"
    >
      <div class="default-body">
        <VueFormField
          class="locale-field"
          title="Input a locale"
          :subtitle="error"
        >
          <VueInput
            v-model="locale"
            @input="onInputLocale"
            placeholder="e.g: ja"
            :status="status"
            :icon-right="icon"
            ref="locale"
          />
        </VueFormField>
         <div class="vue-ui-text info banner">
          <VueIcon icon="info" class="big"/>
          <span>Recommended locale name according to BCP47</span>
        </div>
        <div class="more-info">
          <a target="_blank" href="https://tools.ietf.org/html/bcp47">
            <VueIcon icon="open_in_new" class="big"/>
            See the BCP47 ...
          </a>
        </div>
      </div>
      <div slot="footer" class="actions">
        <VueButton
          class="primary"
          :disabled="disable"
          @click="onAddLocale"
        >
          Add
        </VueButton>
      </div>
    </VueModal>
  </div>
</template>

<script>
import PathField from './components/PathField.vue'
import PathAddField from './components/PathAddField.vue'
import MessageField from './components/MessageField.vue'

export default {
  name: 'app',

  components: {
    PathField,
    PathAddField,
    MessageField
  },

  data: () => ({
    modal: false,
    error: '',
    icon: '',
    status: '',
    locale: ''
  }),

  sharedData () {
    return mapSharedData('vue-i18n-', {
      order: 'order',
      current: 'current',
      defaultLocale: 'defaultLocale',
      locales: 'locales',
      localeMessages: 'localeMessages',
      localePaths: 'localePaths'
    })
  },

  computed: {
    currentPaths () {
      if (!this.current || !this.localePaths) { return [] }
      const paths = this.localePaths[this.current]
      return this.order === 'desc' ? paths.reverse() : paths.sort()
    },

    currentMessages () {
      return (this.current && this.localeMessages)
        ? this.localeMessages[this.current]
        : {}
    },

    sortIcon () {
      return this.order === 'desc' ? 'arrow_drop_down' : 'arrow_drop_up'
    },

    disable () {
      return !this.locale || this.error.length > 0
    }
  },

  methods: {
    getLocaleMessage (path) {
      const paths = path.split('.')
      let p = null
      let value = this.currentMessages
      while ((p = paths.shift())) { value = value[p] }
      return value
    },

    onClickSort () {
      if (this.order === 'asc') {
        this.order = 'desc'
      } else {
        this.order = 'asc'
      }
    },

    onClickAddLocale () {
      this.modal = true
      this.locale = ''
      this.status = ''
      this.icon = ''
      setTimeout(() => {
        this.$refs.locale.focus()
      }, 500)
    },

    onInputLocale () {
      if (~this.locales.indexOf(this.locale)) {
        this.error = 'already exist a locale'
        this.icon = 'error'
        this.status = 'danger'
      } else {
        this.error = ''
        if (this.locale.length > 0) {
          this.icon = 'check_circle'
          this.status = 'success'
        } else {
          this.icon = ''
          this.status = ''
        }
      }
    },

    onAddLocale () {
      this.$callPluginAction('add-locale-action', {
        locale: this.locale
      }).then(data => {
        this.modal = false
      }).catch(error => {
        this.error = error.message
        this.icon = 'error'
        this.status = 'danger'
      })
    }
  }
}
</script>

<style scoped>
.project-localizations {
  width: 100%;
  height: 100%;
  display: grid;
  grid-template-columns: 1fr;
  grid-template-rows: 1fr;
  grid-template-areas: "content";
}

.container {
  grid-area: content;
  overflow-x: hidden;
  overflow-y: auto;
}

.shell {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  height: 100%;
}

.content {
  flex: 100% 1 1;
  height: 0;
  overflow-x: hidden;
  overflow-y: auto;
}
.action-bar {
  padding: 12px;
  background: white;
  display: flex;
  flex-direction: row;
  align-items: center;
  position: relative;
}

.localization-header {
  display: grid;
  grid-gap: 6px;
  grid-template-columns: 6fr 6fr;
}

.localization-header .path,
.localization-header .locale {
  display: flex;
}

.localization-header .path {
  padding: 12px;
  background: #f8fbfb;
  border-bottom-color: #42b983;
  border-bottom-style: solid;
  border-bottom-width: 1px;
}

.localization-header .locale {
  padding: 12px;
  background: #f8fbfb;
  border-bottom-color: #42b983;
  border-bottom-style: solid;
  border-bottom-width: 1px;
}

.localization-header .path .title {
  margin-right: auto;
}

.localization-header .path h3,
.localization-header .locale h3 {
  margin-bottom: 0;
}

ul.localization-body {
  list-style-type: none;
  padding-left: 0;
  margin-top: 0;
}

ul.localization-body li {
  display: grid;
  grid-gap: 6px;
  grid-template-columns: 6fr 6fr;
}

.project-localizations .locale-field {
  min-height: 84px;
}

.project-localizations .more-info {
  color: #42b983;
  padding: 12px;
}

.project-localizations .more-info .vue-ui-icon >>> svg {
  fill: #42b983;
}

.vue-ui-dark-mode .localization-header .path,
.vue-ui-dark-mode .localization-header .locale {
  background: #1d2935;
}
</style>
