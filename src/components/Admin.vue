<template>
  <div>
    <v-toolbar>
      <v-toolbar-title>Settings Administration</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-toolbar-items class="hidden-sm-and-down">
        <v-btn flat @click="goToHome">Home</v-btn>
        <v-btn flat @click="goToPublisherConsole">Publisher Console</v-btn>
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
    <v-alert v-model="alert" dismissible type="error">{{alertMessage}}</v-alert>
    <div class="text-xs-center">
      <v-btn v-if="!alert" color="primary" dark @click="alert=true">Clear Alerts</v-btn>
    </div>
    <v-content>
      <v-container grid-list-md>
        <v-layout row wrap>
          <v-flex xs6>
            <v-card v-if="this.$store.state.isAdminUserLoggedIn">
              <v-card-title primary-title>
                <div>
                  <h3 class="headline mb-0">Application Name</h3>
                </div>
              </v-card-title>
              <v-card-actions>
                <v-container style="height:240px;max-height:300px;overflow-y:scroll">
                  <ul>
                    <v-text-field label="Application Name" v-model="applicationName"></v-text-field>
                    <v-btn
                      color="primary"
                      @click="() => { this.saveApplicationName(this.applicationName) }"
                    >Save</v-btn>
                  </ul>
                </v-container>
              </v-card-actions>
            </v-card>
          </v-flex>
          <v-flex xs6>
            <v-card v-if="this.$store.state.isAdminUserLoggedIn">
              <v-card-title primary-title>
                <div>
                  <h3 class="headline mb-0">Company Logo</h3>
                </div>
              </v-card-title>
              <v-card-actions>
                <v-container style="height:240px;max-height:300px;overflow-y:scroll">
                  <ul>
                    <input type="file" id="file" v-on:change="showFile()" accept="image/*">
                    <v-img :src="this.companyLogo" width="150" alt="Thumb preview..."/>
                    <!--img src="{{this.companyLogo}}" width="150" alt="Thumb preview..."-->
                  </ul>
                </v-container>
              </v-card-actions>
            </v-card>
          </v-flex>
        </v-layout>
        <v-layout row wrap>
          <v-flex xs4>
            <v-container style="max-height:300px;overflow-y:scroll">
              <v-card v-if="this.$store.state.isAdminUserLoggedIn">
                <v-card-title primary-title>
                  <div>
                    <h3 class="headline mb-0">Default Hub & Project</h3>
                    <div>{{ defaultHubProject[0].hub }} | {{ defaultHubProject[0].project }}</div>
                  </div>
                </v-card-title>
                <v-card-actions>
                  <v-btn v-if="!alert" color="primary" dark @click="isDefaultHubProjectDefined=false">Reset</v-btn>
                </v-card-actions>
              </v-card>
            </v-container>
            <v-container  style="height:240px;max-height:300px;overflow-y:scroll">
              <v-stepper
                v-model="step"
                vertical
                v-if="this.$store.state.isAdminUserLoggedIn && !this.isDefaultHubProjectDefined"
              >
                <v-stepper-step :complete="step > 1" step="1">
                  Select default hub
                  <small>{{ selectedHub }}</small>
                </v-stepper-step>
                <v-stepper-content step="1">
                  <v-card>
                    <v-radio-group v-model="selectedHub" v-for="hub in hubs" v-bind:key="hub.id">
                      <v-radio :label="hub.name" :value="hub.id" :checked="hub.id"></v-radio>
                    </v-radio-group>
                  </v-card>
                  <v-btn
                    color="primary"
                    @click="() => { step=2; this.getProjects(selectedHub) }"
                  >Continue</v-btn>
                  <v-btn flat>Cancel</v-btn>
                </v-stepper-content>
                <v-stepper-step :complete="step > 2" step="2">
                  Select default project
                  <small>{{ selectedProject }}</small>
                </v-stepper-step>
                <v-stepper-content step="2">
                  <v-card>
                    <v-radio-group
                      v-model="selectedProject"
                      v-for="project in projects"
                      v-bind:key="project.id"
                    >
                      <v-radio :label="project.name" :value="project.id" :checked="project.id"></v-radio>
                    </v-radio-group>
                    <v-card-actions>
                      <v-btn
                        color="primary"
                        @click="() => { step=3; this.saveDefaultHubProject(selectedHub, selectedProject) }"
                      >Save</v-btn>
                      <v-btn flat>Cancel</v-btn>
                    </v-card-actions>
                  </v-card>
                </v-stepper-content>
              </v-stepper>
            </v-container>
          </v-flex>
          <v-flex xs4>
            <v-card v-if="this.$store.state.isAdminUserLoggedIn">
              <v-card-title primary-title>
                <div>
                  <h3 class="headline mb-0">Global Settings</h3>
                </div>
              </v-card-title>
              <v-container style="height:240px;max-height:300px;overflow-y:scroll">
                <v-list subheader two-line>
                  <v-subheader>Feature Toggles</v-subheader>
                  <v-list-tile>
                    <v-list-tile-action>
                      <v-checkbox v-model="animation"></v-checkbox>
                    </v-list-tile-action>
                    <v-list-tile-content @click="animation = !animation">
                      <v-list-tile-title>Fusion Animation</v-list-tile-title>
                      <v-list-tile-sub-title>Enable support for Fusion360 authored animations</v-list-tile-sub-title>
                    </v-list-tile-content>
                  </v-list-tile>
                  <v-list-tile>
                      <v-list-tile-action>
                      <v-checkbox v-model="arvr"></v-checkbox>
                    </v-list-tile-action>
                    <v-list-tile-content @click="arvr = !arvr">
                      <v-list-tile-title>AR/VR Toolkit (Beta)</v-list-tile-title>
                      <v-list-tile-sub-title>Enables AR/VR Toolkit</v-list-tile-sub-title>
                    </v-list-tile-content>
                  </v-list-tile>
                  <v-list-tile>
                      <v-list-tile-action>
                      <v-checkbox v-model="twin" disabled></v-checkbox>
                    </v-list-tile-action>
                    <v-list-tile-content @click="twin = !twin">
                      <v-list-tile-title>Virtual Operations (Beta)</v-list-tile-title>
                      <v-list-tile-sub-title>Enables digital twin features</v-list-tile-sub-title>
                    </v-list-tile-content>
                  </v-list-tile>
                </v-list>
              </v-container>
              <v-card-actions>
                <v-btn
                  color="primary"
                  @click="() => { this.saveFeatureToggles(animation, arvr, twin) }"
                >Save</v-btn>
              </v-card-actions>
            </v-card>
          </v-flex>
          <v-flex xs4>
            <v-card v-if="this.$store.state.isAdminUserLoggedIn">
              <v-card-title primary-title>
                <div>
                  <h3 class="headline mb-0">Supported File Formats</h3>
                </div>
              </v-card-title>
              <v-container style="height:240px;max-height:300px;overflow-y:scroll">
                <v-list-tile>
                  <v-list-tile-action>
                    <v-checkbox v-model="formats.creo"></v-checkbox>
                  </v-list-tile-action>
                  <v-list-tile-content @click="formats.creo = !formats.creo">
                    <v-list-tile-title>PTC Creo</v-list-tile-title>
                    <v-list-tile-sub-title>.ASM, .DRW, .PRT</v-list-tile-sub-title>
                  </v-list-tile-content>
                </v-list-tile>
                <v-list-tile>
                  <v-list-tile-action>
                    <v-checkbox v-model="formats.fusion"></v-checkbox>
                  </v-list-tile-action>
                  <v-list-tile-content @click="formats.fusion = !formats.fusion">
                    <v-list-tile-title>Fusion</v-list-tile-title>
                    <v-list-tile-sub-title>.F2D, .F3D</v-list-tile-sub-title>
                  </v-list-tile-content>
                </v-list-tile>
                <v-list-tile>
                  <v-list-tile-action>
                    <v-checkbox v-model="formats.inventor"></v-checkbox>
                  </v-list-tile-action>
                  <v-list-tile-content @click="formats.inventor = !formats.inventor">
                    <v-list-tile-title>Inventor</v-list-tile-title>
                    <v-list-tile-sub-title>.IAM, .IDW, .IPT</v-list-tile-sub-title>
                  </v-list-tile-content>
                </v-list-tile>
                <v-list-tile>
                  <v-list-tile-action>
                    <v-checkbox v-model="formats.navisworks"></v-checkbox>
                  </v-list-tile-action>
                  <v-list-tile-content @click="formats.navisworks = !formats.navisworks">
                    <v-list-tile-title>NavisWorks</v-list-tile-title>
                    <v-list-tile-sub-title>.NWD</v-list-tile-sub-title>
                  </v-list-tile-content>
                </v-list-tile>
                <v-list-tile>
                  <v-list-tile-action>
                    <v-checkbox v-model="formats.obj"></v-checkbox>
                  </v-list-tile-action>
                  <v-list-tile-content @click="formats.obj = !formats.obj">
                    <v-list-tile-title>WaveFront</v-list-tile-title>
                    <v-list-tile-sub-title>.OBJ</v-list-tile-sub-title>
                  </v-list-tile-content>
                </v-list-tile>
                <v-list-tile>
                  <v-list-tile-action>
                    <v-checkbox v-model="formats.solidworks"></v-checkbox>
                  </v-list-tile-action>
                  <v-list-tile-content @click="formats.solidworks = !formats.solidworks">
                    <v-list-tile-title>SolidWorks</v-list-tile-title>
                    <v-list-tile-sub-title>.SLDASM .SLDDRW .SLDPRT</v-list-tile-sub-title>
                  </v-list-tile-content>
                </v-list-tile>
                <v-list-tile>
                  <v-list-tile-action>
                    <v-checkbox v-model="formats.step"></v-checkbox>
                  </v-list-tile-action>
                  <v-list-tile-content @click="formats.step = !formats.step">
                    <v-list-tile-title>STEP</v-list-tile-title>
                    <v-list-tile-sub-title>.STP .STEP</v-list-tile-sub-title>
                  </v-list-tile-content>
                </v-list-tile>
              </v-container>
              <v-card-actions>
                <v-btn
                  color="primary"
                  @click="() => { 
                    this.saveFileFormats(
                      formats.creo,
                      formats.fusion, 
                      formats.inventor, 
                      formats.navisworks,
                      formats.obj,
                      formats.solidworks, 
                      formats.step
                    )}"
                >Save</v-btn>
              </v-card-actions>
            </v-card>
          </v-flex>
        </v-layout>
        <v-layout row wrap>
          <v-flex xs12>
            <v-card v-if="this.$store.state.isAdminUserLoggedIn">
              <v-card-title primary-title>
                <div>
                  <h3 class="headline mb-0">WebHooks</h3>
                </div>
              </v-card-title>
              <v-data-table
                :headers="headers"
                :items="webhooks"
                item-key="hookId"
                class="elevation-1"
              >
                <template v-slot:items="props">
                  <td>{{ props.item.id }}</td>
                  <td>{{ props.item.tenant }}</td>
                  <td>{{ props.item.event }}</td>
                  <td>{{ props.item.system }}</td>
                  <td>{{ props.item.status }}</td>
                  <td class="justify-center layout px-0">
                    <v-icon small @click="deleteWebHook(props.item.id)">delete</v-icon>
                  </td>
                </template>
              </v-data-table>
              <v-card-actions>
                <v-btn color="primary" @click="setWebHook">New WebHook</v-btn>
              </v-card-actions>
            </v-card>
          </v-flex>
        </v-layout>
      </v-container>
    </v-content>
  </div>
</template>

<script>
import config from './../config'

export default {
  beforeMount() {
    const retrievedSession = this.validateSession(localStorage.getItem('loggedInSession'))
    // detect if query param isAdminUserLoggedIn is true
    if (this.$route.query.isAdminUserLoggedIn || retrievedSession) {
      this.$store.state.isAdminUserLoggedIn = true
      this.getHubs()
      this.setUserData()
      this.getWebHooks()
      this.getSysAdmins()
      this.getFeatureToggles()
      this.getFileFormatToggles()
      this.getCompanyLogo();
      this.getApplicationName();
    }
  },
  data: () => ({
    alert: false,
    alertMessage: '',
    animation: false,
    applicationName: '',
    arvr: false,
    companyLogo: '',
    defaultHubProject: 'Undefined',
    formats: {
      creo: false,
      fusion: false,
      inventor: false,
      navisworks: false,
      obj: false,
      solidworks: false,
      step: false
    },
    headers: [{
      text: 'Hook Id',
      align: 'left',
      sortable: false,
      value: 'id'
    }, {
      text: 'Tenant',
      value: 'tenant'
    }, {
      text: 'Event',
      value: 'event'
    }, {
      text: 'System',
      value: 'system'
    }, {
      text: 'Status',
      value: 'status'
    }],
    hubs: [],
    isDefaultHubProjectDefined: false,
    isWebAdminsDefined: false,
    projects: [],
    selectedHub: 'No selection',
    selectedProject: 'No selection',
    step: 1,
    twin: false,
    webhooks: [],
    webAdmins: 'Undefined'
  }),
  methods: {
    async deleteWebHook(hookId) {
      try {
        this.$store.dispatch('setDeleting', { webHook: true })
        const res = await this.$axios({
          method: 'DELETE',
          url: new URL(`/api/admin/webhooks/${hookId}`, config.koahost).href
        })
        if (res.status === 200) {
          this.$log.info(`... deleted WebHookId: ${hookId}`)
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        this.$store.dispatch('setDeleting', { webHook: false })
        this.getWebHooks()
      }
    },
    async getApplicationName() {
      try {
        this.$store.dispatch('setLoading', { applicationName: true })
        const res = await this.$axios({
          method: 'GET',
          url: new URL(`/api/admin/ApplicationName`, config.koahost).href
        })
        if (res.status === 200 && res.data.length === 1) {
          this.isWebAdminsDefined = true
          const applicationNameSetting = res.data
          this.applicationName = applicationNameSetting[0].appName
          this.$store.dispatch('setApplicationName', this.applicationName)
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        this.$store.dispatch('setLoading', { applicationName: false })
      }
    },
    async getCompanyLogo() {
      try {
        this.$store.dispatch('setLoading', { companyLogo: true })
        const res = await this.$axios({
          method: 'GET',
          url: new URL(`/api/admin/CompanyLogo`, config.koahost).href
        })
        if (res.status === 200 && res.data.length === 1) {
          this.isWebAdminsDefined = true
          const companyLogoSetting = res.data
          this.companyLogo = companyLogoSetting[0].imageSrc
          this.$store.dispatch('setCompanyLogo', this.companyLogo)
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        this.$store.dispatch('setLoading', { companyLogo: false })
      }
    },
    async getDefaultHubProject() {
      try {
        this.$store.dispatch('setLoading', { defaultHubProjectSetting: true })
        const res = await this.$axios({
          method: 'GET',
          url: new URL(`/api/admin/settings/defaultHubProject/email/${this.$store.state.user.email}`, config.koahost).href
        })
        if (res.status === 200 && res.data.length === 1) {
          this.isDefaultHubProjectDefined = true
          const defaultHubProjectSetting = res.data
          this.defaultHubProject = defaultHubProjectSetting.map((val, i) => {
            return {
              hub: val.hubName,
              project: val.projectName
            }
          })
          this.$store.dispatch('setDefaultHubProjectSetting', res.data[0])
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        this.$store.dispatch('setLoading', { defaultHubProjectSetting: false })
      }
    },
    async getFeatureToggles() {
      try {
        const res = await this.$axios({
          method: 'GET',
          url: new URL('/api/admin/settings/features', config.koahost).href
        })
        if (res.status === 200 && res.data.length > 0) {
          this.$log.info('... retrieved feature toggles in database.')
          this.animation = res.data[0].featureToggles.fusion_animation
          this.arvr = res.data[0].featureToggles.arvr_toolkit
          this.twin = res.data[0].featureToggles.digital_twin
          this.$store.dispatch('setFeatureToggles', {
            animation: res.data[0].featureToggles.fusion_animation,
            arvr: res.data[0].featureToggles.arvr_toolkit,
            twin: res.data[0].featureToggles.digital_twin
          })
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      }
    },
    async getFileFormatToggles() {
      try {
        const res = await this.$axios({
          method: 'GET',
          url: new URL('/api/admin/settings/fileformats', config.koahost).href
        })
        if (res.status === 200 && res.data.length > 0) {
          this.$log.info('... retrieved file format toggles in database.')
          this.formats.creo = res.data[0].fileFormatToggles.creo
          this.formats.fusion = res.data[0].fileFormatToggles.fusion
          this.formats.inventor = res.data[0].fileFormatToggles.inventor
          this.formats.navisworks = res.data[0].fileFormatToggles.navisworks
          this.formats.obj = res.data[0].fileFormatToggles.obj
          this.formats.solidworks = res.data[0].fileFormatToggles.solidworks
          this.formats.step = res.data[0].fileFormatToggles.step
          this.$store.dispatch('setFileFormatToggles', {
            creo: res.data[0].fileFormatToggles.creo,
            fusion: res.data[0].fileFormatToggles.fusion,
            inventor: res.data[0].fileFormatToggles.inventor,
            navisworks: res.data[0].fileFormatToggles.navisworks,
            obj: res.data[0].fileFormatToggles.obj,
            solidworks: res.data[0].fileFormatToggles.solidworks,
            step: res.data[0].fileFormatToggles.step
          })
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      }
    },
    async getHubs() {
      try {
        this.$store.dispatch('setLoading', { hubsInfo: true })
        const res = await this.$axios({
          method: 'GET',
          url: new URL('/api/fusion/hubs', config.koahost).href
        })
        if (res.status === 200) {
          const hubsInfo = res.data.data
          this.hubs = hubsInfo.map((val, i) => {
            return {
              id: val.id,
              name: val.attributes.name
            }
          })
          this.$store.dispatch('setHubs', this.hubs)
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        this.$store.dispatch('setLoading', { hubsInfo: false })
      }
    },
    async getProjects(hubId) {
      try {
        this.$store.dispatch('setLoading', { projectsInfo: true })
        const res = await this.$axios({
          method: 'GET',
          url: new URL(`/api/fusion/hubs/${hubId}/projects`, config.koahost).href
        })
        if (res.status === 200) {
          const projectsInfo = res.data.data
          this.projects = projectsInfo.map((val, i) => {
            return {
              id: val.id,
              name: val.attributes.name
            }
          })
          this.$store.dispatch('setProjects', this.projects)
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        this.$store.dispatch('setLoading', { projectsInfo: false })
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
    goToHome() {
      this.$router.push({ path: '/' })
    },
    goToHelp() {
      window.open('https://mazerab.github.io/forge-digital-catalog/', '_blank')
    },
    goToPublisherConsole() {
      this.$router.push({ path: '/publish' })
    },
    login() {
      window.location.href = new URL('/api/forge/authenticate?state=admin', config.koahost).href
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
          window.open(new URL('/api/forge/logoutaccount', config.koahost).href, '_blank')
          window.location.href = '/'
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      }
    },
    readFileAsync(file) {
      return new Promise((resolve, reject) => {
        let reader = new FileReader();
        reader.onload = () => {
          resolve(reader.result);
        };
        reader.onerror = reject;
        reader.readAsDataURL(file);
      })
    },
    async saveCompanyLogo(imageSrc) {
      try {
        this.$store.dispatch('setSaving', { companyLogo: true })
        const res = await this.$axios({
          data: {
            name: 'companyLogo',
            imageSrc: imageSrc
          },
          headers: {
            'Content-Type': 'application/json'
          },
          method: 'POST',
          url: new URL('/api/admin/CompanyLogo', config.koahost).href,

        })
        if (res.status === 200) {
          this.$store.dispatch('setCompanyLogo', res.data)
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        this.$store.dispatch('setSaving', { companyLogo: false })
        this.getCompanyLogo()
      }
    },
    async saveApplicationName(providedName) {
      try {
        this.$store.dispatch('setSaving', { applicationName: true })
        const res = await this.$axios({
          data: {
            name: 'applicationName',
            value: providedName
          },
          headers: {
            'Content-Type': 'application/json'
          },
          method: 'POST',
          url: new URL('/api/admin/ApplicationName', config.koahost).href
        })
        if (res.status === 200) {
          this.$store.dispatch('setApplicationName', res.data)
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        this.$store.dispatch('setSaving', { applicationName: false })
        this.getApplicationName()
      }
    },
    async saveDefaultHubProject(hub, project) {
      try {
        this.$store.dispatch('setSaving', { defaultHubProjectSetting: true })
        const res = await this.$axios({
          data: {
            name: 'defaultHubProject',
            hubId: hub,
            hubName: this.hubs.filter(hubObj => {
              return hubObj.id === this.selectedHub
            })[0].name,
            projectId: project,
            projectName: this.projects.filter(projectObj => {
              return projectObj.id === this.selectedProject
            })[0].name,
            email: this.$store.state.user.email
          },
          headers: {
            'Content-Type': 'application/json'
          },
          method: 'POST',
          url: new URL('/api/admin/default/hub/project', config.koahost).href,

        })
        if (res.status === 200) {
          this.$store.dispatch('setDefaultHubProjectSetting', res.data)
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        this.$store.dispatch('setSaving', { defaultHubProjectSetting: false })
        this.getDefaultHubProject()
      }
    },
    async saveFeatureToggles(animation, arvr, twin) {
      try {
        this.$store.dispatch('setSaving', { featureToggleSetting: true })
        const res = await this.$axios({
          data: {
            animation,
            arvr,
            twin
          },
          method: 'POST',
          url: new URL('/api/admin/settings/features', config.koahost).href
        })
        if (res.status === 200) {
          this.$log.info('... saved feature toggles in database.')
          this.animation = res.data.featureToggles.fusion_animation
          this.arvr = res.data.featureToggles.arvr_toolkit
          this.twin = res.data.featureToggles.digital_twin
          this.$store.dispatch('setFeatureToggles', {
            animation: res.data.featureToggles.fusion_animation,
            arvr: res.data.featureToggles.arvr_toolkit,
            twin: res.data.featureToggles.digital_twin
          })
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        this.$store.dispatch('setSaving', { featureToggleSetting: false })
      }
    },
    async saveFileFormats(creo, fusion, inventor, navisworks, obj, solidworks, step) {
      try {
        this.$store.dispatch('setSaving', { fileFormatSetting: true })
        const res = await this.$axios({
          data: {
            creo,
            fusion,
            inventor,
            navisworks,
            obj,
            solidworks,
            step
          },
          method: 'POST',
          url: new URL('/api/admin/settings/fileformats', config.koahost).href
        })
        if (res.status === 200) {
          this.$log.info('... saved file formats toggles in database.')
          this.formats.creo = res.data.fileFormatToggles.creo
          this.formats.fusion = res.data.fileFormatToggles.fusion
          this.formats.inventor = res.data.fileFormatToggles.inventor
          this.formats.navisworks = res.data.fileFormatToggles.navisworks
          this.formats.obj = res.data.fileFormatToggles.obj
          this.formats.solidworks = res.data.fileFormatToggles.solidworks
          this.formats.step = res.data.fileFormatToggles.step
          this.$store.dispatch('setFileFormatToggles', {
            creo: res.data.fileFormatToggles.creo,
            fusion: res.data.fileFormatToggles.fusion,
            inventor: res.data.fileFormatToggles.inventor,
            navisworks: res.data.fileFormatToggles.navisworks,
            obj: res.data.fileFormatToggles.obj,
            solidworks: res.data.fileFormatToggles.solidworks,
            step: res.data.fileFormatToggles.step
          })
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        this.$store.dispatch('setSaving', { fileFormatSetting: false })
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
          this.$store.dispatch('setUserFullName', `${res.data.firstName} ${res.data.lastName}`)
          this.$store.dispatch('setUserPicture', res.data.profileImages.sizeX40)
          this.$store.dispatch('setUser', {
            email: this.$store.state.user.email,
            fullName: this.$store.state.user.fullName,
            picture: this.$store.state.user.picture
          })
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        localStorage.setItem('loggedInSession', JSON.stringify(this.$store.state.user))
        this.getDefaultHubProject()
        this.$store.dispatch('setLoading', { userInfo: false })
      }
    },
    async setWebHook() {
      try {
        this.$store.dispatch('setSaving', { newWebHook: true })
        const res = await this.$axios({
          method: 'POST',
          url: new URL(`/api/admin/webhooks`, config.koahost).href
        })
        if (res.status === 200) {
          this.$log.info('Saved new webhook')
        }
      } catch (err) {
        this.alert = true
        this.alertMessage = err
      } finally {
        this.$store.dispatch('setSaving', { newWebHook: false })
        this.getWebHooks()
      }
    },
    async showFile() {
      try {
        const imageFile = document.querySelector('input[type=file]').files[0];
        const contentBuffer = await this.readFileAsync(imageFile)
        await this.saveCompanyLogo(contentBuffer)
      } catch (err) {
        this.alert = true
        this.alertMessage = err
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
    }
  }
}
</script>
