<template>
  <div class="card floating">
    <div class="card-title">
      <h2>{{ "Move to Gallery" }}</h2>
    </div>

    <!-- <div class="card-content">
      <file-list @update:selected="(val) => (dest = val)"></file-list>
    </div> -->

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
        class="button button--flat"
        @click="gallery"
        :disabled="$route.path === dest"
        :aria-label="'Gallery'"
        :title="'Gallery'"
      >
        {{ $t("buttons.move") }}
      </button>
    </div>
  </div>
</template>

<script>
import { mapState } from "vuex";
// import FileList from "./FileList";
import { files as api } from "@/api";
import buttons from "@/utils/buttons";
import * as upload from "@/utils/upload";

export default {
  name: "gallery",
//   components: { FileList },
  data: function () {
    return {
      current: window.location.pathname,
      dest: "/files/Gallery/",
    };
  },
  computed: mapState(["req", "selected"]),
  methods: {
    gallery: async function (event) {
      event.preventDefault();
      let items = [];

      for (let item of this.selected) {
        items.push({
          from: this.req.items[item].url,
          to: this.dest + encodeURIComponent(this.req.items[item].name),
          name: this.req.items[item].name,
        });
      }

      let action = async (overwrite, rename) => {
        buttons.loading("gallery");

        await api
          .move(items, overwrite, rename)
          .then(() => {
            buttons.success("gallery");
            this.$router.push({ path: this.dest });
          })
          .catch((e) => {
            buttons.done("gallery");
            this.$showError(e);
          });
      };

      let dstItems = (await api.fetch(this.dest)).items;
      let conflict = upload.checkConflict(items, dstItems);

      let overwrite = false;
      let rename = false;

      if (conflict) {
        this.$store.commit("showHover", {
          prompt: "replace-rename",
          confirm: (event, option) => {
            overwrite = option == "overwrite";
            rename = option == "rename";

            event.preventDefault();
            this.$store.commit("closeHovers");
            action(overwrite, rename);
          },
        });

        return;
      }

      action(overwrite, rename);
    },
  },
};
</script>
