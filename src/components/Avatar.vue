<template>
  <b-tooltip
    :label="author | usernameDisplay(owner)"
    size="is-small"
    type="is-black">
    <figure class="image avatar" :class="classes">
      <img
        class="is-rounded"
        :title="author | usernameDisplay(owner)"
        :src="avatarURL(author, owner)">
    </figure>
  </b-tooltip>
</template>

<script>
const SIZES = {
  small: 'is-16x16',
  medium: 'is-32x32',
  large: 'is-48x48',
};

export default {
  props: {
    author: String,
    owner: String,
    size: String,
  },
  computed: {
    classes() {
      return SIZES[this.size];
    },
  },
  methods: {
    avatarURL( author, owner ) {
      if ( author !== process.env.VUE_APP_ANON_USER ) {
        return `https://img.busy.org/@${author}`;
      }
      return `https://robohash.org/${owner}.png?set=set4`;
    },
  },
};
</script>
