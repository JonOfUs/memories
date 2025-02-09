<template>
  <Modal
    @close="close"
    size="normal"
    v-if="show"
    :sidebar="!isRoot ? this.folderPath : null"
  >
    <template #title>
      {{ t("memories", "Share Folder") }}
    </template>

    <div v-if="isRoot">
      {{ t("memories", "You cannot share the root folder") }}
    </div>
    <div v-else>
      {{ t("memories", "Use the sidebar to share this folder.") }} <br />
      {{
        t(
          "memories",
          "If you create a public link share, click on refresh and a corresponding link to Memories will be shown below."
        )
      }}
    </div>

    <div class="links">
      <a
        v-for="link of links"
        :key="link.url"
        :href="link.url"
        target="_blank"
        rel="noopener noreferrer"
      >
        {{ link.url }}
      </a>
    </div>

    <template #buttons>
      <NcButton class="primary" @click="refreshUrls">
        {{ t("memories", "Refresh") }}
      </NcButton>
    </template>
  </Modal>
</template>

<script lang="ts">
import { Component, Emit, Mixins } from "vue-property-decorator";

import axios from "@nextcloud/axios";
import { generateOcsUrl, generateUrl } from "@nextcloud/router";
import { NcButton } from "@nextcloud/vue";

import * as utils from "../../services/Utils";
import Modal from "./Modal.vue";
import GlobalMixin from "../../mixins/GlobalMixin";
import UserConfig from "../../mixins/UserConfig";
import { Type } from "@nextcloud/sharing";

@Component({
  components: {
    Modal,
    NcButton,
  },
})
export default class FolderShareModal extends Mixins(GlobalMixin, UserConfig) {
  private show = false;
  private folderPath = "";
  private links: { url: string }[] = [];

  get isRoot() {
    return this.folderPath === "/" || this.folderPath === "";
  }

  @Emit("close")
  public close() {
    this.show = false;
  }

  public open() {
    this.folderPath = utils.getFolderRoutePath(this.config_foldersPath);
    this.show = true;
    globalThis.OCA.Files.Sidebar.setActiveTab("sharing");
    this.refreshUrls();
  }

  private async refreshUrls() {
    const query = `format=json&path=${encodeURIComponent(
      this.folderPath
    )}&reshares=true`;
    const url = generateOcsUrl(`/apps/files_sharing/api/v1/shares?${query}`);
    const response = await axios.get(url);
    const data = response.data?.ocs?.data;
    if (data) {
      this.links = data
        .filter((s) => s.share_type === Type.SHARE_TYPE_LINK && s.token)
        .map((share: any) => ({
          url:
            window.location.origin +
            generateUrl(`/apps/memories/s/${share.token}`),
        }));
    }
  }
}
</script>

<style lang="scss" scoped>
.links {
  margin-top: 1em;
  a {
    display: block;
    margin-bottom: 0.2em;
    color: var(--color-primary-element);
  }
}
</style>