<template>
  <div>
    <component ref="currentComponent" :is="currentComponent"></component>
    <div v-show="showOverlay" @click="resetPrompts" class="overlay"></div>
  </div>
</template>

<script>
import Help from "./Help";
import Info from "./Info";
import Delete from "./Delete";
import Rename from "./Rename";
import Tag from "./Tag";
import Download from "./Download";
import Move from "./Move";
import Gallery from "./Gallery";
import Copy from "./Copy";
import NewFile from "./NewFile";
import NewDir from "./NewDir";
import Replace from "./Replace";
import ReplaceRename from "./ReplaceRename";
import Share from "./Share";
import Upload from "./Upload";
import ShareDelete from "./ShareDelete";
import { mapState } from "vuex";
import buttons from "@/utils/buttons";

export default {
  name: "prompts",
  components: {
    Info,
    Delete,
    Rename,
    Tag,
    Download,
    Move,
    Gallery,
    Copy,
    Share,
    NewFile,
    NewDir,
    Help,
    Replace,
    ReplaceRename,
    Upload,
    ShareDelete,
  },
  data: function () {
    return {
      pluginData: {
        buttons,
        store: this.$store,
        router: this.$router,
      },
    };
  },
  created() {
    window.addEventListener("keydown", (event) => {
      if (this.show == null) return;

      let prompt = this.$refs.currentComponent;

      // Esc!
      if (event.keyCode === 27) {
        event.stopImmediatePropagation();
        this.$store.commit("closeHovers");
      }

      // Enter
      if (event.keyCode == 13) {
        switch (this.show) {
          case "delete":
            prompt.submit();
            break;
          case "copy":
            prompt.copy(event);
            break;
          case "move":
            prompt.move(event);
            break;
          case "gallery":
            prompt.move(event);
            break;
          case "replace":
            prompt.showConfirm(event);
            break;
        }
      }
    });
  },
  computed: {
    ...mapState(["show", "plugins"]),
    currentComponent: function () {
      const matched =
        [
          "info",
          "help",
          "delete",
          "rename",
          "tag",
          "move",
          "gallery",
          "copy",
          "newFile",
          "newDir",
          "download",
          "replace",
          "replace-rename",
          "share",
          "upload",
          "share-delete",
        ].indexOf(this.show) >= 0;

      return (matched && this.show) || null;
    },
    showOverlay: function () {
      return (
        this.show !== null && this.show !== "search" && this.show !== "more"
      );
    },
  },
  methods: {
    resetPrompts() {
      this.$store.commit("closeHovers");
    },
  },
};
</script>
