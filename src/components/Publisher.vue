<template>
  <div>
    <v-toolbar>
      <v-toolbar-title>Publisher Console</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-toolbar-items class="hidden-sm-and-down">
        <v-btn flat @click="goToHome">Home</v-btn>
        <v-btn flat @click="goToAdminConsole">Administration</v-btn>
        <v-btn flat @click="goToHelp">Help</v-btn>
        <v-btn
          flat
          :icon="$vuetify.breakpoint.xs"
          @click="login"
          v-if="!this.$store.state.isAdminUserLoggedIn"
        >
          <v-icon>person</v-icon>
          <span class="hidden-md-and-down">Login</span>
        </v-btn>
        <v-btn
          flat
          :icon="$vuetify.breakpoint.xs"
          @click="logout"
          v-if="this.$store.state.isAdminUserLoggedIn"
        >
          <v-icon>power_settings_new</v-icon>
          <span class="hidden-md-and-down">Logout</span>
        </v-btn>
        <span v-if="this.$store.state.isAdminUserLoggedIn">
          <v-progress-circular
            indeterminate
            color="primary"
            v-if="this.$store.state.loading.userInfo"
          ></v-progress-circular>
          <v-avatar
            :title="this.$store.state.user.fullName"
            v-if="!this.$store.state.loading.userInfo"
          >
            <img :src="this.$store.state.user.picture">
          </v-avatar>
          <strong class="pl-1 hidden-sm-and-down" v-html="this.$store.state.user.fullName"></strong>
        </span>
      </v-toolbar-items>
    </v-toolbar>
    <v-alert v-model="success" dismissible type="success">{{successMessage}}</v-alert>
    <v-alert v-model="alert" dismissible type="error">{{alertMessage}}</v-alert>
    <v-content>
      <v-container grid-list-md bg fluid>
        <v-layout row wrap align-center>
            <v-flex xs5>
              <v-container style="max-height:500px;overflow-y:scroll">
                <v-card v-if="this.$store.state.isAdminUserLoggedIn">
                  <v-card-text>
                    <autodeskTree/>
                  </v-card-text>
                </v-card>
              </v-container>
            </v-flex>
            <v-flex xs2>
              <v-card v-if="this.$store.state.isAdminUserLoggedIn">
                <v-card-actions>
                  <v-flex class='text-xs-center'>
                    <v-btn v-if="this.active" color="primary" dark round @click="publish">Publish</v-btn>
                    <v-btn v-if="!this.active" color="primary" disabled round @click="publish">Publish</v-btn>
                  </v-flex>
                </v-card-actions>
              </v-card>
            </v-flex>
            <v-flex xs5>
              <v-container style="max-height:500px;overflow-y:scroll">
              <v-card v-if="this.$store.state.isAdminUserLoggedIn">
                <v-card-text>
                  <catalogTree/>
                </v-card-text>
              </v-card>
              </v-container>
            </v-flex>
        </v-layout>
        <v-layout row wrap>
          <v-flex xs12>
            <v-progress-linear v-if="!this.active" :indeterminate="true"></v-progress-linear>
          </v-flex>
        </v-layout>
        <v-layout row wrap>
          <v-flex xs12>
            <v-card v-if="this.$store.state.isAdminUserLoggedIn">
            <v-card-title primary-title>
              <div>
                <h3 class="headline mb-0">Publisher Logs</h3>
              </div>
            </v-card-title>
            <v-card-text>
              <publishLogs/>
            </v-card-text>
          </v-card>
          </v-flex>
        </v-layout>
      </v-container>
    </v-content>
    <template>
      <v-dialog v-model="userNotAdminDialog" max-width="290">
        <v-card>
          <v-card-title class="headline">Current User is not an Administrator</v-card-title>
          <v-card-text>{{alertMessage}}</v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="green darken-1" flat="flat" @click="redirectHome">Exit</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </template>
  </div>
</template>

<script>
import config from './../config'
import { encodeBase64 } from '../util/util'
import autodeskTree from './AutodeskTree.vue'
import catalogTree from './CatalogTree.vue'
import publishLogs from './PublishLogs.vue'

export default {
  beforeMount() {
    const retrievedSession = this.validateSession(localStorage.getItem('loggedInSession'))
    // detect if query param isAdminUserLoggedIn is true
    if (this.$route.query.isAdminUserLoggedIn || retrievedSession) {
      this.setUserData()
      this.$store.state.isAdminUserLoggedIn = true
    }
  },
  components: {
    autodeskTree,
    catalogTree,
    publishLogs
  },
  data: () => ({
    active: true,
    alert: false,
    alertMessage: '',
    defaultHubProject: 'Undefined',
    isDefaultHubProjectDefined: false,
    userNotAdminDialog: false,
    success: false,
    successMessage: '',
    webhooks: []
  }),
  methods: {
    encodeBase64,
    async findCatalogItemByName(name) {
      try {
        const res = await this.$axios({
          method: 'GET',
          url: new URL(
            `/api/catalog/file/name/${name}`,
            config.koahost
          ).href
        })
        if (res.status === 200) {
          return res.data
        }
      } catch {
        this.alert = true
        this.alertMessage = err
      }
    },
    async getDefaultHubProject() {
      try {
        this.$store.dispatch('setLoading', { defaultHubProjectSetting: true })
        const res = await this.$axios({
          method: 'GET',
          url: new URL(
            `/api/admin/settings/defaultHubProject/email/${
            this.$store.state.user.email
            }`,
            config.koahost
          ).href
        })
        if (res.status === 200 && res.data.length === 1) {
          this.isDefaultHubProjectDefined = true
          const defaultHubProjectSetting = res.data
          this.defaultHubProject = defaultHubProjectSetting.map(
            (val, i) => {
              return {
                hubId: val.hubId,
                hubName: val.hubName,
                projectId: val.projectId,
                projectName: val.projectName
              }
            }
          )
          this.$store.dispatch('setDefaultHubProjectSetting', res.data[0])
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        this.$store.dispatch('setLoading', { defaultHubProjectSetting: false })
      }
    },
    async getSelectedCatalogInfo() {
      try {
        const id = this.$store.state.selectedCatalog[0]
        const res = await this.$axios({
          method: 'GET',
          url: new URL(`/api/catalog/folder/id/${id}`, config.koahost).href
        })
        if (res.status === 200) {
          return res.data
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      }
    },
    async getSelectedModelInfo() {
      try {
        const projectId = this.$store.state.defaultHubProjectSetting.projectId
        let versionId = this.$store.state.selectedModel
        if (versionId.constructor === Array && versionId.length > 0) { // potential hack due to select checkbox logic
          versionId = versionId.filter(version => version.includes('fs.file'))
        }
        const res = await this.$axios({
          method: 'GET',
          url: new URL(`/api/fusion/projects/${projectId}/versions/${encodeURIComponent(versionId)}`, config.koahost).href
        })
        if (res.status === 200) {
          return res.data
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      }
    },
    async getSysAdmins() {
      try {
        this.$store.dispatch('setLoading', { webAdminsSetting: true })
        const res = await this.$axios({
          method: 'GET',
          url: new URL(`/api/admin/SysAdmins`, config.koahost).href
        })
        if (res.status === 200 && res.data.length === 1) {
          this.isWebAdminsDefined = true
          const webAdminsSetting = res.data
          this.webAdmins = webAdminsSetting[0].webAdmins
          this.$store.dispatch('setWebAdmins', this.webAdmins)
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        this.$store.dispatch('setLoading', { webAdminsSetting: false })
      }
    },
    async getWebHooks() {
      try {
        this.$store.dispatch('setLoading', { webHooksInfo: true })
        const res = await this.$axios({
          method: 'GET',
          url: new URL(`/api/admin/webhooks`, config.koahost).href
        })
        if (res.status === 200) {
          const webHooksInfo = res.data.data
          this.webhooks = webHooksInfo.map((val, i) => {
            return {
              id: val.hookId,
              tenant: val.tenant,
              event: val.event,
              system: val.system,
              status: val.status
            }
          })
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        this.$store.dispatch('setLoading', { webHooksInfo: false })
      }
    },
    goToAdminConsole() {
      this.$router.push({ path: '/admin' })
    },
    goToHelp() {
      window.open('https://mazerab.github.io/forge-digital-catalog/', '_blank')
    },
    goToHome() {
      this.$router.push({ path: '/' })
    },
    async listObjectRefs() {
      try {
        this.$store.dispatch('setLoading', { modelRefs: true })
        const selectedModelInfo = await this.getSelectedModelInfo()
        this.$store.dispatch('setRootFileName', selectedModelInfo.name)
        const fileType = selectedModelInfo.fileType
        const projectId = this.$store.state.defaultHubProjectSetting.projectId
        const versionId = this.$store.state.selectedModel
        const res = await this.$axios({
          method: 'GET',
          url: new URL(`/api/fusion/projects/${projectId}/versions/${encodeURIComponent(versionId)}/refs`, config.koahost).href
        })
        if (res.status === 200 && Array.isArray(res.data.included) && res.data.included.length > 0) {
          this.$log.info('... successfully retrieved the object references')
          const refsData = res.data.included
          const refs = await this.setReferenceTree(refsData)
          if (fileType === 'iam') {
            this.$store.dispatch('setFileType', 'Inventor')
            const invTree = await this.setInventorChildReferences(projectId, refs, selectedModelInfo.name)
            if (Array.isArray(invTree) && invTree.length > 0) {
              this.$log.info('... storing Inventor reference tree in state')
              this.$store.dispatch('setModelRefs', invTree)
            }
          }
          if (fileType.includes('autodesk.fusion360:Design')) {
            this.$store.dispatch('setFileType', 'Fusion')
            const fusionRefs = refs.filter(ref => {
              return ref.extension.includes('autodesk.fusion360')
                && ref.name !== selectedModelInfo.name 
            })
            if (Array.isArray(fusionRefs) && fusionRefs.length > 0) {
              this.$store.dispatch('setModelRefs', fusionRefs)
            }
          }
          if (fileType === 'nwd') {
            this.$store.dispatch('setFileType', 'NavisWorks')
          }
          if (fileType === 'sldasm') {
            this.$store.dispatch('setFileType', 'SolidWorks')
            const slwTree = await this.setSolidWorksChildReferences(projectId, refs, selectedModelInfo.name)
            if (Array.isArray(slwTree) && slwTree.length > 0) {
              this.$log.info('... storing SolidWorks reference tree in state')
              this.$store.dispatch('setModelRefs', slwTree)
            }
          }
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        this.$log.info(`... found ${this.$store.state.modelRefs.length} `)
        this.$store.dispatch('setLoading', { modelRefs: false })
      }
    },
    login() {
      window.location.href = new URL(
        '/api/forge/authenticate?state=publish',
        config.koahost
      ).href
    },
    async logout() {
      try {
        localStorage.removeItem('loggedInSession')
        const res = await this.$axios.get(
          new URL('/api/forge/logout', config.koahost).href
        )
        if (res.status === 200) {
          this.$store.state.isAdminUserLoggedIn = false
          this.$store.dispatch('setUser', null)
          window.open(
            new URL('/api/forge/logoutaccount', config.koahost).href,
            '_blank'
          )
          window.location.href = '/'
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      }
    },
    async moveObject() {
      try {
        const selectedModelInfo = await this.getSelectedModelInfo()
        const bucketKey = selectedModelInfo.storageLocation.split('/')[0].split(':')[3]
        const objectName = selectedModelInfo.storageLocation.split('/')[1]
        const res = await this.$axios({
          data: {
            fileType: (this.$store.state.fileType) ? this.$store.state.fileType: '',
            name: selectedModelInfo.name,
            projectId: this.$store.state.defaultHubProjectSetting.projectId,
            refs: (this.$store.state.modelRefs.length > 0) ? this.$store.state.modelRefs : [],
            versionId: this.$store.state.selectedModel[0]
          },
          method: 'POST',
          url: new URL(`/api/fusion/transfer/bucket/${bucketKey}/object/${objectName}`, config.koahost).href
        })
        if (res.status === 200 || res.status === 204) {
          this.$log.info('... successfully moved object to OSS bucket')
          const updateRes = await this.$axios({
            data: {
              srcDesignUrn: selectedModelInfo.storageLocation
            },
            method: 'PUT',
            url: new URL(`/api/catalog/file/oss/${res.data.objectId}`, config.koahost).href
          })
          if (updateRes.status === 200) {
            this.$store.dispatch('setDesignUrn', res.data.objectId)
          }
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      }
    },
    async publish() {
      try {
        await this.getWebHooks()
        if (this.webhooks.length === 0) {
          throw new Error('Publishing aborted. Please create new webHook in admin console first.')
        }
        if (
          Object.entries(this.$store.state.selectedModel).length === 0
          && this.$store.state.selectedModel.constructor === Object
          && Object.entries(this.$store.state.selectedCatalog).length === 0
          && this.$store.state.selectedCatalog.constructor === Object
        ) {
          throw new Error('Please select a catalog folder and an item version.')
        }
        if (Object.entries(this.$store.state.selectedModel).length === 0 && this.$store.state.selectedModel.constructor === Object) {
          throw new Error('Please select an item version.')
        }
        if (Object.entries(this.$store.state.selectedCatalog).length === 0 && this.$store.state.selectedCatalog.constructor === Object) {
          throw new Error('Please select a catalog folder.')
        }
        if (!this.$store.state.selectedModel[0].includes('?version=')) {
          throw new Error('Please select a version to publish.')
        }
        if (this.$store.state.selectedCatalog[0] === 1) {
          throw new Error('You cannot publish a model to the Catalog root folder!')
        }
        this.active = false
        await this.setCatalogItem()
        if (!this.alert) await this.listObjectRefs()
        if (!this.alert) await this.moveObject()
        if (!this.alert) await this.translate()
        if (!this.alert) await this.setPublishLogEntry()
        this.active = true
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      }
    },
    async setCatalogItem() {
      try {
        this.$store.dispatch('setSaving', { newCatalogItem: true })
        const now = new Date()
        this.$store.dispatch('setStartDate', now)
        const selectedCatalogInfo = await this.getSelectedCatalogInfo()
        this.$store.dispatch('setAutodeskPath', `${selectedCatalogInfo.path}${selectedCatalogInfo.name},`)
        const selectedModelInfo = await this.getSelectedModelInfo()
        const existingCatalogItem = await this.findCatalogItemByName(selectedModelInfo.name)
        if (existingCatalogItem && (selectedModelInfo.name === existingCatalogItem.name)) {
          throw new Error('Found existing catalog item with same name, aborting publishing job ...')
        }
        const res = await this.$axios({
          data: {
            isFile: true,
            isPublished: false,
            name: selectedModelInfo.name,
            path: `${selectedCatalogInfo.path}${selectedCatalogInfo.name},`,
            ossDesignUrn: '',
            size: selectedModelInfo.size,
            srcDesignUrn: selectedModelInfo.storageLocation,
            svfUrn: ''
          },
          method: 'POST',
          url: new URL('/api/catalog/file', config.koahost).href
        })
        if (res.status === 200) {
          this.$log.info('... new catalog item created')
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        this.$store.dispatch('setSaving', { newCatalogItem: false })
      }
    },
    async setInventorChildReferences(projectId, refs, selectedModelName) {
      try {
        const invRefs = refs.filter(ref => { 
          return ref.fileType.toLowerCase() === 'ipt' 
            || ( ref.fileType.toLowerCase() === 'iam' && ref.name !== selectedModelName )
        })
        const invTree = await Promise.all(invRefs.map(async (ref) => {
          if (ref.fileType.toLowerCase() === 'iam') { 
            const res = await this.$axios({
              method: 'GET',
              url: new URL(`/api/fusion/projects/${projectId}/versions/${encodeURIComponent(ref.id)}/refs`, config.koahost).href
            })
            if (res.status === 200 && Array.isArray(res.data.included) && res.data.included.length > 0) {
              const subRefsData = res.data.included
              const subRefs = await this.setReferenceTree(subRefsData, ref.name)
              ref.children = subRefs // Sets children on sub-assembly
              await this.setInventorChildReferences(projectId, subRefs, ref.name) // recurse through children
            }
          }
          return Promise.resolve(ref)
        }))
        return invTree
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      }
    },
    async setPublishLogEntry() {
      try {
        const now = new Date()
        this.$store.dispatch('setEndDate', now)
        const payload = {
          endDate: this.$store.state.endDate,
          job: {
            input: {
              designUrn: this.$store.state.designUrn,
              path: this.$store.state.autodeskPath
            },
            output: {
              svfUrn: ''
            }
          },
          name: 'File Transfer & Translation',
          startDate: this.$store.state.startDate,
          submittedBy: this.$store.state.user.fullName
        }
        const res = this.$axios({
          data: payload,
          method: 'POST',
          url: new URL('/api/admin/publish/logs', config.koahost).href
        })
        if (res.status === 200) {
          this.$log.info('.. new publish log entry created')
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        this.$store.dispatch('setDesignUrn', '')
        this.$store.dispatch('setStartDate', '')
        this.$store.dispatch('setEndDate', '')
        if (!this.alertMessage) {
          this.success = true
          this.successMessage = 'Successfully published model to the catalog, translation still in progress.'
        }
      }
    },
    async setReferenceTree(refsRawData, parentName) {
      try {
        const refs = refsRawData.reduce((result, val) => {
          if (val.relationships && val.relationships.storage && val.attributes.displayName !== parentName) {
            result.push({
              children: [],
              extension: (val.attributes.extension.type) ? val.attributes.extension.type: '',
              fileType: (val.attributes.fileType) ? val.attributes.fileType: '',
              id: (val.id) ? val.id : '',
              location: (val.relationships.storage.data.id) ? val.relationships.storage.data.id : '',
              name: (val.attributes.displayName) ? val.attributes.displayName: '',
              type: (val.type) ? val.type : '' 
            })
          }
          return result
        }, [])
        return refs
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      }
    },
    async setSolidWorksChildReferences(projectId, refs, selectedModelName) {
      try {
        const slwRefs = refs.filter(ref => {
          return (ref.fileType.toLowerCase() === 'sldprt')
            || ( (ref.fileType.toLowerCase() === 'sldasm') && ref.name !== selectedModelName )
        })
        const slwTree = await Promise.all(slwRefs.map(async (ref) => {
          if (ref.fileType.toLowerCase() === 'sldasm') { 
            const res = await this.$axios({
              method: 'GET',
              url: new URL(`/api/fusion/projects/${projectId}/versions/${encodeURIComponent(ref.id)}/refs`, config.koahost).href
            })
            if (res.status === 200 && Array.isArray(res.data.included) && res.data.included.length > 0) {
              const subRefsData = res.data.included
              const subRefs = await this.setReferenceTree(subRefsData, ref.name)
              ref.children = subRefs // Sets children on sub-assembly
              await this.setSolidWorksChildReferences(projectId, subRefs, ref.name) // recurse through children
            }
          }
          return Promise.resolve(ref)
        }))
        return slwTree
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      }
    },
    async setUserData() {
      try {
        this.$store.dispatch('setLoading', { userInfo: true })
        const res = await this.$axios({
          method: 'GET',
          url: new URL('/api/fusion/user/profile', config.koahost).href
        })
        if (res.status === 200) {
          this.$store.dispatch('setUserEmail', res.data.emailId)
          this.$store.dispatch(
            'setUserFullName',
            `${res.data.firstName} ${res.data.lastName}`
          )
          this.$store.dispatch(
            'setUserPicture',
            res.data.profileImages.sizeX40
          )
          this.$store.dispatch('setUser', {
            email: this.$store.state.user.email,
            fullName: this.$store.state.user.fullName,
            picture: this.$store.state.user.picture
          })
          await this.getSysAdmins()
          if (this.$store.state.webAdmins.indexOf(this.$store.state.user.email) !== -1) {
            this.$store.state.isAdminUserLoggedIn = true
          } else {
            //Add a modal dialog infoming that the user is not an admin
            this.alertMessage = `Please add ${this.$store.state.user.email} to the webAdmins setting for this site.`
            this.userNotAdminDialog = true
          }

        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        localStorage.setItem('loggedInSession', JSON.stringify(this.$store.state.user))
        await this.getDefaultHubProject()
        this.$store.dispatch('setLoading', { userInfo: false })
      }
    },
    async translate() {
      try {
        const urn = encodeBase64(this.$store.state.designUrn)
        const extension = this.$store.state.designUrn.split('.').pop()
        let job = {
          input: {
            urn 
          },
          output: {
            destination: { region: '' },
            formats: [{ type: 'svf', views: ['2d', '3d'] }]
          },
          misc: { workflow: '' }
        } // workflow and region will be set by the server controller
        if (extension === 'zip') {
          job.input.compressedUrn = true
          job.input.rootFilename = this.$store.state.rootFileName
        }
        const res = await this.$axios({
          data: job,
          method: 'POST',
          url: new URL('/api/admin/translate', config.koahost).href
        })
        if (res.status === 200 && res.data.result === "success") {
          this.$log.info('... translation job submitted')
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        this.$store.dispatch('setFileType', null)
        this.$store.dispatch('setModelRefs', [])
      }
    },
    validateSession(storageVariable) {
      try {
        const userObject = JSON.parse(storageVariable)
        if (userObject) {
          const retrievedEmail = String(userObject.email)
          if (retrievedEmail.indexOf('@') > -1) {
            return true
          }
        }
        return false
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      }
    },
    redirectHome() {
      this.$router.push({ path: '/' })
    }
  }
}
</script>

