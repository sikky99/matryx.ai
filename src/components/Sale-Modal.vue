<template>
  <section class="sale-modal">
    <b-modal id="showSaleModal"
      v-model="showSaleModal"
      hide-header
      hide-footer
      no-close-on-esc
      no-close-on-backdrop
      size="lg"
    >
    <div id="sale-modal--content">
      <div class="salemodal__header">
        <img src="../assets/images/Matryx-Logo-Grey-CMYK.png" alt="">
        <div class="close-modal text-muted" @click.prevent="closeModal">
          X
        </div>
      </div>
      <div class="salemodal__body">
        <transition name="fade">
          <div class="salemodal__body__terms" v-if="!showSaleContract">
            <br/>
            <h3 class="text-color--matryx-grey-blue">MATRYX TOKEN SALE TERMS</h3>
            <Tokensale-Sale-Terms class="salemodal__body__terms__document" id="sale-terms">
            </Tokensale-Sale-Terms>
            <p class="salemodal__body__terms__link text-color--matryx-blue" v-if="language === 'ch'">
              <a href="/sale-terms-ch.pdf" target="_blank">
                销售条款
              </a>
            </p>
            <p class="salemodal__body__terms__link text-color--matryx-blue" v-if="language === 'es'">
              <a href="/sale-terms-ch.pdf" target="_blank">
                Ver términos de venta en formato PDF.
              </a>
            </p>
            <p class="salemodal__body__terms__link text-color--matryx-blue" v-else>
              <a href="/sale-terms.pdf" target="_blank">
                View sale terms in PDF format.
              </a>
            </p>
            <div class="salemodal__body__terms__checklist">
              <Checkbox name="saleTermsRead" :obj="saleTermsRead">
                <span slot="label">
                  <span v-if=" language === 'ch' ">我已阅读并同意Matryx币销售条款</span>
                  <span v-if=" language === 'es' ">He leído y acepto los términos de venta de la Venta de Token de Matryx.</span>
                  <span v-else>I have read and agree to the Sale Terms of the Matryx Token Sale.</span>
                </span>
              </Checkbox>
              <Checkbox name="erc20WalletAddress" :obj="erc20WalletAddress">
                <span slot="label">
                  <span v-if=" language === 'ch' ">我理解并同意我必须用一个有效的ERC20兼容钱包地址</span>
                  <span v-if=" language === 'es' ">Entiendo y acepto que debo usar una dirección de billetera compatible con ERC20 válida.</span>
                  <span v-else>I understand and agree that I must use a valid ERC20 compatible wallet address.</span>
                </span>
              </Checkbox>
              <Checkbox name="noExchangeAddress" :obj="noExchangeAddress">
                <span slot="label">
                  <span v-if=" language === 'ch' ">我理解并同意我用的地址并不是一个交易平台地址</span>
                  <span v-if=" language === 'es' ">Entiendo y acepto que la dirección que uso no debe ser una dirección de intercambio.</span>
                  <span v-else>I understand and agree that the address I use must not be an exchange address.</span>
                </span>
              </Checkbox>
              <Checkbox name="havePrivateKeys" :obj="havePrivateKeys">
                <span slot="label" v-if="language === 'ch'">我理解并同意假设我用交易平台地址我必须拥有它的私匙</span>
                <span slot="label" v-if="language === 'es'">Entiendo y acepto que si uso una dirección de intercambio, debo ser dueño de las claves privadas.</span>
                <span slot="label" v-else>
                  I understand and agree that if I use an exchange address I must own the private keys.
                </span>
              </Checkbox>

              <div class="terms-email">
                <input class="terms-email--input" type="email" placeholder="Email Address" v-model="email"/>
                <p class="warn text-smaller" v-show="showEmailWarn">
                  Please enter a valid email address
                </p>
                <span class="text-smaller">* Sign up to receive updates</span>
              </div>

              <button class="submit-btn matryx-button matryx-button--blue"
                @click.prevent="submitVerified"
                :disabled="!allChecked"
              >
                SUBMIT
              </button>

              <div class="email-warn text-center" v-show="submitError">
                <p class="warn">
                 {{ submitError }}
                </p>

                 <p>
                   <small>See our <span class="link" @click="goToTroubleshooting">Troubleshooting FAQs</span> for additional details.
                   </small>
                </p>
              </div>

            </div>
          </div>
        </transition>

        <transition name="fade">
          <Sale-Modal-Contract-Info class="salemodal__body__address"
            v-if="showSaleContract"
            :dataField="contractInfo.dataField" :gas="contractInfo.gas"
            :saleAddress="contractInfo.saleAddress"
            :handleSubmit="closeModal"
          >
          </Sale-Modal-Contract-Info>
        </transition>
      </div>
    </div>
    </b-modal>
  </section>
</template>

<script>
import { appAnalytics } from '@/analytics'
import { mapState } from 'vuex'
import { isValidEmail } from '@/utils'
import Checkbox from '@/components/Checkbox'
import SaleModalContractInfo from '@/components/Sale-Modal-Contract-Info'
import TokensaleSaleTerms from '@/components/Tokensale-Sale-Terms'
import axios from 'axios'
import config from '../../config'

export default {
  name: 'SaleModal',

  components: {
    Checkbox,
    SaleModalContractInfo,
    TokensaleSaleTerms
  },

  data () {
    return {
      email: '',
      showEmailWarn: false,
      saleTermsRead: {
        checked: false,
        enable: false
      },
      erc20WalletAddress: {
        checked: false,
        enable: false
      },
      noExchangeAddress: {
        checked: false,
        enable: false
      },
      havePrivateKeys: {
        checked: false,
        enable: false
      },
      submitError: ''
    }
  },

  computed: {
    allChecked () {
      if (!this.saleTermsRead.checked || !this.erc20WalletAddress.checked ||
        !this.noExchangeAddress.checked || !this.havePrivateKeys.checked) {
        return false
      }
      return true
    },
    ...mapState({
      showSaleModal: state => state.showSaleModal,
      language: state => state.language,
      contractInfo: state => state.contractInfo,
      showSaleContract: state => state.showSaleContract
    })
  },

  methods: {
    submitVerified () {
      this.showEmailWarn = false

      if (this.email && !isValidEmail(this.email)) {
        this.showEmailWarn = true
        return
      }

      appAnalytics.submitVerify(this.email)

      if (this.allChecked) {
        this.submitError = ''
        this.$store.commit('togglePulseSpinner', true)

        axios.post(`${config.app.host}/api/token`, {
          email: this.email,
          allVerified: this.allChecked
        })
        .then((result) => {
          setTimeout(() => {
            this.submitError = ''
            this.$store.commit('setContractInfo', result.data.info)
            this.$store.commit('togglePulseSpinner', false)
            this.$store.commit('toggleSaleContract', true)
          }, 1200)
        })
        .catch((err) => {
          setTimeout(() => {
            this.$store.commit('togglePulseSpinner', false)

            if (err.message === 'Network Error') {
              this.submitError = 'Error processing request.'
              this.clearAllChecked()
            }
          }, 1200)
        })
      } else {
        this.$store.commit('togglePulseSpinner', false)
        console.log('error no checked')
      }
    },

    closeModal () {
      this.$store.commit('clearContractInfo')
      this.$store.commit('toggleSaleContract', false)
      this.$store.commit('showSaleModal', false)

      this.clearAllChecked()
    },

    clearAllChecked () {
      this.email = ''
      this.showEmailWarn = false
      this.saleTermsRead.checked = false
      this.saleTermsRead.enable = false
      this.erc20WalletAddress.checked = false
      this.erc20WalletAddress.enable = false
      this.noExchangeAddress.checked = false
      this.noExchangeAddress.enable = false
      this.havePrivateKeys.checked = false
      this.havePrivateKeys.enable = false
    },

    resetTerms () {
      const vm = this

      vm.saleTermsRead.enable = true
      vm.erc20WalletAddress.enable = true
      vm.noExchangeAddress.enable = true
      vm.havePrivateKeys.enable = true

      // var saleTerms = document.querySelector('#sale-terms')
      // saleTerms.addEventListener('scroll', function () {
      //   if (this.scrollHeight - (this.offsetHeight - 2) === this.scrollTop) {
      //     vm.saleTermsRead.enable = true
      //     vm.erc20WalletAddress.enable = true
      //     vm.noExchangeAddress.enable = true
      //     vm.havePrivateKeys.enable = true
      //   }
      // })
    },

    goToTroubleshooting () {
      this.$store.commit('showSaleModal', false)
      this.$router.push({ path: '/faq/troubleshooting/why-is-my-purchase-submission-not-working' })
    }
  },

  mounted () {
    this.resetTerms()
  },

  updated () {
    this.resetTerms()
  }
}
</script>

<style lang="scss">
@import '../assets/css/colors';

section.sale-modal {
  z-index: 1000002;
  margin: 0;
}

#showSaleModal .modal-content {
  .close-modal {
    position: absolute;
    right: 20px;

    &:hover {
      cursor: pointer;
    }
  }
}

#sale-modal--content {
  height: calc(100% + 30px);
  width: calc(100% + 30px);
  position:relative;
  left: -15px;
  top: -15px;
  border-radius: 10px;
  overflow: hidden;
  background-color: #fff;

  .matryx-button--blue {
    max-width: 300px;

    &:hover {
      background-color: $light-green;
      border-color: $light-green;
    }
  }

  .email-warn {
    margin: 0 auto;
  }

  .salemodal__header {
    height:50px;
    width: 100%;
    margin: 0;
    padding: 15px;
    position:relative;
    background-color: $matryx-lighter-grey;

    img {
      height: 100%;
      float: left;
    }
  }
  .salemodal__body {
    padding: 15px;
    text-align:center;

    &__terms {

      h3 {
        margin-bottom: 20px;
      }

      &__link {
        margin-top: 10px;
        margin-bottom: -20px;
        a {
          &:focus {
            outline: none;
          }
        }
      }

      &__document {
        width: calc(100% - 40px);
        max-height: 200px;
        margin: 0 auto;
        padding: 10px;
        text-align: left;
        overflow-y: scroll;
        overflow-x: hidden;
        border: 1px solid $matryx-dark-grey !important;

        &::-webkit-scrollbar {
          width: 1em;
        }
        &::-webkit-scrollbar-track {
          border-left: 1px solid $matryx-dark-grey;
        }
        &::-webkit-scrollbar-thumb {
          background-color: $matryx-dark-grey;
        }
      }

      &__checklist {
        display: flex;
        flex-direction: column;
        align-items: flex-start;
        padding: 30px 20px;

        .terms-email {
          display: flex;
          flex-direction: column;
          align-items: center;
          width: 100%;
        }

        .terms-email--input {
          padding: 5px 20px;
          width: 300px;
          height: 50px;
          font-size: 14px;

          &:focus {
            outline: none;
          }
        }

        p {
          font-size: 18px;
          margin-bottom: 5px;
        }
        .submit-btn{
          margin: 30px auto;
          height: 70px;
        }
        .submit-btn:disabled {
          opacity: 0.4;
          box-shadow: none;
          cursor: not-allowed;

          &:hover {
            background-color: $matryx-blue;
          }
        }

        div {
          text-align:left;
          padding: 10px 20px;
        }
      }

      &__address {
        display: none;
      }
    }
  }
}

.fade-enter-active, .fade-leave-active {
  transition: opacity .5s
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0
}


</style>
