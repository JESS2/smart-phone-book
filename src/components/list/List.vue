<template>
  <div class="list">
    <div class="listHeader" :class="{ noDisplay: openOtherPage === true }">
      <div class="headerTop">
      <span class="tagSpan" @click="openTagFunc">태그</span>
      <span class="tagSpan" @click="openFavoriteFunc">즐겨찾기</span>
      <span class="rightIcon">
        <b-dropdown right class="dropdown">
          <b-dropdown-item @click="openCreateFunc">연락처 추가</b-dropdown-item>
          <b-dropdown-item @click="ready">명함 추가</b-dropdown-item>
          <b-dropdown-divider></b-dropdown-divider>
          <b-dropdown-item @click="ready">연락처 가져오기</b-dropdown-item>
          <b-dropdown-item @click="ready">연락처 내보내기</b-dropdown-item>
          <b-dropdown-divider></b-dropdown-divider>
          <b-dropdown-item href="https://soda-phonebook.ga/api/logout_processing">로그아웃</b-dropdown-item>
        </b-dropdown>
      </span>
      </div>
      <h2>{{ title }}</h2>
      <div class="search">
        <i class="fa fa-search"></i>
        <input type="text" placeholder="검색" v-model="searchContent" @keyup="inputKeyup" id="searchId">
      </div>
    </div>

    <div class="listBody" :class="{ noDisplay: openOtherPage === true }">
      <ul v-if="contact.type === 'ME'" class="myContact" v-for="contact in nameSortList">
        <!-- 나 -->
        <li @click="openDetailFunc(contact.id)">
          <i class="fa fa-user-circle"></i><h5>{{ contact.name }}</h5>
        </li>
      </ul>
      <ul v-if="contact.type !== 'ME'" :class="{ favorites: contact.type === 'FAVORITED' }" v-for="contact in nameSortList" :key="contact.id">
        <!-- 사람들 연락처 -->
        <li @click="openDetailFunc(contact.id)">
          {{ contact.name }}<i class="fa fa-star" v-if="contact.type === 'FAVORITED'"></i>
        </li>
      </ul>
    </div>

    <favorite-component :show="openFavorite" @close="openFavorite = false"></favorite-component>
    <tag-component :show="openTag" @close="openTag = false"></tag-component>
    <detail-component :show="openDetail" :userId="selectedUserId" :root="'list'" @close="openDetail = false"></detail-component>
    <create-component :show="openCreate" @close="openCreate = false"></create-component>
    <confirm-modal :show="openConfirmModal" :content="confirmContent" @close="openConfirmModal = false"></confirm-modal>
  </div>
</template>

<script>
import FavoriteComponent from '../favorite/Favorite'
import TagComponent from '../tag/Tag'
import DetailComponent from '../detail/Detail'
import CreateComponent from '../create/Create'
import ConfirmModal from '../../utilities/confirmModal/ConfirmModal'
import ConfirmData from '../../utilities/confirmModal/ConfirmData.json'

export default {
  name: 'List',
  data () {
    return {
      title: '연락처',
      openDetail: false,
      openCreate: false,
      openTag: false,
      openFavorite: false,
      contactData: [],
      selectedUserId: 0,
      searchContent: "",
      currentUrl: window.location.href,
      confirmContent: {},
      openConfirmModal: false,
    }
  },
  mounted () {
    console.log('mounted!!!!!!!!!!!', new Date())
    // console.log('contactData', this.contactData);
    setTimeout(() => {
      this.getContactList();
      this.init();
    }, 500)
    console.log(this.currentUrl)
  },
  watch: {
    currentUrl () {
      console.log('current Url ---')
    },
    openDetail () {
      this.getContactList();
      window.scrollTo(0,0);
      console.log('show opendetail')
    },
    openCreate () {
      this.getContactList();
      window.scrollTo(0,0);
      console.log('show openCreate')
    }
  },
  computed: {
    openOtherPage () {
      if (this.openDetail === true || this.openCreate === true || this.openTag === true || this.openFavorite === true) {
        return true;
      }
      return false;
    },
    contactFilteredList () {
      return this.contactData.filter(item => {
        // 검색어에 번호 포함
        for (let i=0; i<item.digits.length; i++) {
          let number = item.digits[i].numbers.first + item.digits[i].numbers.second + item.digits[i].numbers.third
          if (number.includes(this.searchContent.replace(/-/gi, ""))) {
            return true;
          }
        }
        // 검색어에 이름 포함
        if (item.name.toUpperCase().includes(this.searchContent.toUpperCase())) {
          return true;
        }
      })
      return this.contactData;
    },
    nameSortList () {
      /* 이름순으로 정렬 */
      return this.contactFilteredList.sort((a, b) => {
        return a.name < b.name ? -1 : a.name > b.name ? 1 : 0;
      });
    },
  },
  methods: {
    ready () {
      console.log('ready')
      this.openConfirmModal = true;
      this.confirmContent = ConfirmData['ready'];
      console.log(this.openConfirmModal)
    },
    loginFunc () {
      this.$emit('login', false);
    },
    openTagFunc () {
      this.openTag = true;
    },
    openFavoriteFunc () {
      console.log('즐겨찾기 열기')
      this.openFavorite = true;
    },
    openDetailFunc (userId) {
      this.openDetail = true;
      this.selectedUserId = userId;
    },
    openCreateFunc () {
      this.openCreate = true;
    },
    inputKeyup () {
      this.searchContent = $('#searchId').val();
    },
    // 연락처 리스트 가져오기
    getContactList () {
      console.log('getContactList')
      this.$http.get(`/contacts`, {
      }).then((result => {
          this.contactData = result.data;
          console.log('api 호출', this.contactData)
        }))
        .catch(error => {
          alert('오류가 발생했습니다.')
        })
    },
    init () {
      // this.addTag("가족");
      // this.addTag("친구");
      // this.addTag("학교");
      // this.addTag("직장");
      // this.addCategory('DIGIT', '휴대전화')
      // this.addCategory('DIGIT', '집')
      // this.addCategory('DIGIT', '직장')
      // this.addCategory('DIGIT', '팩스')
      // this.addCategory('DIGIT', '기타')
      // this.addCategory('URL', '개인')
      // this.addCategory('URL', '직장')
      // this.addCategory('URL', '기타')
      // this.addCategory('EMAIL', '개인')
      // this.addCategory('EMAIL', '직장')
      // this.addCategory('EMAIL', '기타')
      // this.addCategory('DATE', '생일')
      // this.addCategory('DATE', '기념일')
      // this.addCategory('DATE', '기타')
      // this.addCategory('ADDRESS', '집')
      // this.addCategory('ADDRESS', '직장')
      // this.addCategory('ADDRESS', '기타')
    },
    addTag (_name) {
      console.log('태그 추가')
      this.$http.post(`/tags/`, {
        name: _name
      }).then((result => {
        }))
        .catch(error => {
          alert('오류가 발생했습니다.')
        })
    },
    addCategory (_categoryType, _categoryName) {
      console.log('add Category')
      this.$http.post(`/categories/${_categoryType}`, {
        name: _categoryName,
        type: _categoryType
      }).then((result => {
        }))
        .catch(error => {
          alert('오류가 발생했습니다.')
        })
    }
  },
  components: {
    DetailComponent,
    CreateComponent,
    TagComponent,
    FavoriteComponent,
    ConfirmModal,
  }
}
</script>

<style lang="scss">
    @import "List.scss"
</style>
