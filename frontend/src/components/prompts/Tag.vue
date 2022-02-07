<template>
  <div class="card floating">
    <div class="card-title">
      <h2>{{ "Add Tag" }}</h2>
    </div>

    <div class="card-content">
      <p>
        {{ "Insert Tag for" }} <code>{{ oldName() }}</code
        >:
      </p>
      <input
        class="input input--block"
        v-focus
        type="text"
        v-model.trim="name"
        @keyup.enter="submit"
      />
    </div>

    <div class="card-action">
      <button
        class="button button--flat button--grey"
        @click="$store.commit('closeHovers')"
        :aria-label="$t('buttons.cancel')"
        :title="$t('buttons.cancel')"
      >
        {{ $t("buttons.cancel") }}
      </button>
      <button
        @click="submit"
        class="button button--flat"
        type="submit"
        :aria-label="'Tag'"
        :title="'Tag'"
      >
        {{ "add" }}
      </button>
    </div>
  </div>
</template>

<script>
import { mapState, mapGetters } from "vuex";
import url from "@/utils/url";
import { files as api } from "@/api";
export default {
  name: "tag",
  data: function () {
    return {
      name: "",
    };
  },
  computed: {
    ...mapState(["req", "selected", "selectedCount"]),
    ...mapGetters(["isListing"]),
  },
  methods: {
    cancel: function () {
      this.$store.commit("closeHovers");
    },
    oldName: function () {
      if (!this.isListing) {
        return this.req.name;
      }
      if (this.selectedCount === 0 || this.selectedCount > 1) {
        // This shouldn't happen.
        return;
      }
      return this.req.items[this.selected[0]].name;
    },
    submit: async function () {
      let oldLink = "";
      let newLink = "";
      let result = "";
      if (!this.isListing) {
        oldLink = this.req.url;
      } else {
        oldLink = this.req.items[this.selected[0]].url;
      }
      if (this.oldName().indexOf("T?" !== -1)) {
        let slice1 = this.oldName().indexOf("T?") + 2;
        let slice2 = this.oldName().indexOf(".jpg");
        result = this.oldName().slice(slice1, slice2);
      }
      //   newLink =
      //     url.removeLastDir(oldLink) + "/" + encodeURIComponent(this.oldName().replace('.jpg',"T?" + this.name + ".jpg"));
      //   console.log('newLink: ' + newLink);
      //   console.log('this.name: ' + this.name);
      // newLink =
      //   url.removeLastDir(oldLink) + "/" + this.oldName().replace('.jpg',"T?" + this.name + ".jpg")
      newLink =
        url.removeLastDir(oldLink) +
        "/" +
        encodeURIComponent(
          this.oldName().indexOf("T?") == -1
            ? this.oldName().replace(".jpg", "T?" + this.name + ".jpg")
            : this.oldName().replace(result, this.name)
        );
      try {
        await api.move([{ from: oldLink, to: newLink }]);
        if (!this.isListing) {
          this.$router.push({ path: newLink });
          return;
        }
        this.$store.commit("setReload", true);
      } catch (e) {
        this.$showError(e);
      }
      this.$store.commit("closeHovers");
    },
  },
};
</script>