<template>
  <div>
    <div v-if="!loggedIn && !hidden">
      <button @click="login">
        {{ text || 'Login to reply' }}
      </button>
    </div>
    <slot v-if="loggedIn"></slot>
  </div>
</template>

<script>

export default {
  props: {
    text: String,
    hidden: Boolean,
  },
  computed: {
    loggedIn() {
      return this.$store.state.auth.username;
    },
  },
  methods: {
    login() {
      this.$store.commit( 'auth/toggleAccountModal' );
    },
  },
};
</script>
