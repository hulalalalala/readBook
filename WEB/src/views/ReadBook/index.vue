<template>
    <div style="height:100%;background-color: #E6E0D0;">
      <yd-navbar height="2.4rem" fontsize="1rem" :fixed="true" bgcolor="#333">
        <span slot="left" @click="back" style="font-size:1rem;">
            <yd-navbar-back-icon>返回</yd-navbar-back-icon>
        </span>
        <span slot="center" style="font-size:1rem;color:#d2d4e1;overflow: hidden;text-overflow: ellipsis;white-space: nowrap;">{{$route.query.name}}</span>
      </yd-navbar>
      <div class="content">
        <div><span ref="bookTitle">{{dataObj.title}}</span></div>
        <div v-html="dataObj.con">
        </div>
        <div>
          <span @click="last" :class="dataObj.lastPath.indexOf('html') < 0 ? 'noneClick' : ''">上一章</span>
          <span @click="showCatalog = true">目录</span>
          <span @click="next">下一章</span>
        </div>
      </div>
      <catalog-view :show="showCatalog" @onHandle="onHandle" :path="$route.query.url" @on-clickRead="clickRead"></catalog-view>
    </div>
</template>
<script>
import CatalogView from './Catalog'
export default {
  name: 'ReadBook',
  components: { CatalogView },
  methods: {
    back () {
      this.$router.go(-1)
      this.$dialog.loading.close()
    },
    onHandle (val) {
      this.showCatalog = val
    },
    clickRead (data) {
      this.axiosPost(data.path, true)
    },
    axiosPost (path, flag) {
      this.$dialog.loading.open('正在加载')
      this.$api['getBook.readBook'](
        {
          path: path
        }
      ).then(res => {
        if (res.msg.indexOf('OK') >= 0) {
          this.dataObj = res.data
          window.scrollTo(0, 0)
          this.$dialog.loading.close()
          if (this.$storage.getItem('books') !== null) {
            this.storageBooks = JSON.parse(this.$storage.getItem('books'))
          }
          this.$set(this.storageBooks, this.$route.query.name, {})
          this.$set(this.storageBooks[this.$route.query.name], 'bookZhangjieName', this.dataObj.title)
          this.$set(this.storageBooks[this.$route.query.name], 'bookZhangjiePath', path)
          this.$set(this.storageBooks[this.$route.query.name], 'img', this.$route.query.img)
          this.$set(this.storageBooks[this.$route.query.name], 'author', this.$route.query.author)
          this.setStorageBooks(this.storageBooks)
          if (flag) {
            this.$router.go(-1)
          }
          if (this.$storage.getItem('TOKEN_STR')) {
            // 更新书架数据
            this.updataBookShelf(path, this.$storage.getItem('TOKEN_STR'), this.$route.query.name)
          }
        }
      }).catch(error => {
        console.log(error)
        this.$dialog.loading.close()
        this.$dialog.toast('网络出错，请稍后重试！', 'error', 2000)
      })
    },
    next () {
      this.axiosPost(this.dataObj.nextPath)
    },
    last () {
      if (this.dataObj.lastPath.indexOf('html') < 0) { return }
      this.axiosPost(this.dataObj.lastPath)
    },
    setStorageBooks (obj) { // 存储阅读记录
      var checkedIdStr = JSON.stringify(obj)
      this.$storage.setItem('books', checkedIdStr)
    },
    updataBookShelf (path, userId, bookName) {
      this.$api['system.updataBookShelf']({
        path: path,
        userId: userId,
        bookName: bookName
      }).then(res => {
        if (res.msg.indexOf('OK') >= 0) {

        } else {
          this.$dialog.toast({
            mes: res.msg,
            timeout: 1500,
            icon: 'error'
          })
        }
      // eslint-disable-next-line handle-callback-err
      }).catch(error => {
        this.$dialog.toast({
          mes: '服务器异常，请稍后再试',
          timeout: 1500,
          icon: 'error'
        })
      })
    }
  },
  mounted () {
    let path = this.$route.query.path
    const storageBooks = JSON.parse(this.$storage.getItem('books'))
    if (storageBooks && !this.$route.query.isCatalog) {
      for (const key in storageBooks) {
        if (key !== 'undefined' && key === this.$route.query.name) {
          path = storageBooks[key].bookZhangjiePath
        }
      }
    }
    this.axiosPost(path)
    this.$store.state.botNav.showTopNav = false
    this.$store.state.botNav.showBottomNav = false
  },
  data () {
    return {
      dataObj: {},
      storageBooks: {},
      showCatalog: false
    }
  }
}
</script>
<style scoped>
  .content{
    display: flex;
    flex-direction: column;
    margin-top: 2rem;
    padding: 1rem 1rem;
    background-color: #E6E0D0;
  }
  .content div:nth-child(1){
    text-align: center;
    font-size: 1rem;
    font-weight: 600;
    margin-bottom: 1rem;
  }
  .content div:nth-child(2){
    font-size: .9rem;
    padding-bottom: 4rem;
  }
  .content div:nth-child(3){
    font-size: .9rem;
    position: fixed;
    bottom: 0;
    left: 0;
    background: #333;
    color: #d2d4e1;
    width: 100%;
    height: 3rem;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
    padding: 0 2rem;
  }
  .noneClick{
    color: #8b8282!important;
  }
</style>
