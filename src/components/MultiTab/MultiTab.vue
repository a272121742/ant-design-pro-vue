<!--
<template>
  <div style="margin: -23px -24px 24px -24px">
    &lt;!&ndash;<a-dropdown :trigger="['contextmenu']" overlayClassName="multi-tab-menu-wrapper">
      <a-menu slot="overlay">
        <a-menu-item key="1">1st menu item</a-menu-item>
        <a-menu-item key="2">2nd menu item</a-menu-item>
        <a-menu-item key="3">3rd menu item</a-menu-item>
      </a-menu>
    </a-dropdown>&ndash;&gt;
    <a-tabs
      hideAdd
      v-model="activeKey"
      type="editable-card"
      :tabBarStyle="{ background: '#FFF', margin: 0, paddingLeft: '16px', paddingTop: '1px' }"
      @edit="onEdit"
    >
      <a-tab-pane v-for="page in pages" :style="{ height: 0 }" :tab="page.meta.title" :key="page.fullPath" :closable="pages.length > 1">
      </a-tab-pane>
      <template slot="renderTabBar" slot-scope="props, DefaultTabBar">
        <component :is="DefaultTabBar" {...props} />
      </template>
    </a-tabs>
  </div>
</template>
-->

<script>
export default {
  name: 'MultiTab',
  data () {
    return {
      fullPathList: [],
      pages: [],
      activeKey: '',
      newTabIndex: 0
    }
  },
  created () {
    this.pages.push(this.$route)
    this.fullPathList.push(this.$route.fullPath)
    this.selectedLastPath()
  },
  methods: {
    onEdit (targetKey, action) {
      console.log('onEdit', targetKey, action)
      this[action](targetKey)
    },
    remove (targetKey) {
      this.pages = this.pages.filter(page => page.fullPath !== targetKey)
      this.fullPathList = this.fullPathList.filter(path => path !== targetKey)
      // 跳转到最后一个还存在的标签页
      this.selectedLastPath()
    },
    selectedLastPath () {
      this.activeKey = this.fullPathList[this.fullPathList.length - 1]
    },

    // content menu
    closeThat (e) {
      console.log('close that', e)
      this.remove(e)
    },
    closeLeft (e) {
      // TODO
      console.log('close left', e)
      const index = this.fullPathList.indexOf(e)
      if (index > -1) {
        this.fullPathList.splice(index, -1)
      }
    },
    closeRight (e) {
      console.log('close right', e)
    },
    closeAll (e) {
      console.log('close all', e)
    },
    closeMenuClick ({ key, item, domEvent }) {
      console.log('key', key)
      console.log('item', item.$attrs['data-vkey'])
      const vkey = domEvent.target.getAttribute('data-vkey')
      switch (key) {
        case 'close-right':
          this.closeRight(vkey)
          break
        case 'close-left':
          this.closeLeft(vkey)
          break
        case 'close-all':
          this.closeAll(vkey)
          break
        default:
        case 'close-that':
          this.closeThat(vkey)
          break
      }
    },
    renderTabPaneMenu (e) {
      return (
        <a-menu {...{ on: { click: this.closeMenuClick } }}>
          <a-menu-item key="close-that" data-vkey={e}>关闭当前标签</a-menu-item>
          <a-menu-item key="close-right" data-vkey={e}>关闭右侧</a-menu-item>
          <a-menu-item key="close-left" data-vkey={e}>关闭左侧</a-menu-item>
          <a-menu-item key="close-all" data-vkey={e}>关闭全部</a-menu-item>
        </a-menu>
      )
    },
    // render
    renderTabPane (title, keyPath) {
      const menu = this.renderTabPaneMenu(keyPath)

      return (
        <a-dropdown overlay={menu} trigger={['contextmenu']}>
          <span style={{ userSelect: 'none' }}>{ title }</span>
        </a-dropdown>
      )
    }
  },
  watch: {
    '$route': function (newVal) {
      this.activeKey = newVal.fullPath
      if (this.fullPathList.indexOf(newVal.fullPath) < 0) {
        this.fullPathList.push(newVal.fullPath)
        this.pages.push(newVal)
      }
    },
    activeKey: function (newPathKey) {
      console.log('activeKey', newPathKey)
      this.$router.push({ path: newPathKey })
    }
  },
  render () {
    const { onEdit, $data: { pages } } = this
    const panes = pages.map(page => {
      return (<a-tab-pane style={{ height: 0 }} tab={this.renderTabPane(page.meta.title, page.fullPath)} key={page.fullPath} closable={pages.length > 1}></a-tab-pane>)
    })

    return (
      <div style="margin: -23px -24px 24px -24px">
        <a-tabs
          hideAdd
          type={'editable-card'}
          v-model={this.activeKey}
          tabBarStyle={{ background: '#FFF', margin: 0, paddingLeft: '16px', paddingTop: '1px' }}
          {...{ on: { edit: onEdit } }}>
          {panes}
        </a-tabs>
      </div>
    )
  }
}
</script>