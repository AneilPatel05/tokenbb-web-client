<template>
  <ShowIfLoggedIn :hidden="true" class="upvote">
    <div class="field has-addons">
      <div class="value-control">
      <div class="control control-value">
        <a>$ {{ value }}</a>
      </div>
      <div class="control control-length">
        <a>{{ votes.length }} Votes</a>
      </div>
      </div>
      <b-dropdown position="is-top-left" hoverable :disabled="voted">
        <p class="upvote-button" slot="trigger">
          <a class="is-small"
            :class="{ 'is-loading': this.fetching }"
            :disabled="voted"
            @click="handleClick">
            <!--<span>Upvote</span>-->
            <b-icon icon="arrow-up-drop-circle-outline" size="is-medium">
            </b-icon>
          </a>
        </p>

        <b-dropdown-item custom>
          <div class="container popover">
            <div class="columns is-mobile">
              <div class="column is-one-quarter vertical">
                <span class="percent-label">{{ percent }}%</span>
              </div>
              <div class="column is-three-quarter is-mobile">
                <input orient="horizontal" class="slider is-primary is-circle"
                  ref="slider"
                  @input="handleChange"
                  step="1"
                  min="0"
                  max="100"
                  :value="percent"
                  type="range">
              </div>
            </div>
          </div>
        </b-dropdown-item>
      </b-dropdown>
    </div>
  </ShowIfLoggedIn>
</template>

<script>
import { vote } from '../services/api.service.js';
import { Client } from 'dsteem';

import ShowIfLoggedIn from '@/components/ShowIfLoggedIn.vue';

import Timeout from 'await-timeout';

const client = new Client( 'https://api.steemit.com' );

export default {
  components: {
    ShowIfLoggedIn,
  },
  props: {
    author: String,
    permlink: String,
  },
  data() {
    return {
      percent: 10,
      fetching: false,
      paid: 0,
      value: '0.000',
      votes: [],
    };
  },
  computed: {
    voted() {
      return this.paid > 0
        || this.$store.state.auth.current === 'anon'
        || this.votes.filter( ( _vote ) => _vote.voter === this.$store.state.auth.current ).length > 0;
    },
  },
  mounted() {
    this.updateValue();
  },
  methods: {
    handleChange() {
      this.percent = this.$refs.slider.value;
    },
    async updateValue() {
      const data = await client.call( 'condenser_api', 'get_content', [ this.author, this.permlink ] );
      const pending = parseFloat( data.pending_payout_value.split( ' ' )[0] );
      this.paid = parseFloat( data.total_payout_value.split( ' ' )[0] );
      this.value = ( this.paid + pending ).toFixed( 3 );
      this.votes = data.active_votes;
    },
    async handleClick() {
      if ( this.voted ) {
        return this.$toast.open( {
          message: 'Oops! Already voted!.',
          type: 'is-danger',
        } );
      }
      this.fetching = true;

      try {
        await vote( this.author, this.permlink, this.$store.state.auth.current, this.percent );
        this.$toast.open( {
          message: 'Upvoted!',
          type: 'is-primary',
        } );
        await Timeout.set( 3000 );
        await this.updateValue();
      } catch ( err ) {
        console.error( 'oops!', err );
        this.$toast.open( {
          message: 'Oops! Could not upvote at this time. ' + err,
          type: 'is-danger',
        } );
      }
      this.fetching = false;
    },
  },
};
</script>
