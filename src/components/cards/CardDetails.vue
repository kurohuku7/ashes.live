<template>
  <div v-if="error">
    <h1 class="phg-discard">No card found</h1>

    <p class="text-lg">
      Sorry, this does not appear to be a valid card!
      <router-link v-if="showLegacy" :to="{ name: 'CardDetails', params: { stub: stub } }"
        >Try the Ashes Reborn version.</router-link
      >
      <router-link v-else :to="{ name: 'CardDetailsLegacy', params: { stub: stub } }"
        >Try the Legacy version.</router-link
      >
    </p>
  </div>
  <div v-else-if="!card">
    <h1 class="phg-side-action text-gray">
      <i class="fas fa-circle-notch fa-spin"></i> Loading...
    </h1>
  </div>
  <div v-else>
    <h1 class="phg-side-action">{{ card.name }}</h1>

    <div class="lg:flex">
      <div class="lg:w-2/3">
        <section class="md:flex md:flex-nowrap">
          <div class="md:flex-none md:mr-8 mb-8">
            <deck-qty-buttons :card="card" class="mb-4" standalone></deck-qty-buttons>
            <img
              class="bg-gray-light"
              :class="[!showLegacy ? 'rounded-lg shadow-lg' : '']"
              :src="imageURL"
              alt=""
              :width="showLegacy ? 332 : 300"
              :height="showLegacy ? 452 : 420"
            />
          </div>

          <div>
            <div class="md:flex-grow">
              <h2>English</h2>

              <h3 class="mb-0">
                {{ card.name }}
                <span v-if="card.phoenixborn" class="text-gray" :title="card.phoenixborn">
                  ({{ card.phoenixborn.split(/,?[ ]/)[0] }})
                </span>
                <span v-if="card.release.is_phg === false" class="text-gray pl-1">†</span>
              </h3>

              <p class="mt-0 mb-2 text-sm">
                {{ card.type }}
                <span v-if="card.placement">
                  <span class="divider"><span class="alt-text">-</span></span>
                  {{ card.placement }}
                </span>
              </p>

              <card-costs class="mb-4" :costs="card.cost" is-horizontal></card-costs>

              <div v-if="isPhoenixborn" class="my-2">
                <strong
                  v-if="card.battlefield !== undefined"
                  class="inline-block border border-red-light px-1"
                  >Battlefield {{ card.battlefield }}</strong
                >
                <strong
                  v-if="card.life !== undefined"
                  class="inline-block border border-green-light px-1 mx-1"
                  >Life {{ card.life }}</strong
                >
                <strong
                  v-if="card.spellboard !== undefined"
                  class="inline-block border border-blue-dark px-1"
                  >Spellboard {{ card.spellboard }}</strong
                >
              </div>
              <div v-if="card.text">
                <hr v-if="!isPhoenixborn" class="my-4 border-gray-light" />
                <div class="leading-snug text-sm">
                  <card-codes
                    :content="card.text"
                    :is-legacy="showLegacy"
                    is-card-effect
                  ></card-codes>
                </div>
              </div>
              <div v-if="hasStats && !isPhoenixborn" class="my-2">
                <!-- Placeholders ensure that our stats are always in about the same locations -->
                <span
                  v-if="card.copies !== undefined"
                  class="inline-block border border-gray-dark px-1 mr-1"
                  >{{ card.copies }}</span
                >

                <strong
                  v-if="card.attack !== undefined"
                  class="inline-block border border-red-light px-1"
                  >Attack {{ card.attack }}</strong
                >
                <span v-else class="inline-block invisible">Attack --</span>

                <strong
                  v-if="card.life !== undefined"
                  class="inline-block border border-green-light px-1 mx-1"
                  >Life {{ card.life }}</strong
                >
                <span v-else class="inline-block invisible mx-1">Life --</span>

                <strong
                  v-if="card.recover !== undefined"
                  class="inline-block border border-blue-dark px-1"
                  >Recover {{ card.recover }}</strong
                >
                <span v-else class="inline-block invisible">Recover --</span>
              </div>
            </div>

            <div class="md:flex-grow mt-8">
              <h2 class="mb-4">日本語</h2>
              <h3 id="name-ja" contenteditable="true" class="mb-0 outline-none">
                {{
                  !card.name_ja || card.name_ja === '' || card.name_ja === '\n'
                    ? 'カード名を入力'
                    : card.name_ja
                }}

                <span v-if="card.phoenixborn" class="text-gray" :title="card.phoenixborn">
                  ({{ card.phoenixborn.split(/,?[ ]/)[0] }})
                </span>
                <span v-if="card.release.is_phg === false" class="text-gray pl-1">†</span>
              </h3>

              <p class="mt-0 mb-2 text-sm">
                {{ cardTypeJa }}

                <span v-if="card.placement">
                  <span class="divider"><span class="alt-text">-</span></span>
                  {{ cardPlacementJa }}
                </span>
              </p>

              <!-- <card-costs class="mb-4" :costs="card.cost" is-horizontal></card-costs> -->

              <div v-if="isPhoenixborn" class="my-2">
                <strong
                  v-if="card.battlefield !== undefined"
                  class="inline-block border border-red-light px-1"
                  >戦場 {{ card.battlefield }}</strong
                >
                <strong
                  v-if="card.life !== undefined"
                  class="inline-block border border-green-light px-1 mx-1"
                  >ライフ {{ card.life }}</strong
                >
                <strong
                  v-if="card.spellboard !== undefined"
                  class="inline-block border border-blue-dark px-1"
                  >魔法置場 {{ card.spellboard }}</strong
                >
              </div>
              <div v-if="card.text">
                <hr v-if="!isPhoenixborn" class="my-4 border-gray-light" />
                <div id="text-ja" class="leading-snug text-sm outline-none" contenteditable="true">
                  <card-codes
                    :content="card.text_ja"
                    :is-legacy="showLegacy"
                    is-card-effect
                  ></card-codes>
                </div>
              </div>
            </div>
            <button
              id="jp-submit-button"
              class="btn btn-blue px-4 py-1 mb-4 mt-2"
              @click="saveJapaneseText"
            >
              日本語データを保存
            </button>
          </div>
        </section>
        <hr class="my-4 border-gray-light lg:hidden" />
        <!-- <comments :entity-id="entity_id" :last-seen-entity-id="last_seen_entity_id" /> -->
      </div>
      <div class="lg:w-1/3 lg:pl-8">
        <hr class="my-4 border-gray-light lg:hidden" />

        <h2>Card stats</h2>

        <table class="border-collapse table-auto mb-4" cellspacing="0" cellpadding="0">
          <tbody>
            <card-meta-row
              v-if="
                card.dice &&
                card.dice.length &&
                !(card.dice.length === 1 && card.dice[0] === 'basic')
              "
              aria-label="Dice"
              label="Needs:"
            >
              <span
                v-for="(dieType, index) of card.dice"
                :key="dieType"
                :class="`phg-${dieType}-power`"
                :title="`${capitalize(dieType)} Magic`"
              >
                <span class="alt-text">{{ capitalize(dieType) }} Magic</span>
                <span v-if="index < card.dice.length - 1" class="divider mr-1"
                  ><span class="alt-text">, </span></span
                >
              </span>
            </card-meta-row>
            <card-meta-row label="Usage:">
              <strong>{{ usage.users }} user<span v-if="usage.users != 1">s</span></strong>
              <span v-if="usage.users != 1">have</span><span v-else>has</span> this card in
              <strong>{{ usage.decks }} deck<span v-if="usage.decks != 1">s</span></strong>
            </card-meta-row>
            <card-meta-row label="Release:">
              <router-link
                v-if="preconstructedDeck && preconstructedDeck.title == card.release.name"
                :to="{ name: 'DeckDetails', params: { id: preconstructedDeck.id } }"
                >{{ card.release.name }}</router-link
              >
              <span v-else>{{ card.release.name }}</span>
            </card-meta-row>
            <card-meta-row
              v-if="preconstructedDeck && preconstructedDeck.title != card.release.name"
              label="Deck:"
            >
              <router-link :to="{ name: 'DeckDetails', params: { id: preconstructedDeck.id } }">{{
                preconstructedDeck.title
              }}</router-link>
            </card-meta-row>
          </tbody>
        </table>
        <div v-if="phoenixborn || (relatedConjurations && relatedConjurations.length)">
          <h3>Related cards</h3>

          <ul class="list-disc pl-4 mb-6">
            <card-related-cards
              v-if="relatedSummons"
              :card="card"
              :summons="relatedSummons"
              :conjurations="relatedConjurations"
              :show-legacy="showLegacy"
            ></card-related-cards>
            <card-related-cards
              v-if="phoenixborn"
              :card="card"
              :summons="phoenixborn"
              :conjurations="phoenixbornConjurations"
              :show-legacy="showLegacy"
            ></card-related-cards>
            <card-related-cards
              v-if="phoenixbornUnique"
              :card="card"
              :summons="phoenixbornUnique"
              :conjurations="phoenixbornUniqueConjurations"
              :show-legacy="showLegacy"
            ></card-related-cards>
          </ul>
        </div>
        <button
          v-if="isAuthenticated"
          class="btn py-1 mb-8 w-full"
          :class="{ 'btn-blue': !last_seen_entity_id }"
          @click="toggleSubscription()"
        >
          <i
            class="fa-bookmark"
            :class="{ far: last_seen_entity_id, fas: !last_seen_entity_id }"
          ></i>
          <span v-if="last_seen_entity_id"> Unsubscribe</span><span v-else> Subscribe</span>
        </button>
        <card-usage :stub="stub"></card-usage>
      </div>
    </div>
  </div>
</template>

<script>
import { request } from '/src/utils/index.js'
import { capitalize } from '/src/utils/text.js'
import useHandleResponseError from '/src/composition/useHandleResponseError.js'
import CardCodes from '../shared/CardCodes.vue'
import CardCosts from '../shared/CardCosts.vue'
import Comments from '../shared/Comments.vue'
import DeckQtyButtons from '../shared/DeckQtyButtons.vue'
import CardMetaRow from './CardMetaRow.vue'
import CardRelatedCards from './CardRelatedCards.vue'
import CardUsage from './CardUsage.vue'

export default {
  name: 'CardDetails',
  props: ['stub'],
  setup() {
    // Standard composite containing { toast, handleResponseError }
    return useHandleResponseError()
  },
  components: {
    CardCodes,
    CardCosts,
    Comments,
    CardMetaRow,
    CardRelatedCards,
    DeckQtyButtons,
    CardUsage,
  },
  data: () => ({
    card: null,
    entity_id: null,
    last_seen_entity_id: null,
    usage: null,
    preconstructedDeck: null,
    relatedSummons: null,
    relatedConjurations: null,
    phoenixborn: null,
    phoenixbornConjurations: null,
    phoenixbornUnique: null,
    phoenixbornUniqueConjurations: null,
    error: false,
  }),
  beforeMount() {
    request(`/v2/cards/${this.stub}/details${this.showLegacy ? '?show_legacy=true' : ''}`)
      .then((response) => {
        this.card = response.data.card
        console.log(this.card)
        this.entity_id = response.data.entity_id
        this.last_seen_entity_id = response.data.last_seen_entity_id
        this.usage = response.data.usage
        this.preconstructedDeck = response.data.preconstructed_deck
        if (response.data.related_cards.phoenixborn !== undefined) {
          this.phoenixborn = response.data.related_cards.phoenixborn
          this.phoenixbornConjurations = response.data.related_cards.phoenixborn_conjurations
          this.phoenixbornUnique = response.data.related_cards.phoenixborn_unique
          this.phoenixbornUniqueConjurations =
            response.data.related_cards.phoenixborn_unique_conjurations
        } else if (response.data.related_cards.summoning_cards !== undefined) {
          this.relatedSummons = response.data.related_cards.summoning_cards
          this.relatedConjurations = response.data.related_cards.conjurations
        }

        // And set the site title
        document.title = `${this.card.name} - Ashes.live`

        this.card.text_ja = this.card.text_ja ?? this.card.text
      })
      .catch((error) => {
        this.error = true
      })
  },
  computed: {
    showLegacy() {
      return !!this.$route.meta.showLegacy
    },
    imageURL() {
      if (!this.card) return ''
      if (this.showLegacy)
        return `${import.meta.env.VITE_CDN_URL}/legacy/images/cards/${this.stub}.jpg`
      return `${import.meta.env.VITE_CDN_URL}/images/cards/${this.stub}.jpg`
    },
    isPhoenixborn() {
      return this.card.type === 'Phoenixborn'
    },
    hasStats() {
      // This covers Phoenixborn, too, because they always have a life value
      return (
        this.card.attack !== undefined ||
        this.card.life !== undefined ||
        this.card.recover !== undefined ||
        this.card.copies !== undefined
      )
    },
    isAuthenticated() {
      return this.$store.getters['player/isAuthenticated']
    },
    cardTypeJa() {
      switch (this.card.type) {
        case 'Phoenixborn':
          return 'フェニックスボーン'
        case 'Ready Spell':
          return '設置魔法'
        case 'Action Spell':
          return '即時魔法'
        case 'Conjuration':
          return '召喚獣'
        case 'Ally':
          return '仲間'
        default:
          return this.card.type
      }
    },
    cardPlacementJa() {
      switch (this.card.placement) {
        case 'Spellboard':
          return '魔法置場'
        case 'Battlefield':
          return '戦場'
        case 'Discard':
          return '捨札'
        default:
          return this.card.placement
      }
    },
  },
  methods: {
    capitalize,
    toggleSubscription() {
      const method = this.last_seen_entity_id ? 'delete' : 'post'
      request(`/v2/subscription/${this.entity_id}`, { method }).then((response) => {
        if (method === 'delete') {
          this.last_seen_entity_id = null
          this.toast.info('You have unsubscribed from this card!')
        } else {
          this.last_seen_entity_id = response.data.last_seen_entity_id
          this.toast.info("You have subscribed to this card's comments!")
        }
      })
    },
    async saveJapaneseText() {
      this.card.name_ja = document.querySelector('#name-ja').textContent.trim()

      this.card.text_ja = ''

      document.querySelectorAll('#text-ja > *').forEach((part) => {
        // 疲労アイコンをテキストに保存するために追加
        part.querySelectorAll('.phg-exhaust').forEach((el) => {
          el.textContent = '[[exhaust]]'
        })

        // 捨札アイコン
        part.querySelectorAll('#text-ja .phg-discard').forEach((el) => {
          el.textContent = '[[discard]]'
        })

        // 改行
        if (part.tagName === 'br') {
        }

        if (part.classList.contains('inexhaustible-effects')) {
          const paragraphs = []
          part.querySelectorAll('p').forEach((p) => {
            paragraphs.push(
              '* ' + p.textContent.replace('(Inexhaustible effect) ', '').replace('&nbsp;', ' ')
            )
          })
          this.card.text_ja += paragraphs.join('\n\n')
        } else {
          this.card.text_ja += part.textContent + '\n\n'
        }

        // 取得し終わったら疲労トークンのテキストを削除
        part.querySelectorAll('.phg-exhaust').forEach((el) => {
          el.textContent = ''
        })

        part.querySelectorAll('.phg-discard').forEach((el) => {
          el.textContent = ''
        })
      })

      // 疲労影響なしの能力を * に変える
      this.card.text_ja = this.card.text_ja.replace('(Inexhaustible effect) ', '\n\n* ')

      console.log(this.card.text_ja)

      await fetch(`${import.meta.env.VITE_API_URL}/v2/cards/${this.stub}/update-ja`, {
        method: 'post',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          name_ja: this.card.name_ja,
          text_ja: this.card.text_ja,
        }),
      })

      const submitButton = document.querySelector('#jp-submit-button')
      submitButton.innerText = '保存完了！'
      submitButton.className = 'btn btn-green px-4 py-1 mb-4'
      setTimeout(() => {
        submitButton.innerText = '日本語データを保存'
        submitButton.className = 'btn btn-blue px-4 py-1 mb-4'
      }, 2000)
    },
  },
}
</script>
