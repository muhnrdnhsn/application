<template>
  <b-nav class="side-bar-layout side-bar-bg side-bar-font side-bar-padding">
    <b-nav vertical>
      <!-- User profile info -->
      <b-nav-item link-classes="side-bar-color mt-5 ml-4">
        <!-- Username -->
        <div id="user-name">
          {{ username }}
        </div>
        <!-- Role -->
        <div id="role">
          <div v-if="role">
            Role: {{ role.charAt(0).toUpperCase() + role.slice(1) }}
          </div>
        </div>
      </b-nav-item>

      <!-- Logout -->
      <b-nav-item link-classes="side-bar-color mt-3 ml-4" @click="logout()">
        <div style="font-size: 0.77rem;">
          Logout
        </div>
      </b-nav-item>

      <div v-if="role === 'admin'">
        <!-- Upload Dataset -->
        <b-nav-item link-classes="side-bar-color mt-3 ml-4" @click="changeActiveElmtID('upload-dataset')">
          <div id="upload-dataset">
            Upload Dataset
          </div>
        </b-nav-item>
      </div>

      <!-- Label Dataset -->
      <b-nav-item link-classes="side-bar-color mt-3 ml-4" @click="changeActiveElmtID('label-dataset')">
        <div id="label-dataset">
          Label Dataset
        </div>
      </b-nav-item>

      <div v-if="role === 'editor' || role === 'admin'">
        <!-- Edit Dataset -->
        <b-nav-item link-classes="side-bar-color mt-3 ml-4" @click="changeActiveElmtID('edit-dataset')">
          <div id="edit-dataset">
            Edit Dataset
          </div>
        </b-nav-item>
      </div>

      <div v-if="role === 'editor' || role === 'admin'">
        <b-nav-item
          v-b-toggle.collapse-delete
          link-classes="side-bar-color mt-3 ml-4"
        >
          <div class="icon-text-wrapper">
            Delete Dataset
            <i class="ml-3 mt-1 fas fa-caret-down" />
          </div>
        </b-nav-item>

        <b-collapse id="collapse-delete" visible role="tabpanel">
          <!-- Delete Dataset -->
          <b-nav-item link-classes="side-bar-color mb-2 mt-3 ml-4" @click="changeActiveElmtID('delete-dataset')">
            <div class="icon-text-wrapper">
              <i class="mt-1 mr-3 fas fa-trash" />
              <div id="delete-dataset">
                Unlabeled Dataset
              </div>
            </div>
          </b-nav-item>

          <b-nav-item link-classes="side-bar-color mt-3 ml-4" @click="changeActiveElmtID('delete-labeled-dataset')">
            <div class="icon-text-wrapper">
              <i class="mt-1 mr-3 fas fa-tag" />
              <div id="delete-labeled-dataset">
                Labeled Dataset
              </div>
            </div>
          </b-nav-item>
        </b-collapse>
      </div>

      <div>
        <b-nav-item
          v-b-toggle.collapse-outputs
          link-classes="side-bar-color mt-3 ml-4"
        >
          <div class="icon-text-wrapper">
            Download Result
            <i class="ml-3 mt-1 fas fa-caret-down" />
          </div>
        </b-nav-item>

        <b-collapse id="collapse-outputs" visible role="tabpanel">
          <!-- COCO Output -->
          <b-nav-item link-classes="side-bar-color mt-3 ml-4" @click="changeActiveElmtID('coco-output')">
            <div class="icon-text-wrapper">
              <div class="mr-3 output-icon">
                {  }
              </div>
              <div id="coco-output">
                COCO File
              </div>
            </div>
          </b-nav-item>

          <!-- Pascal Output -->
          <b-nav-item link-classes="side-bar-color mt-3 ml-4" @click="changeActiveElmtID('pascal-output')">
            <div class="icon-text-wrapper">
              <div class="mr-3 output-icon">
                &lt; &gt;
              </div>
              <div id="pascal-output" style="margin-left: -5px;">
                Pascal VOC File
              </div>
            </div>
          </b-nav-item>

          <!-- All Images Output -->
          <b-nav-item link-classes="side-bar-color mt-3 ml-4" @click="changeActiveElmtID('all-images')">
            <div class="icon-text-wrapper">
              <i class="mt-1 mr-3 fas fa-image" />
              <div id="all-images">
                All Images
              </div>
            </div>
          </b-nav-item>
        </b-collapse>
      </div>

      <div v-if="role === 'admin'">
        <!-- User Management -->
        <b-nav-item
          v-b-toggle.collapse-user-management
          link-classes="side-bar-color mt-3 ml-4"
        >
          <div class="icon-text-wrapper">
            User Management
            <i class="ml-3 mt-1 fas fa-caret-down" />
          </div>
        </b-nav-item>

        <b-collapse id="collapse-user-management" visible role="tabpanel">
          <!-- Show All Users -->
          <b-nav-item link-classes="side-bar-color mt-3 ml-4" @click="changeActiveElmtID('show-all-users')">
            <div class="icon-text-wrapper">
              <i class="mt-1 mr-3 fas fa-users" />
              <div id="show-all-users">
                Show All Users
              </div>
            </div>
          </b-nav-item>

          <!-- Add User -->
          <b-nav-item link-classes="side-bar-color mt-3 ml-4" @click="changeActiveElmtID('add-user')">
            <div class="icon-text-wrapper">
              <i class="mt-1 mr-3 fas fa-plus-circle" />
              <div id="add-user">
                Add User
              </div>
            </div>
          </b-nav-item>

          <!-- Edit User -->
          <b-nav-item link-classes="side-bar-color mt-3 ml-4" @click="changeActiveElmtID('edit-user')">
            <div class="icon-text-wrapper">
              <i class="mt-1 mr-3 fas fa-pen" />
              <div id="edit-user">
                Edit User
              </div>
            </div>
          </b-nav-item>
          
          <!-- Delete User -->
          <b-nav-item link-classes="side-bar-color mt-3 ml-4" @click="changeActiveElmtID('delete-user')">
            <div class="icon-text-wrapper">
              <i class="mt-1 mr-3 fas fa-trash" />
              <div id="delete-user">
                Delete User
              </div>
            </div>
          </b-nav-item>
        </b-collapse>
      </div>

      
      <div>
        <b-nav-item
          v-b-toggle.collapse-settings
          link-classes="side-bar-color mt-3 ml-4"
        >
          <div class="icon-text-wrapper">
            Settings
            <i class="ml-3 mt-1 fas fa-caret-down" />
          </div>
        </b-nav-item>

        <b-collapse id="collapse-settings" visible role="tabpanel">
          <!-- Change Password -->
          <b-nav-item link-classes="side-bar-color mt-3 ml-4" @click="changeActiveElmtID('change-password')">
            <div class="icon-text-wrapper">
              <i class="mt-1 mr-3 fas fa-cog" />
              <div id="change-password">
                Change Password
              </div>
            </div>
          </b-nav-item>
          
          <!-- Change Username -->
          <b-nav-item link-classes="side-bar-color mt-3 ml-4 mb-5" @click="changeActiveElmtID('change-username')">
            <div class="icon-text-wrapper">
              <i class="mt-1 mr-3 fas fa-user" />
              <div id="change-username">
                Change Username
              </div>
            </div>
          </b-nav-item>
        </b-collapse>  
      </div>
    </b-nav>
  </b-nav>
</template>

<script>
import cookies from 'js-cookie'
export default {
  props: {
    username: {
      type: String,
      required: true
    },
    role: {
      type: String,
      required: true
    }
  },
  data () {
    return {
      activeElmtID: 'label-dataset'
    }
  },
  watch: {
    activeElmtID (newElmtID, oldElmtID) {
      
      
      this.setClass(oldElmtID, '')
      this.setClass(newElmtID, 'active')
    }
  },
  mounted () {
    this.onPathChangeHandler(window.location.pathname)
    // this.activeElmtID = 'label-dataset'
    // 
  },
  methods: {
    setClass (elmtID, className) {
      let elmt = document.getElementById(elmtID)
      elmt.className = className
    },
    changeActiveElmtID (elmtID) {
      this.activeElmtID = elmtID
      this.redirectRoute(elmtID)
    },
    redirectRoute (elmtID) {
      switch (elmtID) {
      case 'label-dataset':
        this.$nuxt.$router.replace({ path: '/main/label'})
        break
      case 'pascal-output':
        this.$nuxt.$router.replace({ path: '/main/pascal'})
        break
      case 'coco-output':
        this.$nuxt.$router.replace({ path: '/main/coco'})
        break
      case 'all-images':
        this.$nuxt.$router.replace({ path: '/main/all-images'})
        break
      case 'edit-dataset':
        this.$nuxt.$router.replace({ path: '/main/edit'})
        break
      case 'delete-dataset':
        this.$nuxt.$router.replace({ path: '/main/delete'})
        break
      case 'delete-labeled-dataset':
        this.$nuxt.$router.replace({ path: '/main/delete-labeled'})
        break
      case 'upload-dataset':
        this.$nuxt.$router.replace({ path: '/main/upload'})
        break
      case 'change-username':
        this.$nuxt.$router.replace({ path: '/main/change-username'})
        break
      case 'change-password':
        this.$nuxt.$router.replace({ path: '/main/change-password'})
        break
      case 'show-all-users':
        this.$nuxt.$router.replace({ path: '/main/show-user'})
        break
      case 'add-user':
        this.$nuxt.$router.replace({ path: '/main/add-user'})
        break
      case 'edit-user':
        this.$nuxt.$router.replace({ path: '/main/edit-user'})
        break
      case 'delete-user':
        this.$nuxt.$router.replace({ path: '/main/delete-user'})
        break
      }
    },
    onPathChangeHandler (browserURL) {
      var elmtID = ''
      
      if (((/^\/main\/output-view(\/|(\?)|$)/.test(browserURL)))) {
        var type = this.$route.query.type
        if (type === 'xml') {
          elmtID = 'pascal-output'
        } else {
          elmtID = 'coco-output'
        }
        this.activeElmtID = elmtID
        this.setClass(elmtID, 'active')
      } else {
        if ((/^\/main\/label(\/|(\?)|$)/.test(browserURL))) {
          elmtID ='label-dataset'
        } else if((/^\/main\/pascal(\/|(\?)|$)/.test(browserURL))){
          elmtID = 'pascal-output'
        } else if ((/^\/main\/coco(\/|(\?)|$)/.test(browserURL))) {
          elmtID ='coco-output'
        } else if ((/^\/main\/edit(\/|(\?)|$)/.test(browserURL))) {
          elmtID ='edit-dataset'
        } else if ((/^\/main\/all-images(\/|(\?)|$)/.test(browserURL))) {
          elmtID = 'all-images'
        } else if ((/^\/main\/delete-labeled(\/|(\?)|$)/.test(browserURL))) {
          elmtID ='delete-labeled-dataset'
        } else if ((/^\/main\/delete(\/|(\?)|$)/.test(browserURL))) {
          elmtID ='delete-dataset'
        } else if ((/^\/main\/upload(\/|(\?)|$)/.test(browserURL))) {
          elmtID ='upload-dataset'
        } else if ((/^\/main\/change-username(\/|(\?)|$)/.test(browserURL))) {
          elmtID ='change-username'
        } else if ((/^\/main\/change-password(\/|(\?)|$)/.test(browserURL))) {
          elmtID ='change-password'
        } else if ((/^\/main\/show-user(\/|(\?)|$)/.test(browserURL))) {
          elmtID ='show-all-users'
        } else if ((/^\/main\/add-user(\/|(\?)|$)/.test(browserURL))) {
          elmtID ='add-user'
        } else if ((/^\/main\/edit-user(\/|(\?)|$)/.test(browserURL))) {
          elmtID ='edit-user'
        } else if (((/^\/main\/delete-user(\/|(\?)|$)/.test(browserURL)))) {
          elmtID ='delete-user'
        }
        this.changeActiveElmtID(elmtID)
        this.setClass(elmtID, 'active')
      }
    },
    logout () {
      cookies.remove('Authorization')
      this.$nuxt.$router.replace({ path: '/'})
    }
  }
}
</script>

<style scoped>
  .side-bar-layout {
    /* height: 100%; */
    height: auto;
    left: 0;

    /* min-height: 100vh; */
    width: 250px;

    padding-right: 30px;

    overflow-y: scroll;
    position: fixed;
    top: 0;
    bottom: 0;
  }

  .side-bar-layout::-webkit-scrollbar {
    display: none;
  }

  .side-bar-layout {
      -ms-overflow-style: none;
  }

  .side-bar-bg {
    background-color: #1E889B;
  }

  .side-bar-font {
    font-family: 'Open Sans Regular';
    font-size: 0.95rem;

    letter-spacing: 0.035rem;
  }

  #user-name {
    /* font-size: 1.15rem; */
    font-size: 1.6rem;
    font-weight: 700;

    font-family: 'Open Sans Bold';
  }

  .icon-text-wrapper {
    display: flex;
  }

  .active {
    font-family: 'Open Sans Bold';
  }

  .side-bar-color {
    color: white;
  }

  .side-bar-color:hover {
    background-color: #166472;
  }

  .output-icon {
    margin-top: -1px; font-family: 'Open Sans Bold';
  }

</style>