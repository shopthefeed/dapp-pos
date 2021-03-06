<template>
<div class="settings">
  <div class="_wrapper">
    <div class="_section">
      <div class="_title">Currency</div>
      <div class="_content">
        <q-btn-dropdown :label="get('settings/currencyLabel')" outline >
          <!-- dropdown content -->
          <q-list link>
            <q-item
              v-for="(curr, key) in get('settings/availableCurrencies')"
              @click.native="set('settings/currency', key)"
              :key="`curr-dd-${key}`"
              v-close-overlay
            >
              <q-item-main>
                <q-item-tile label>{{ curr.label }}</q-item-tile>
              </q-item-main>
            </q-item>
          </q-list>
        </q-btn-dropdown>
      </div>
    </div>
    <div class="_section">
      <div class="_title">Ethereum Wallet Address</div>
      <div class="_content _wallet">
        <q-input
          :value="get('settings/wallet.address')"
          @change="newVal => { set('settings/wallet.address', newVal) }"
        />
        <!-- todo: change with v-modal.lazy when it becomes available -->
        <!-- v-model.lazy="state.settings.wallet.address" -->
      </div>
    </div>
    <div class="_section">
      <div class="_title">Required confirmation count</div>
      <div class="_content _wallet">
        <div class="_info">
          The required minimum confirmation count you require per payment.
          This is counted by the amount of blocks on the Ethereum blockchain since the transaction.
        </div>
        <q-slider
          :value="Number(state.settings.requiredConfirmationCount)"
          @change="val => { set('settings/requiredConfirmationCount', Number(val)) }"
          :min="0" :max="10"
          :step="1" label-always snap
        />
      </div>
    </div>
    <div class="_section">
      <div class="_title full-width">Network provider</div>
      <div class="_content _provider">
        <div class="_info">
          Select a network provider.
        </div>
        <q-btn-dropdown :label="state.settings.networkProvider.selected" outline no-wrap>
          <!-- dropdown content -->
          <q-list link>
            <q-item
              v-for="(curr, key) in networksDropdown"
              @click.native="selectNetwork(key)"
              :key="`netw-sel-${key}`"
              v-close-overlay
            >
              <q-item-main>
                <q-item-tile label>{{ curr.label }}</q-item-tile>
              </q-item-main>
            </q-item>
          </q-list>
        </q-btn-dropdown>
        <div v-if="addingCustomRPC" class="_adding-custom-rpc">
          <q-field>
            <q-input
              stack-label="Name"
              v-model="customRPC.name"
            />
          </q-field>
          <q-field
            helper="We currently only support Web Socket URLs."
          >
            <q-input
              stack-label="URL"
              v-model="customRPC.url"
              placeholder="wss://..."
            />
          </q-field>
          <div class="flex">
            <button @click="addCustomRPC" class="o-btn ml-auto mr-lg">Add</button>
            <button @click="addingCustomRPC = false" class="o-txt-btn">Cancel</button>
          </div>
        </div>
      </div>
    </div>
    <div class="_section">
      <div class="_title full-width">Accepted tokens (ERC-20)</div>
      <div class="_content">
        <div class="_erc20-tokens-grid">
          <template
            v-for="token in get('settings/availableTokensOnlyErc20')"
          >
            <div class="_token-symbol" :key="token.id + '1'">{{ token.id }}</div>
            <q-input
              readonly
              :value="token.networks[state.settings.networkProvider.selected].address"
              class="_token-address"
              :key="token.id + '2'"
            />
            <button
              @click="dispatch('settings/delete/tokens.customTokens.*', token.id)"
              class="o-txt-btn _token-delete-btn"
              :key="token.id + '3'"
            >
              Delete
            </button>
          </template>
        </div>
        <div
          v-if="addingNewToken"
          class="_new-token-grid animate-pop"
        >
          <q-input
            stack-label="Token symbol"
            v-model="newToken.id"
            class="__symbol-new"
            placeholder="eth"
          />
          <q-input
            stack-label="Decimals"
            v-model="newToken.decimals"
            type="number"
            class="__decimals"
          />
          <q-input
            stack-label="ERC20 contract address"
            v-model="newToken.address"
            class="__address"
            placeholder="0x..."
          />
          <q-toggle
            label="Fiat<br>conversion" left-label
            v-model="newToken.fiatConversion"
            class="__fiat-conversion"
          />
          <q-input
            v-if="newToken.fiatConversion"
            stack-label="Coingecko ID"
            v-model="newToken.coingeckoId"
            class="__coingeckoId"
            placeholder="ethereum"
          />
          <button
            @click="addNewToken"
            class="o-btn __add-btn"
          >
            Add
          </button>
          <button
            @click="addingNewToken = false"
            class="o-txt-btn __cancel-btn"
          >
            Cancel
          </button>
        </div>
        <button
          v-if="!addingNewToken"
          @click="addingNewToken = true"
          class="o-txt-btn _token-new-erc-btn"
        >
          Add new ERC-20 token
        </button>
      </div>
    </div>
  </div>
</div>
</template>

<script>
import storeAccess from '@mixins/storeAccess'
import { openURL } from 'quasar'

// Todo: add link to page to search coingecko id
// https://api.coingecko.com/api/v3/coins/list
const newTokenFields = function () {
  return {id: '', decimals: 18, address: '', fiatConversion: true, coingeckoId: ''}
}

export default {
  components: {},
  props: [],
  mixins: [ storeAccess ],
  // ⤷ get(path)  set(path, val)  commit(path, val)  dispatch(path, val)  state
  data () {
    return {
      addingCustomRPC: false,
      addingNewToken: false,
      editingToken: false,
      customRPC: {name: '', url: ''},
      newToken: newTokenFields()
    }
  },
  computed:
  {
    networksDropdown () {
      const networks = this.get('settings/availableNetworks')
      const networksFormatted = Object.values(networks)
        .reduce((carry, n) => {
          carry[n.name] = {
            label: n.name,
            value: n.name
          }
          return carry
        }, {})
      return {...networksFormatted, add: {label: 'add custom RPC', value: 'add'}}
    },
  },
  methods:
  {
    openURL,
    selectNetwork (net) {
      if (net === 'add') {
        this.addingCustomRPC = true
        return
      }
      this.set('settings/networkProvider.selected', net)
    },
    addCustomRPC () {
      this.set(
        'settings/networkProvider.customRPCs.*',
        {[this.customRPC.name]: {name: this.customRPC.name, url: this.customRPC.url}}
      )
      this.set('settings/networkProvider.selected', this.customRPC.name)
      this.addingCustomRPC = false
      this.customRPC.name = ''
      this.customRPC.url = ''
    },
    addNewToken () {
      this.set(
        'settings/tokens.customTokens.*',
        {
          id: this.newToken.id,
          decimals: this.newToken.decimals,
          fiatConversion: this.newToken.fiatConversion,
          coingeckoId: this.newToken.coingeckoId,
          erc20: true,
          networks: {
            [this.state.settings.networkProvider.selected]: {address: this.newToken.address}
          },
        }
      )
      this.addingNewToken = false
      this.newToken = newTokenFields()
    },
  }
}
</script>

<style lang="stylus" scoped>
@import '~styl/variables'

.settings
  pa xl
._section
  mb lg
  display flex
  flex-wrap wrap
  align-items baseline
._title
  mt lg
  mr lg
  font-weight 500
._content
  flex 1
._wallet
  width 100%
  flex auto
  mt md
._adding-custom-rpc
  mt md
  >*
    mb sm

._erc20-tokens-grid
  display grid
  grid-template-columns auto 1fr auto
  align-items baseline
  grid-gap sm
  button, .q-input
    font-size .8em

._new-token-grid
  padding 1em
  border-radius 0.5em
  background #f3f4f7
  mt lg
  display grid
  align-items baseline
  justify-items stretch
  grid-template-columns 1fr 1fr
  grid-gap md
  grid-template-areas "symbol decimals" \
                      "address address" \
                      "fiat-conversion coingecko-id" \
                      "add cancel"
  button, .q-input
    font-size .8em
  .__symbol
    grid-area symbol
  .__decimals
    grid-area decimals
  .__address
    grid-area address
  .__fiat-conversion
    grid-area fiat-conversion
    font-size .8em
    justify-self center
  .__coingeckoId
    grid-area coingecko-id
  .__add-btn
    grid-area add
    justify-self center
  .__cancel-btn
    grid-area cancel
    justify-self center

._token-new-erc-btn
  mt lg
  font-size .8em

</style>
