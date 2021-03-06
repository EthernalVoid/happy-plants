<template>
  <div>
    <storage-dialog
      id="storage-dialog"
      app-root=".settings-data"
      :show="showDialog"
      @close-dialog="closeDialog">
      <span slot="headline">Login to Google</span>
      <div>
        <p>
          When using the cloud storage option, you have to sign in.
        </p>

        <v-button @click.native="loginUser">
          Open Google sign-in
        </v-button>
      </div>
    </storage-dialog>

    <div class="intro-storage box">
      <h2>How do you want to save your plant data?</h2>
      <p>
        You can change this at any time in the settings later on.
      </p>
      <ul>
        <li v-for="(type, index) of options" :key="index">
          <input
            type="radio"
            name="storage-type"
            :checked="selectedStorageType === type.id"
            :id="type.id"
            @change="updateStorageMethod(type.id)">
          <label :for="type.id">
            <span class="storage-title">
              <component
                :is="`feather-${type.icon}`"
                height="22"
                width="22" />
              {{ type.label }}
            </span>
            <span class="storage-description">
              {{ type.description }}
            </span>
          </label>
        </li>
      </ul>
    </div>

    <div class="intro-footer">
      <router-link to="/intro" class="btn plain">
        <div class="button-icon">
          <feather-left />
        </div>
        <span>Back</span>
      </router-link>
      <router-link to="/intro/howto" class="btn">
        <div class="button-icon">
          <feather-right />
        </div>
        <span>Next</span>
      </router-link>
    </div>
  </div>
</template>

<script>
  import { mapActions, mapState } from 'vuex'
  import HappyDialog from '@/components/HappyDialog'
  import Button from '@/components/Button'

  export default {
    name: 'IntroStorage',

    components: {
      'storage-dialog': HappyDialog,
      'v-button': Button,
      'feather-right': () =>
        import('vue-feather-icons/icons/ArrowRightIcon' /* webpackChunkName: "icons" */),
      'feather-left': () =>
        import('vue-feather-icons/icons/ArrowLeftIcon' /* webpackChunkName: "icons" */),
      'feather-cloud': () =>
        import('vue-feather-icons/icons/CloudIcon' /* webpackChunkName: "icons" */),
      'feather-smartphone': () =>
        import('vue-feather-icons/icons/SmartphoneIcon' /* webpackChunkName: "icons" */)
    },

    computed: mapState({
      authenticated: state => state.user.authenticated,
      selectedStorageType: state => state.storage.type
    }),

    data: () => ({
      showDialog: false,
      options: [
        {
          label: 'Locally on phone',
          icon: 'smartphone',
          description: 'This is the default option. All application data is stored on your device.',
          id: 'local'
        },
        {
          label: 'In the cloud',
          icon: 'cloud',
          description: `Stores your plant data in Google Firebase. Let's you login from different devices. Required by some features.`,
          id: 'cloud'
        }
      ]
    }),

    methods: {
      ...mapActions([
        'updateStorage',
        'signInUser'
      ]),
      updateStorageMethod (type) {
        if (type === 'cloud') {
          this.showDialog = true
        } else {
          this.updateStorage({ type })
        }
      },
      closeDialog () {
        this.showDialog = false
      },
      async loginUser () {
        await this.updateStorage({ type: 'cloud' })
        await this.signInUser('google')
        if (!this.authenticated) return
        this.closeDialog()
        this.$router.push('/intro/howto')
      }
    }
  }
</script>

<style lang="postcss" scoped>
  .intro-storage {
    padding: var(--base-gap);

    & p {
      margin-top: var(--base-gap);
    }

    & ul {
      margin-top: calc(2 * var(--base-gap));
      list-style: none;
    }

    & li {
      display: flex;
      align-items: flex-start;

      &:not(:last-child) {
        margin-bottom: var(--base-gap);
      }
    }

    & input {
      margin-right: var(--base-gap);
      flex: 0 0 auto;
    }

    & .storage-title {
      display: flex;
      align-items: center;

      & svg {
        margin-right: calc(var(--base-gap) / 2);
      }
    }

    & .storage-description {
      display: block;
      margin-top: calc(var(--base-gap) / 2);
      font-size: var(--text-size-small);
      color: var(--text-color-secondary);
    }
  }

  .intro-footer {
    margin-top: calc(2 * var(--base-gap));
    display: flex;
    justify-content: space-between;

    & .btn .button-icon svg {
      filter: none;
    }
  }
</style>
