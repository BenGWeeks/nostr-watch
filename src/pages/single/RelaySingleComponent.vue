<template>
  <td class="status-indicator" :key="generateKey(relay, 'aggregate')">
    <span :class="result?.aggregate" class="aggregate indicator">
        <span></span>
        <span></span>
    </span>
  </td>

  <td class="relay left-align relay-url">
    <router-link :to="`/relay/${relayClean(relay)}`" active-class="active">{{ relay }}</router-link>
  </td>

  <td class="verified">
    <span v-if="result?.identities">
      <span v-tooltip:top.tooltip="identityList()"> <span class="verified-shape-wrapper" v-if="Object.entries(result?.identities).length"><span class="shape verified"></span></span></span>
    </span>
  </td>

  <td class="location">{{ getFlag() }}</td>

  <td class="latency">
    <span>{{ result?.latency.final }}<span v-if="result?.check.latency">ms</span></span>
  </td>

  <td class="connect" :key="generateKey(relay, 'check.connect')">
    <span :class="getResultClass(relay, 'connect')"></span>
  </td>

  <td class="read" :key="generateKey(relay, 'check.read')">
    <span :class="getResultClass(relay, 'read')"></span>
  </td>

  <td class="write" :key="generateKey(relay, 'check.write')">
    <span :class="getResultClass(relay, 'write')"></span>
  </td>

  <!-- <td class="info">
    <ul v-if="result?.observations && result?.observations.length">
      <li class="observation" v-for="(alert) in result?.observations" :key="generateKey(relay, alert.description)">
        <span v-tooltip:top.tooltip="alert.description" :class="alert.type" v-if="alert.type == 'notice'">✉️</span>
        <span v-tooltip:top.tooltip="alert.description" :class="alert.type" v-if="alert.type == 'caution'">⚠️</span>
      </li>
    </ul>
  </td> -->
</template>

<script>
import { defineComponent} from 'vue'
import { InspectorResult } from 'nostr-relay-inspector'
import { countryCodeEmoji } from 'country-code-emoji';
import emoji from 'node-emoji';

export default defineComponent({
  name: 'RelaySingleComponent',
  components: {
  },
  props: {
    relay: String,
    result: {
      type: Object,
      default(){
        return structuredClone(InspectorResult)
      }
    },
    geo: {
      type: Object,
      default(){
        return {}
      }
    },
    showColumns: {
      type: Object,
      default() {
        return {
          connectionStatuses: false,
          nips: false,
          geo: false,
          additionalInfo: false
        }
      }
    },
    connection: {
      type: Object,
      default() {
        return {
          connectionStatuses: false,
          nips: false,
          geo: false,
          additionalInfo: false
        }
      }
    }
  },
  data() {
    return {
      showModal: false
    }
  },
  methods: {
     getResultClass (url, key) {
       let result = this.result?.check?.[key] === true
       ? 'success'
       : this.result?.check?.[key] === false
         ? 'failure'
         : 'pending'
       return `indicator ${result}`
     },
     getLoadingClass () {
       return this.result?.state == 'complete' ? "relay loaded" : "relay"
     },
     generateKey (url, key) {
       return `${url}_${key}`
     },

     getFlag () {
       return this.geo?.countryCode ? countryCodeEmoji(this.geo.countryCode) : emoji.get('shrug');
     },

     setCheck (bool) {
       return bool ? '✅ ' : ''
     },

     setCross (bool) {
       return !bool ? '❌' : ''
     },
     setCaution (bool) {
       return !bool ? '⚠️' : ''
     },
     identityList () {
       let string = '',
           extraString = '',
           users = Object.entries(this.result?.identities),
           count = 0

       if(this.result?.identities) {
         if(this.result?.identities.serverAdmin) {
           string = `Relay has registered an administrator pubkey: ${this.result?.identities.serverAdmin}. `
           extraString = "Additionally, "
         }

         const total = users.filter(([key]) => key!='serverAdmin').length,
               isOne = total==1

         if(total) {
           string = `${string}${extraString}Relay domain contains NIP-05 verification data for:`
           users.forEach( ([key]) => {
             if(key == "serverAdmin") return
             count++
             string = `${string} ${(count==total && !isOne) ? 'and' : ''}  @${key}${(count!=total && !isOne) ? ', ' : ''}`
           })
         }
       }
       return string
     },
     relayClean(relay) {
       return relay.replace('wss://', '')
     },
     nipSignature(key){
       return key.toString().length == 1 ? `0${key}` : key
     },
     nipFormatted(key){
       return `NIP-${this.nipSignature(key)}`
     },
     nipLink(key){
       return `https://github.com/nostr-protocol/nips/blob/master/${this.nipSignature(key)}.md`
     },
     async copy(text) {
       try {
         await navigator.clipboard.writeText(text);
       } catch(err) {
        console.error(err)
       }
     },
   }
})
</script>

<style scoped>
ul {
  margin:0;
  padding:0;
}

li {
  margin:0;
  padding:0;
  list-style:none;
}

td.nip-11,
td.verified span {
  cursor: pointer
}

::v-deep(.modal-container) {
  display: flex;
  justify-content: center;
  align-items: center;
}
::v-deep(.modal-content) {
  position: relative;
  display: flex;
  flex-direction: column;
  max-height: 90%;
  margin: 0 1rem;
  padding: 1rem;
  border: 1px solid #e2e8f0;
  border-radius: 0.25rem;
  background: #fff;
}
.modal__title {
  margin: 0 2rem 0 0;
  font-size: 1.5rem;
  font-weight: 700;
}
.modal__content {
  flex-grow: 1;
  overflow-y: auto;
}
.modal__action {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-shrink: 0;
  padding: 1rem 0 0;
}
.modal__close {
  position: absolute;
  top: 0.5rem;
  right: 0.5rem;
}


.dark-mode div ::v-deep(.modal-content) {
  border-color: #2d3748;
  background-color: #1a202c;
}

.restricted.aggregate.indicator {
  position:relative;
  left:-7px;
}

 td {
    padding-top:2px;
    padding-bottom:2px;
  }
</style>
